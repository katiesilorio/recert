# Design Decisions

This document records the key product decisions made during FocusBlock's design and why they were made. Each decision involved meaningful tradeoffs. The goal is to document not just what was decided, but what was considered and what was given up.

---

## Decision: One Feed card per Block

**Options considered:**
- One card per Signal (more granular, more cards in the Feed)
- One card per Block (less granular, one consolidated card)
- Hybrid (group by urgency tier across all Blocks, regardless of which Block they belong to)

**Decision:** One card per Block.

**Rationale:** The Feed is a work session planner, not a notification list. One card per Block means the user starts the day by choosing which Block to engage with first, not by processing an arbitrary stream of individual messages. The AI summary paragraph does the work of consolidating multiple Signals into one coherent picture, finding the connective thread across a Slack cluster, a Jira comment, and a Doc annotation and expressing it as a single paragraph is exactly the job the AI should be doing.

The hybrid option (group by urgency tier across all Blocks) was ruled out because it destroys project context. If a user sees three "blocking" items from three different Blocks mixed together, they lose the ability to do a focused session on one Block. They're back to the same context-switching problem FocusBlock is designed to solve.

**Tradeoff:** A Block with three active Signals across different urgency levels shows them all on a single card. A user with one blocking Signal and two FYI Signals in the same Block might not immediately see that only one requires action. Mitigated by: the urgency indicator dot on the Feed card (shows the highest urgency level in the Block), the Signal count label, and the Signal list in the side panel which sorts by urgency automatically.

---

## Decision: AI inference as the default association model, not manual tagging

**Options considered:**
- Manual-first: users tag every message to a Block; AI assists as a secondary option
- AI-first: AI associates everything automatically; manual tagging is a fallback for edge cases
- Hybrid: AI suggests, user confirms every association before it is applied

**Decision:** AI-first, with manual tagging as a fallback and an Unassigned Queue for low-confidence cases.

**Rationale:** Manual tagging requires the user to interrupt their current work to categorize every incoming message. This is the same interruption problem that FocusBlock exists to eliminate. A system where the user has to tag things to get value from the AI is a system where the user does the AI's job.

The hybrid "AI suggests, user confirms" model was considered and rejected for the same reason: confirmation overhead on every message is the wrong default. At 50 messages a day, that's 50 interruptions to approve decisions the AI got right 85% of the time.

The key insight is that the cost of a wrong AI association is low and recoverable: one tap to reassign in the side panel. The cost of a wrong user decision in a manual-tagging system is identical. The AI-first model produces the same correction rate but eliminates the routine overhead.

**Tradeoff:** Some messages will be silently mis-associated, and users won't know unless they notice. This is the most significant trust risk in the product. Mitigated by: a conservative default confidence threshold (0.75), a visible Unassigned Queue in the Feed for low-confidence items, and a one-tap correction path in the side panel that feeds back into the model.

---

## Decision: Block association confidence threshold at 0.75

**Options considered:**
- High threshold (0.90+): fewer auto-assignments, more items in the Unassigned Queue
- Moderate threshold (0.75): balanced auto-assignment with a meaningful confidence bar
- Low threshold (0.60 or below): more auto-assignments, higher wrong-association rate
- User-configurable with no default

**Decision:** 0.75 default, user-configurable per their tolerance.

**Rationale:** The threshold is not a technical parameter. It is a product decision about where to put the burden of uncertainty. At 0.90, the system routes too many messages to the Unassigned Queue, which means the user spends time managing a queue instead of doing work. The promise of AI-first association isn't kept. At 0.60, the wrong-association rate climbs to the point where users learn to distrust the system and start checking manually, which is worse than no AI at all.

0.75 is calibrated at the point where wrong associations are rare enough that the occasional correction costs less than the queue management a higher threshold would create. The model errs toward delivery (auto-assign) over caution (route to queue) because a correctable wrong assignment is less expensive than a queue the user has to process.

The threshold is user-configurable because the right answer is genuinely different for different users. A user who reviews their Unassigned Queue quickly and prefers control can lower it. A user who trusts the AI and wants minimal friction can raise it. The default serves the median case; the setting handles the edges.

**Tradeoff:** At 0.75, some wrong auto-assignments will happen silently. The user may not notice immediately. The mitigation is making corrections easy (one tap), visible (the reassign option is always in the side panel), and lightweight (corrections feed the model). This is a deliberate bet that ease of correction is more valuable than avoidance of all errors.

---

## Decision: Reply with AI uses five layers of context, not just the current message

**Options considered:**
- Single-Cue context: AI reads only the message being replied to
- Signal context: AI reads the current conversation thread
- Block context: AI reads everything in the Block — all Signals, decision log, documents, tickets

**Decision:** Five layered context sources, applied in order of specificity: the Cue, the Signal, the decision log, the linked Jira ticket, and the full Google Doc body.

**Rationale:** A reply that is contextually correct but historically wrong is worse than no suggestion at all. Consider: a developer asks "should we use 24-hour token expiry for admin users?" A model with only the current Cue might draft "yes, 24-hour expiry sounds reasonable." A model with the decision log knows the team decided two weeks ago to exempt admin users from the 24-hour limit, and drafts "we agreed on Feb 28 to exempt admin users from the 24-hour limit. Should we revisit that decision?" The second reply is the product. The first reply is a liability.

Each layer serves a distinct purpose:

| Layer | Why it's included |
|---|---|
| The Cue | What is being replied to |
| The Signal | Conversation history — prevents contradiction or repetition |
| Decision log | Prior outcomes — prevents suggesting something already decided |
| Jira ticket | Technical ground truth for engineering-related replies |
| Google Doc body | Written context for document-related replies |

