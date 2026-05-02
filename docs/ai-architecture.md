# AI Architecture
Note: this doc was AI generated

FocusBlock is built around a central premise: the cognitive cost of async collaboration is not caused by too many messages, it is caused by the reconstruction tax — the work a person has to do every time they switch contexts to figure out where they are, what was decided, and what they need to do next. Every AI design decision in FocusBlock is an attempt to eliminate that tax at a specific point in the workflow.

This document explains how the AI pipeline works, why it is designed the way it is, and what tradeoffs were made at each decision point.

---

## Why Events Are Processed Asynchronously

The first architectural decision is one that shapes everything else: webhook events are never processed inline.

When Slack, Jira, or Google sends a webhook — a new message, a comment, a ticket update — FocusBlock writes the raw payload to the `raw_events` table immediately and returns `200 OK`. Processing happens separately, on a 30-second pg_cron schedule, via a Supabase edge function that drains the queue.

```
Webhook received
      │
      ▼
raw_events (status: 'pending')   ← write and return 200 immediately
      │
      │  (30 seconds later)
      ▼
Event processor edge function
      │
      ├── Block association
      ├── Urgency classification
      ├── Signal clustering
      ├── AI summarization
      └── Feed update via Supabase Realtime
```

This is not just an infrastructure choice. It is a product requirement.

Associating a message with the right Block, classifying its urgency, clustering it into a Signal, and regenerating the Block's Feed summary requires multiple AI calls with substantial context — the Block's existing Signals, the decision log, prior Cues from the same source, and cached document content. Running all of that synchronously inside a webhook handler would be slow, fragile, and expensive. More importantly, it would mean the system's ability to surface good context is constrained by webhook response time.

Decoupling receipt from processing means each of these jobs can be done thoroughly. A message that arrives at 9:03 AM will appear in the user's Feed at 9:03:30 AM — half a minute later, fully contextualized, correctly routed, and summarized — rather than instantly, raw, and undifferentiated.

**The tradeoff:** A 30-second processing delay. This is acceptable because FocusBlock is designed for async-first teams where a 30-second delay on triage is imperceptible. It is explicitly not designed for incident response or real-time chat. Blocking items break through this model — they are flagged and surfaced as soon as the next processing cycle runs, which is fast enough for the target use case.

---

## The Five AI Jobs

Every event that enters the pipeline runs through up to five AI jobs in sequence. Each job has a specific input, a specific output, and a defined failure mode.

| Job | Input | Output | Failure behavior |
|---|---|---|---|
| Block association | Raw Cue content + existing Blocks | `block_id` or `unassigned` | Routes to Unassigned Queue |
| Urgency classification | Cue content + sender metadata | `blocking` / `action_required` / `fyi` | Defaults to `action_required` |
| Signal clustering | New Cue + existing open Signals in Block | Assign to existing Signal or create new | Creates new Signal |
| AI summarization | All Cues in Signal + Block context | `ai_summary` string per Signal + Block Feed summary | Retains prior summary |
| Task suggestion | Cue content + full doc body (if linked) | `task_suggestions` row or null | Skips silently |

Jobs 1 and 2 run on every incoming Cue. Jobs 3 and 4 run only after a Block has been assigned. Job 5 runs only for Google Doc comment Cues where a doc body is cached.

---

## Block Association: Confidence Thresholds and the Unassigned Queue

The association job answers one question: which Block does this message belong to?

FocusBlock uses a three-tier model. The first tier requires no AI at all.

**Tier 1 — Structural association.** If a message arrives in a Slack channel, Jira ticket, or Google Doc that was created from within a Block (the `spawned = true` flag in `block_sources`), it is associated with that Block immediately. No inference. The connection is permanent and structural. This is the highest-confidence path and the reason spawning tools from a Block is the recommended workflow — it eliminates the association problem entirely for all activity in that source.

**Tier 2 — AI inference.** For messages in sources not yet connected to a Block, the AI reads the Cue content and evaluates it against every active Block the user has. The evaluation uses four signals:

1. **Semantic similarity** — embedding-based similarity between the Cue content and the Block's name, description, and recent Signal summaries
2. **Participant overlap** — messages involving people who are active in a Block are weighted toward that Block
3. **Artifact links** — a Slack message that references a Jira ticket ID already in a Block is auto-assigned to that Block regardless of confidence score
4. **Temporal clustering** — activity that clusters around the same time window as an active Block's recent messages is scored higher

The AI produces a confidence score between 0 and 1 for each candidate Block. If the top candidate exceeds the user's threshold (default: 0.75, tunable per user in settings), the message is silently associated and the user never sees the decision. If no candidate clears the threshold, the message goes to Tier 3.

**Why 0.75 and not higher?** A higher threshold (say, 0.90) would route more messages to the Unassigned Queue, which means more manual work for the user and slower AI learning. A lower threshold (say, 0.60) would produce more wrong associations, which is worse than no association — a message silently routed to the wrong Block erodes trust in the whole system. 0.75 was chosen as the point where wrong associations are rare enough that the occasional correction in the side panel (one tap) costs less than the queue management that a higher threshold would create. This is tunable precisely because different users have different tolerances.

**Tier 3 — Unassigned Queue.** When no Block clears the threshold, the Cue is stored with `is_unassigned = true` and surfaced in the Feed as part of the Unassigned card. The card shows the AI's best-guess Block (even if it didn't clear the threshold) and a confidence label. Every time a user accepts or corrects an association, that signal is fed back into the model. The threshold tuning happens at the user level; the learning happens at the model level.

```
Incoming Cue
      │
      ├── Spawned source? ──────────────────────────────► Tier 1: auto-assign (no AI)
      │
      ├── Known source, not spawned? ──► AI scores all active Blocks
      │                                         │
      │                               Top score ≥ 0.75?
      │                                    │         │
      │                                   YES        NO
      │                                    │         │
      │                                    ▼         ▼
      │                             Tier 2: silent   Tier 3: Unassigned Queue
      │                             association      (best guess shown, user decides)
      │
      └── Unknown source entirely? ──► Tier 3 always
```

---

## Two-Tier Slack Clustering: How Conversation Boundaries Are Detected

Slack is the hardest integration to handle well. A busy channel might have 200 messages a day spanning a dozen unrelated conversations — and none of those conversations have clean boundaries. There are no ticket IDs, no explicit topics, often no threading.

FocusBlock handles this with a two-tier clustering model that uses the structure Slack provides before asking the AI to do inference.

**Tier 1 — Thread boundaries.** A Slack thread (a reply to a specific message) is treated as a discrete Signal. This is the highest-confidence boundary in Slack: when someone uses the thread feature, they are explicitly scoping a conversation. FocusBlock respects that. Every threaded reply is associated with the Signal created from the parent message. No AI needed; the `thread_ts` field in the `cues` table preserves this structure.

**Tier 2 — AI clustering for top-level messages.** Top-level messages (messages not in a thread) are the harder problem. A sequence of top-level messages in a channel might be three separate conversations interleaved with each other, or one long conversation that nobody threaded, or background noise.

The AI reads consecutive top-level messages and groups them into clusters based on:

- **Semantic similarity** — messages about the same topic are grouped together
- **Participant overlap** — messages between the same people are more likely to be one conversation
- **Temporal proximity** — messages close in time are more likely to be related
- **Explicit references** — a message that replies to or references another top-level message is grouped with it

Each cluster is treated as a single Signal, equivalent to a thread. If the AI is uncertain about a boundary (two messages that could belong to either of two clusters), it surfaces the proposed grouping as a low-confidence suggestion. The user confirms or adjusts the boundary once, and that confirmation becomes a training signal for future messages in that channel.

**Why not use AI for everything, including threads?** Threads are a user-declared boundary. Using AI to override or reinterpret a boundary the user explicitly set would produce wrong results and erode trust. The principle is: use structure before inference, and use inference only where structure is absent.