The layering is ordered by specificity because the most specific context should dominate. The AI is instructed to treat the decision log as a constraint, not just context. It cannot contradict a logged decision.

**Tradeoff:** More context means slower generation and a larger context window on every Reply with AI call. Mitigated by the block_context cache (doc body and ticket descriptions are pre-fetched and stored, not fetched live), which means the context assembly is fast even for large documents. The latency cost is accepted because a fast wrong reply is worse than a slightly slower right one.

---

## Decision: Tasks are Block-level, not Signal-level

**Options considered:**
- Signal-level tasks: a Task is attached to the Signal that triggered it; resolving the Signal removes the Task
- Block-level tasks: Tasks live in the Block independently of any Signal, and persist after Signals resolve
- Separate task system: Tasks are a standalone feature not connected to Signals or Blocks at all

**Decision:** Block-level tasks, coexisting with Signals but independent of them.

**Rationale:** A Signal has a natural lifecycle: it starts when someone raises something and ends when it's resolved. A Task is about intent: something someone has decided to do regardless of what conversation prompted it. Attaching a Task to a Signal would mean the Task disappears when the Signal resolves, which is wrong. "Update the error state section in the Technical Brief" is still a real piece of work whether or not the Signal that surfaced it has been closed.

The Block is the right home because it is the persistent unit of work. Signals come and go; Blocks persist for the lifetime of a project. A Task that belongs to a Block will still be there when the relevant Signal is resolved, when new Signals are created, and when the Block is reviewed in a weekly sync.

The separate system option was ruled out because disconnecting Tasks from the rest of FocusBlock's context would prevent the AI from referencing open Tasks in Feed summaries, from suggesting Tasks based on doc content, and from surfacing Task-to-Signal relationships in the future.

**Tradeoff:** In V1, Tasks and Signals are fully independent — completing a Task doesn't resolve a Signal, and resolving a Signal doesn't complete a Task. This means a user could resolve a Signal and leave a related Task open without any prompt. The mitigation in V1 is the AI check: when a Doc comment Signal is resolved, FocusBlock asks "is the linked task also complete?" Cross-linking between Tasks and Signals is deferred to V2 when usage patterns make the right relationship model clearer.

---

## Decision: Cache full document body in block_context rather than fetching live

**Options considered:**
- Live fetch: fetch the Google Doc or Jira ticket description every time Block chat is opened
- Summary cache: fetch once, have the AI summarize the document, store the summary
- Full-content cache: fetch the full document body on link, refresh in background, use live fallback

**Decision:** Full-content cache with background refresh and live fallback.

**Rationale:** Live fetch fails on three dimensions. It is slow (an API call on every chat open), expensive (API quota consumed on every interaction), and unreliable (a large document may exceed the response time budget or hit rate limits mid-conversation). More importantly, it means Block chat performance degrades as documents get longer — the inverse of what users need.

The summary cache option was ruled out because it destroys the value of having document access at all. If a user asks "what does section 3 of the Technical Brief say about timeout error handling?" a pre-summarized version of the document won't have that. Summaries compress and discard. The AI needs the full text to answer specific, reference-style questions, which are exactly the questions users ask when they are in a Block work session.

The full-content cache solves all three live-fetch problems while preserving the full text. The database trigger on `block_sources` INSERT means the content is populated before the user ever opens chat — there is no cold-start latency. The 60-minute background refresh keeps it current throughout the workday without user-triggered fetches. The live fallback handles the edge case where the cache is stale at the moment a user opens chat.

**Tradeoff:** The cache can be stale. A document edited 45 minutes ago might be answered based on the version from 60 minutes ago. This is acceptable for the target use case: users doing sustained async work where a 60-minute documentation lag is not a blocker. It would not be acceptable for real-time collaborative editing, which is explicitly out of scope. The live fallback reduces the staleness window further for active chat sessions.

---

## Decision: Signals are reactive (AI-created); Tasks are proactive (user-created or AI-suggested)

**Options considered:**
- Single model: one type of work item created by both users and AI
- Two separate models: Signals for reactive AI-detected activity; Tasks for proactive user intent
- Jira-only tasks: don't build a native task system; rely on Jira for all task tracking

**Decision:** Two distinct models, Signals and Tasks, that coexist in the same Block without merging.

**Rationale:** The distinction is not cosmetic; it reflects a fundamentally different relationship between the item and the user's agency. A Signal is something that happened to you — the AI detected activity and surfaced it. A Task is something you decided to do. Conflating them into one model produces a Feed where AI-detected conversations and user-declared intentions look and behave the same way, which makes it harder to process either effectively.

The Jira-only option was ruled out because it creates a hard dependency on Jira for a feature that all target users need, including those who don't use Jira. It also routes users out of FocusBlock to create tasks, which breaks the attention-layer model. If the user has to switch to Jira to track intent that arose in a FocusBlock session, FocusBlock is just another notification hub.

The two-model design also enables AI-suggested Tasks: the AI can read a Google Doc comment, infer an untracked action item, and surface a Task suggestion card, a proactive item that originated from reactive detection. This is only possible if Tasks and Signals are separate concepts with separate creation paths.

**Tradeoff:** Two models means two mental models for the user to hold. "Is this a Signal or a Task?" is a question users will ask, especially early. The mitigation is making the distinction viscerally clear in the UI: Signals come from incoming activity and have a source drawer showing the original messages; Tasks are created with a `+ Add Task` button and look like a lightweight to-do. The conceptual names ("Signal" = something that happened; "Task" = something to do) are chosen to make the distinction intuitive rather than requiring explanation.