**DM handling.** Direct messages and group DMs are clustered identically to channel messages, but every Signal sourced from a DM is tagged with `is_dm = true` in the `cues` table and displayed with a DM indicator in the UI. This ensures users always know when a Signal contains private context before they respond or forward anything.

---

## AI Summarization: Two Levels, One Paragraph Each

Summarization runs at two levels: the Signal and the Block.

**Signal-level summarization** runs after a new Cue is added to a Signal. The AI reads all Cues in the Signal and writes a one-to-two sentence summary that captures: what the conversation is about, what was said, and whether anything is unresolved. This summary appears in the side panel Signal list — it is the first thing a user reads when they open a Feed card.

The prompt is intentionally constrained:

```
You are summarizing a conversation thread for a knowledge worker who has not read it.
Write 1-2 sentences. Cover: what the conversation is about, the current state,
and whether action is required. Plain prose only. No bullet points. No filler.
```

The constraint against bullet points is deliberate. Bullets encourage the AI to list everything. Prose forces it to find the connective thread — which is the actual product value.

**Block-level summarization** runs after Signal summaries are updated. The AI reads all open Signal summaries for the Block, the last three decision log entries, and the count and urgency of open Tasks, and writes a single paragraph for the Feed card.

The key design constraint here is **consolidation over enumeration**. If two Signals and one Jira comment are all pointing at the same underlying issue — a release being blocked — the Block summary says that once, names the blocker, and says who needs to act. It does not list three separate items. The AI is explicitly prompted to find the connective tissue:

```
You are generating a Feed card summary. Write 2-4 sentences in plain prose.
Find the connective thread between Signals — if multiple Signals point at the
same root issue, say that once. End with what requires the user's action.
Do not use bullet points. Do not list each Signal separately.
```

This is the hardest summarization job in the product, and the one that most directly delivers the core value: one glance at the Feed card tells you the state of a Block.

---

## Block Context Caching: Why the AI Needs the Full Document

Block-level AI chat is only as good as the context it has access to. The naive implementation — fetch documents and tickets live when the user opens chat — has three problems: it is slow, it is expensive (API calls on every chat open), and it fails when a document is large.

FocusBlock solves this with a background caching layer in the `block_context` table.

When a Google Doc or Jira ticket is linked to a Block, a database trigger fires immediately:

```sql
CREATE TRIGGER on_block_source_linked
  AFTER INSERT ON block_sources
  FOR EACH ROW
  WHEN (NEW.provider IN ('google', 'jira'))
  EXECUTE FUNCTION trigger_context_sync();
```

This trigger calls the `refresh-block-context` edge function, which fetches the full document body (for Google Docs) or the full ticket description and acceptance criteria (for Jira) and writes it to `block_context`. By the time a user first opens Block chat, the content is already there.

A pg_cron job refreshes all cached content every 60 minutes. If a user opens chat and the cache is stale (older than 60 minutes), a live fetch runs as a fallback before the AI responds.

**Why cache the full document body and not a summary?** Because the AI needs to answer specific questions about document content. "What does the Technical Brief say about the error state for timeout?" requires the full text. A summary would lose the specific language. For long documents, the full body is chunked into the context window using a sliding window approach — but the content is never pre-summarized, because pre-summarization would discard exactly the details users are likely to ask about.

**What the AI has access to in Block chat:**

| Context source | What it provides |
|---|---|
| All open Signals with `ai_summary` | Current state of active conversations |
| Decision log (last 10 entries) | What was decided and when |
| Full Google Doc body (from `block_context`) | Complete document content for specific questions |
| Full Jira ticket description + acceptance criteria (from `block_context`) | Complete ticket context, not just comments |
| Task list (open tasks with urgency and status) | What work is in progress |

The context boundary is intentional: Block chat cannot see other Blocks. This prevents the AI from hallucinating connections between unrelated projects and ensures answers are grounded in the specific Block's history.

---

## Reply with AI: Layered Context for In-Context Drafting

Reply with AI is not a general-purpose chat feature. It is a specific, constrained drafting tool that runs at the Cue level — the individual message level — and uses a carefully layered context window to generate a reply that is actually useful.

When a user taps Reply with AI on a Cue, the AI receives five layers of context in order of specificity:

```
Layer 1 (most specific):  The Cue itself — the exact message being replied to
Layer 2:                  The full Signal — all Cues in this conversation thread
Layer 3:                  The Block decision log — what was decided previously
Layer 4:                  The linked Jira ticket — description, status, acceptance criteria
Layer 5 (broadest):       The full Google Doc body — if a doc is linked to this Block
```

Each layer serves a different purpose. Layer 1 tells the AI what it is replying to. Layer 2 gives it the conversation history so it doesn't repeat what was already said or contradict a prior message. Layer 3 prevents the AI from suggesting something that was already decided and logged. Layer 4 gives it the technical ground truth for Jira-related replies. Layer 5 gives it the written context for doc-related replies.

**Why not just use the Cue and the Signal?** Because a reply without Layers 3–5 will often be technically correct but contextually wrong. Consider: a developer asks "should we use 24-hour token expiry for admin users?" If the AI only has the current conversation, it might draft "yes, 24-hour expiry sounds reasonable." If it has the decision log, it knows the team already decided the opposite two weeks ago and drafts "we decided on Feb 28 to exempt admin users from the 24-hour limit — should we revisit that?"

The second reply is the product. The first reply is a liability.

The prompt for Reply with AI makes the priority order explicit:

```
You are drafting a reply to a specific message in an async collaboration tool.

Priority order for your response:
1. Never contradict or ignore prior decisions in the decision log
2. Be consistent with the current state of the Jira ticket if one is linked
3. Be consistent with the content of linked documents if relevant
4. Be direct. This is a work reply, not a letter.

Write the reply only. No preamble, no explanation of what you are doing.
The user will edit before sending.
```

The final instruction — "the user will edit before sending" — is included because it shapes the AI's output register. When the AI knows the output is a draft, it writes more naturally and less hedged than when it thinks it is producing a final answer.

---

## Task Suggestion: Reading the Full Document

Task suggestion is the fifth AI job, and the one that requires the most context. It runs after a Google Doc comment Cue is processed and asks: does this comment imply an actionable item that isn't already tracked in a Signal or Jira ticket?

The AI reads the full document body (from `block_context`) before answering. This matters because a comment like "see note in section 3" is meaningless without section 3. A comment like "this contradicts what we decided" is only actionable if the AI can read what "this" refers to in the document.

The suggestion is lightweight by design. The AI produces a single task title and a suggested urgency level — nothing else. The user sees a dismissable card below the Signal list:

```
Suggested task: Update error state section in Technical Brief — Action Required
[Add task]  [Dismiss]
```

The card is explicitly dismissable with one tap. The AI suggesting a task that the user dismisses is not a failure — it is the correct behavior for a feature that is intentionally opt-in. Task suggestions are surfaced, not imposed.

---

## AI Confidence as a First-Class Product Concept

One design principle runs through every AI job in FocusBlock: **confidence is a product decision, not just a model output.**

Every AI job that routes or classifies has an explicit confidence threshold that determines what happens when the model is uncertain. These thresholds are not hidden inside the model — they are surfaced as product behavior:

- Block association below the threshold → Unassigned Queue (visible, actionable)
- Slack cluster boundary uncertain → surfaced as a suggestion (user confirms once)
- Urgency uncertain → defaults to `action_required`, not `fyi` (errs toward delivery)
- Task suggestion low-confidence → not surfaced (suppressed, not a low-quality card)

The pattern is: when uncertain, prefer the action that costs the user less and preserves their trust. A message in the Unassigned Queue costs one tap to assign. A message silently mis-associated costs the user noticing it in the wrong Block, losing trust, and manually checking more things. The asymmetry drives every threshold decision.

This is also why the confidence threshold is user-tunable. A user who processes their Unassigned Queue quickly and prefers manual control can lower the threshold and see more items there. A user who trusts the AI and wants minimal interruption can raise it and let more things be silently associated. The default (0.75) is calibrated for the median user who wants the system to do most of the work but not make irreversible decisions without signal.
