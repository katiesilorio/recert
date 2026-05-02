# FocusBlock

**An AI attention layer that sits on top of your existing tools, reading everything, surfacing only what matters, and handing you full context before you ever have to respond.**

| Field | Value |
|---|---|
| Status | Draft — v2.0 |
| Author | Katie |
| Date | March 2026 |
| Version | Version A |
| Primary Users | Project-based contributors doing sustained, async, collaborative work at 10–200 person companies |

---

## Core Concepts

FocusBlock uses three named concepts that appear throughout this document. They are defined here precisely to avoid ambiguity, and their names are intentional. They form a coherent vocabulary that reflects the product's focus-first philosophy.

| Concept | Definition |
|---|---|
| **Block** | The highest-level unit of organization. A Block represents a project, initiative, or ongoing area of work with a lifespan of days to weeks. For example, "Checkout Redesign" or "Auth Flow Bug." A Block is the container everything else lives inside. It has a name, members, a decision log, connections to tools (Slack channels, Jira tickets, Google Docs), and a Task list. The Feed shows one card per Block. |
| **Signal** | A discrete conversation or thread of activity within a Block: one question, one decision, one issue being worked through. A Signal has a beginning (someone raises something) and an end (it gets resolved). A single Block can have many active Signals at once. A Signal can span multiple tools: a Slack thread and a Jira comment about the same bug are one Signal. Signals appear as individual sections in the side panel when a Block's Feed card is opened. |
| **Cue** | The atomic unit: a single raw message, comment, or notification from a connected tool. A Slack message is a Cue. A Jira comment is a Cue. A Google Doc comment or suggested edit is a Cue. Cues are the raw material the AI reads to form Signals. They appear when a Signal's source drawer is expanded in the side panel or Block view. |

### How they nest

**Block** contains **Signals**. **Signals** are made of **Cues**. The Feed shows one card per Block. The side panel shows the Signals within that Block. The drawer inside each Signal shows the raw Cues. The AI reads Cues, groups them into Signals, and summarizes Signals into the Block's Feed card.

---

## 1. The Problem: Context Switching Is a Tax on Deep Work

Most teams today coordinate across at least four tools: Slack for messages, Jira for tasks, Google Docs for written work, and Gmail for external communication. Each tool sends its own notifications. Each notification arrives on the sender's timeline, not the recipient's. Every one forces the same expensive cognitive sequence:

1. Stop current work
2. Read the message
3. Mentally locate which project, decision, or thread this belongs to
4. Determine whether action is required right now
5. Respond, defer, or ignore
6. Attempt to reload the previous mental context and restart

Research estimates it takes an average of **23 minutes** to fully regain deep focus after an interruption. For someone receiving 50+ messages a day across these tools, uninterrupted deep work is not degraded, it is structurally impossible.

### The Root Cause

Context switching is not caused by too many conversations. It is caused by conversations being spread across tools that do not talk to each other, arriving at random times, requiring the recipient to reconstruct the full picture from scratch on every single interaction.

---

## 2. What FocusBlock Is

FocusBlock is an AI attention layer that sits on top of your existing tools. It does not replace Slack, Jira, Google Docs, or Gmail. It connects to them, reads everything across all of them, and becomes the single surface where your team's communication is organized, triaged, and acted on.

The core organizing unit is a **Block**: a shared container that groups all communication and work artifacts related to a specific project, feature, or initiative, regardless of which tool they originated from. A Slack thread, a Jira ticket, a Google Doc, and a Gmail reply - if they are all about the same thing, they live in the same Block.

Blocks are shared with the teammates you invite. When you create a Block, you can spawn the tools you need: a Slack channel, a Jira ticket, a Google Doc, directly from FocusBlock. Everything created this way is automatically associated with the Block from day one, with no tagging required. The AI handles everything else, reading your existing tools and associating activity to the right Block in the background.

### The Two-Sentence Pitch

> Most tools organize your work. FocusBlock organizes your attention. It reads every message across your entire stack, decides what can wait, and when something does need you, it surfaces it with full context pre-loaded so you never have to reconstruct before responding.

---

## 3. Blocks: The Core Organizing Unit

A Block is a named, shared container that aggregates all communication and work artifacts related to a single project, feature, initiative, or decision, across every connected tool. It is the unit FocusBlock uses to hold context, triage messages, and protect attention.

### What a Block Contains

- A name, description, and status (active, paused, archived), set by the creator
- Members — teammates explicitly invited by the creator
- All Slack messages and threads associated with this Block
- All Jira tickets linked to this Block, including title, description, acceptance criteria, status, assignee, and comments
- All Google Docs linked to this Block, listed with title, last edited date, and open comment count. Google Doc body text is cached in full and available to the AI for context, summaries, and chat. Clicking opens the doc in Google Docs
- All Gmail threads tagged to this Block
- A running AI-generated decision log: key outcomes from past conversations, one sentence each, updated automatically
- Open items: unresolved questions or threads still waiting for action

### Jira Context — Not Just Comments

FocusBlock pulls the full Jira ticket: title, description, acceptance criteria, priority, assignee, status, and linked tickets, not just comments. When the AI surfaces a message related to a Jira ticket, it prepends the full ticket context, not just the thread. This means a teammate asking a question about a bug gets the bug description, current status, and prior comments in a single view — before they type a word.

### Block Visibility & Membership

- Blocks are private by default — visible only to the creator and explicitly invited members
- The creator invites teammates by name or email during setup or at any point afterward
- All members see the same Block: the same context, the same decision log, the same open items
- Each member's digest and triage is personalized: the AI surfaces items relevant to that person based on their role in the Block's conversations
- Members can be removed by the creator at any time; removed members lose access to the Block immediately

### How Blocks Are Created

Creating a Block is the starting point for any new project or initiative. The creator names the Block, invites members, and then uses FocusBlock to spawn the tools the team will need, rather than creating those tools separately and trying to connect them after the fact.

### Spawning Tools from a Block

From inside any Block, a user can create:

- A new **Slack channel** — created in the team's Slack workspace, automatically associated with the Block
- A new **Jira ticket or epic** — created in the connected Jira project, automatically associated with the Block
- A new **Google Doc** — created in the team's Google Drive, automatically associated with the Block
- A new **Email** - created in the user's email client, automatically associated with the block

Everything spawned from a Block is associated automatically and immediately. No tagging, no linking, no manual setup. This is the lowest-friction path to a fully connected Block and the recommended way to start any new project.

> **Why this matters:** The hardest part of any context system is the cold start: getting the initial associations right before there is enough history for AI to learn from. Spawning tools from a Block bypasses the cold start entirely. The connection is established at creation, not inferred after the fact.

---

## 4. How Messages Get Associated With Blocks

FocusBlock uses an **AI-first association model**. The AI reads all activity across connected tools and assigns messages to Blocks automatically in the background. No tagging required from users. Manual association is available as a secondary option for edge cases. A structured queue handles everything the AI cannot confidently assign.

### The Three Association Tiers

**Tier 1 — Automatic: Spawned from a Block (highest confidence)**

Any Slack channel, Jira ticket, Google Doc, or email created directly from within a Block is permanently and automatically associated. No AI inference needed. The connection is structural. All activity in that channel, ticket, doc, or email is associated with the Block for its entire lifetime.

**Tier 2 — AI Inference (background, no user action required)**

For all other activity across connected tools, the AI reads incoming messages and assigns them to the most likely Block based on:

- Topic and keyword similarity to the Block's name, description, and existing content
- Participant overlap: messages involving the same people as an active Block are weighted toward that Block
- Linked artifacts: a Slack message referencing a Jira ticket already in a Block is auto-assigned to that Block
- Temporal patterns: activity clustering around the same period as a Block's active conversations

When the AI's confidence exceeds the assignment threshold, the message is silently associated with the Block and surfaced in that Block's digest for relevant members. No user action required.

**Tier 3 — Unassigned Queue (low AI confidence)**

When the AI cannot confidently assign a message to any existing Block, it holds the message in a per-user **Unassigned Queue** rather than guessing. The Unassigned Queue is a dedicated section in FocusBlock, visible alongside the user's Blocks.

Each unassigned Cue shows: the message content, its source tool, the AI's best-guess Block (if any), and a confidence label. The user can:

- Assign it to an existing Block with one tap
- Accept the AI's suggested Block
- Dismiss it as not relevant
- Create a new Block from it, if the message represents something genuinely new

---

## 5. The Unified Feed: One View for Everything

FocusBlock has a single primary view: **the Feed**. There is no separate inbox section, no separate queue screen, no separate Block list. Activity from active Blocks, unassociated messages, Unassigned Queue items, and AI-suggested new Blocks appears in one prioritized, scrollable feed.

At the very top of the Feed, locked in place above the scrollable content, is the **Feed-level AI chat prompt**. It is always visible regardless of scroll position. Below it, the Feed cards begin.

### Feed Ordering

Items in the Feed are sorted by two dimensions in order:

1. **Urgency first**: blocking items always appear at the top, followed by action required, followed by FYI
2. **Block priority second**: within the same urgency tier, items from higher-priority Blocks surface before lower-priority ones. Block priority is AI-suggested based on recent activity, urgency patterns, and signal density — and is always overridable by the user

Unassigned items are interleaved at their triaged urgency level, labeled "Unassigned" so they are visually distinct but not buried.

> **Block Priority as a Work Session Model:** Block priority is not just a sort order. It is how a user structures their day. High-priority Blocks contain the work that defines the day. By reviewing all high-priority Block activity before touching normal-priority Blocks, users spend focused sessions on what matters most rather than bouncing between every incoming message in arrival order. The AI keeps priority current as Block activity evolves; the user corrects it when the AI gets it wrong.

### One Feed Item Per Block

The Feed shows at most **one item per Block** at any given time. No matter how much activity a Block has (three Slack clusters, two Jira comments, an open Doc annotation) it surfaces as a single card. The AI's job is to read everything happening across all sources in the Block and write a short paragraph that captures the meaningful state: what is unresolved, what requires action, and what the connective thread is across the activity.

The summary is intentionally consolidated. If two Slack clusters and a Jira comment are all pointing at the same underlying issue (a release being blocked) the summary says that once, not three times. The AI finds the connective tissue and reflects it, rather than listing each item mechanically.

**Example Feed Item:**
```
Auth Flow Bug  ·  3 Signals  ·  2 require action

"QA flagged they can't reproduce the staging bug and Liam is waiting on
sign-off before the release can move forward. Marcus raised a timeline
concern in a DM. Your approval is needed in Jira."
```

Tapping the Feed item opens the side panel, where the activity separates into individual Signals — each with its own sub-summary and reply composer. The Feed summary is a signal about what is happening in the Block; the side panel is where the user acts on it.

### Slack Clustering: How Conversation Boundaries Are Detected

Slack conversations do not have clean Signal boundaries. A single channel may contain dozens of messages spanning several unrelated Signals. FocusBlock handles this with a two-tier clustering model:

1. **Threads first** — a Slack thread (replies to a specific message) is treated as a discrete Signal. This is the primary and highest-confidence boundary.
2. **AI clustering for top-level messages** — consecutive top-level messages without threads are read by the AI and grouped into Signals based on semantic similarity, participant overlap, and temporal proximity. Each cluster is treated as a single Signal, equivalent to a thread.

When the AI's confidence in a cluster boundary is low, it surfaces the proposed grouping as a suggestion, the user confirms or adjusts the boundary once, and the AI uses that confirmation as a training signal for future messages in that channel.

DM and group DM clusters are handled identically to channel clusters, but are labeled with a **DM indicator** so the user always knows the source is a private conversation, not a shared channel.

> **Why this matters:** Without clustering, a busy Slack channel produces dozens of Feed signals from a single Block. With clustering, that same channel produces three or four named Signal groups, each collapsed to a summary, each independently actionable. The channel stops being a firehose and starts being a structured list of conversations.

### Unassigned Items in the Feed

Activity that the AI cannot confidently assign to any Block appears in the Feed as a single **Unassigned card** — consolidated the same way as Block items, with an AI paragraph summary of what arrived and why it could not be matched. From the Unassigned card the user can:

- Accept the AI's suggested Block: item is associated and the AI learns from the confirmation
- Assign to a different existing Block: one-tap from a dropdown
- Create a new Block: AI pre-fills the name based on the content
- Dismiss: removed from the Feed

### AI-Suggested New Blocks in the Feed

When the AI detects a cluster of related unassociated items representing a genuinely new Block, it surfaces a **Block suggestion card** in the Feed. The card shows the AI-generated Block name (editable), a one-sentence reason, and one-tap accept or dismiss. Accepted suggestions create the Block and associate all triggering items automatically.

> **Example:** Three Jira tickets filed by QA all reference a new payment retry system. FocusBlock surfaces a Feed card: "Suggested Block: Payment Retry — Q2 (based on 3 Jira tickets filed today by QA)." Maya edits the name, accepts, invites two engineers. Block is live in 20 seconds.

---

## 6. AI Triage: Protecting Attention by Default

Association tells FocusBlock *what* a message is about. Triage tells FocusBlock *whether and when* to surface it in the Feed. These are two separate AI jobs that run in sequence on every incoming message.

### Urgency Classification

Every incoming message, whether Block-associated or unassociated, is classified as **blocking**, **action required**, or **FYI**:

| Level | Behavior |
|---|---|
| **Blocking** | Surfaces in the Feed immediately, breaks through focus mode |
| **Action Required** | Held during focus mode, delivered in the next digest at a natural break point |
| **FYI** | Batched into a daily summary at end of day |

Urgency signals: sender role (EM, CEO, on-call), keywords ("blocking," "down," "P0," "urgent"), unanswered question threads older than 2 hours, Jira P1/P0 priority fields, and direct mentions.

### Context Injection on Every Feed Item

Every item surfaced in the Feed, whether from a Block or unassigned, is pre-loaded with context before the user reads the first word:

- Block name and current status (for Block-associated items)
- AI summary of all cross-tool activity on this Signal
- Last decision logged in this Block
- Open items currently waiting for action
- For Jira-linked items: ticket title, description, acceptance criteria, current status, assignee, and priority

The user has everything they need to act before opening the drawer. Most items can be resolved (replied to, marked done, or dismissed) without expanding anything.

### Focus Mode

- User activates Focus Mode manually; calendar sync available in V2
- FocusBlock updates the user's status in Slack: "[Name] is in focus mode until [time]."
- All action required and FYI items are held — Feed goes quiet
- Blocking items always break through with full context pre-loaded
- When focus mode ends, held items are released into the Feed in urgency + Block priority order, not a time-ordered flood

---

## 7. A Day in FocusBlock: End-to-End Workflow

*Maya is a PM at a 40-person SaaS company. Her team uses Slack, Jira, and Google Docs. She has four active Blocks: Checkout Redesign (high priority), Auth Flow Bug (high priority), Q2 Roadmap (normal), and Vendor Contract (normal).*

**8:45 AM** — Maya opens FocusBlock. One view — the Feed. At the top: the AI chat prompt. She types: *"What requires my action today?"* The AI responds: *"Two blocking items in Auth Flow Bug — Liam needs sign-off confirmation and Marcus flagged release risk. One Jira approval needed in Checkout Redesign. One Unassigned Slack message to sort."* Four Block cards match exactly.

**8:47 AM** — Maya taps Auth Flow Bug. Side panel opens. She goes to Signal 1 — QA and Liam on staging sign-off. Taps **Reply with AI** on Liam's Slack Cue. Draft: *"Hey Liam — QA confirmed they can't reproduce the original bug in staging. Should be clear for sign-off."* She sends it. Marks Signal 1 resolved. Signal 2: Marcus DM, no action, resolved. Signal 3: Jira approval — approved from the composer. Auth Flow Bug disappears from the Feed.

**8:55 AM** — She works through the remaining three Block cards. Checkout Redesign: one Signal — a design question spanning a Slack Cue and a Jira comment. Q2 Roadmap: two FYI Signals, marked read in 30 seconds. Vendor Contract: one action-required Signal, replied to. Unassigned card: one Slack message assigned to Checkout Redesign with one tap. Done in under 13 minutes.

**9:05 AM** — She accepts an AI-suggested new Block: *"Suggested Block: Payment Retry — Q2. Three Jira tickets filed today by QA all reference the same new system."* Maya edits the name, accepts, invites two engineers. Block is live with all three tickets inside it.

**9:10 AM** — Maya activates Focus Mode until noon. Feed goes quiet. All incoming messages continue to be read, clustered, and associated in the background. Nothing surfaces unless it is blocking.

**10:22 AM** — A blocking item breaks through focus mode. Auth Flow Bug reappears. AI summary: *"QA confirmed sign-off on staging. Marcus needs your Jira approval to release."* Maya approves the Jira ticket from the composer. Returns to work in under 3 minutes.

**12:00 PM** — Focus Mode ends. Four Block cards with accumulated activity. Maya uses the Feed-level chat prompt: *"Summarize what came in while I was in focus mode."* She then works through each side panel in sequence — 17 minutes total.

**2:30 PM** — Maya opens Checkout Redesign for a deliberate review session. She taps the chat icon: *"What is the current state of the error modal decision?"* The AI responds: *"Agreed Feb 28 — error states follow design system v2. One open question about timeout modal reuse remains."* She accepts an AI-suggested Task: *"Update error state section after modal decision."* Sets urgency to Action Required, links to the doc, assigns to herself. 20 minutes, Block fully current with one Task created.

**3:00 PM** — Maya starts a new project: Onboarding Redesign. She creates a new Block, invites three teammates. From within the Block: creates a Jira epic, a Slack channel, and a Google Doc — all auto-associated. Full connectivity before a single message is sent.

**5:00 PM** — End of day. Three FYI Signals remain across lower-priority Blocks. She marks them read. Decision logs updated by the AI. Auth Flow Bug: *"Release approved. QA sign-off confirmed."* Zero unresolved questions about where things stand.

---

## 8. Side Panel & Block View

FocusBlock uses a single progressive surface for acting on work, starting lightweight in the side panel and expanding into a full Block view without navigating away from the Feed.

### Layer 1 — Side Panel (quick action)

Tapping a Feed item opens a side panel to the right of the Feed. The Feed remains visible and scrollable. The side panel shows, in order from top to bottom:

1. **Block header** — name, status, last decision, and open item count
2. **AI summary paragraph** — the consolidated paragraph shown on the Feed card, with full cross-Signal context
3. **Block-level chat prompt** — scoped to this Block's full history (Signals, Cues, decision log, Jira)
4. **Signal list** — each Signal in urgency order, with a one-to-two sentence AI sub-summary and a collapsed source drawer
5. **Source drawer (per Signal)** — expands to show individual Cues in their original form, labeled by tool and source
6. **Reply composer (per Cue)** — inline text box posting natively back to the originating tool, with a **Reply with AI** button
7. **Resolve button (per Signal)** — marks the Signal resolved; moves to the bottom visually deprioritized but not hidden
8. **Quick actions** — reassign the Block, create a follow-up Jira ticket, or dismiss a Signal

> **Reply with AI:** When a user taps Reply with AI below a Cue's text box, the AI reads the full Signal history, the Block's decision log, and the linked Jira ticket context to generate a relevant draft. The draft populates the text box directly — the user edits if needed and sends. The reply posts natively to the originating tool. This is not a chat interaction — it is in-context drafting at the Cue level, designed to eliminate the cognitive load of composing a reply from scratch.

**Example:**
```
Maya taps the Auth Flow Bug Feed card. Side panel shows the AI summary paragraph,
then a chat prompt, then three Signals.

She asks the chat: "What was the last decision made about the auth token expiry?"
AI responds: "On Feb 28 the team agreed to limit token expiry to 24 hours for all
non-admin users."

She reads Signal 1 — QA and Liam asking about staging sign-off. Expands the drawer,
taps Reply with AI on Liam's Slack Cue. AI drafts: "Hey Liam — QA confirmed they
can't reproduce the original bug in staging. We should be good to sign off."
She sends it. Marks Signal 1 resolved. Two more Signals cleared in under 2 minutes.
```

### Layer 2 — Full Block View (deliberate review)

From the side panel, the user can expand to the full Block view with one tap. The panel widens to fill the right portion of the screen while the Feed collapses to a narrow rail on the left. The full Block view shows:

**Block header** — name, status, members, AI-suggested priority with override control, running decision log, and a chat icon

**Five tabs:**

| Tab | Contents |
|---|---|
| **All** | Every Signal in this Block sorted by status (requires action first) then date, using the AI sub-summary + drawer pattern |
| **Slack** | All Signals sourced from Slack channels or DMs. DM Signals are labeled with a DM indicator. Each Signal shows its AI sub-summary, raw Cues, and inline reply composer with Reply with AI. |
| **Jira** | All tickets and epics linked to this Block. Each shows title, description, acceptance criteria, status, assignee, and full comment thread with inline reply composer and Reply with AI. |
| **Docs** | All Google Docs linked to this Block. Each row shows the document title, last edited timestamp, and open comment count. Clicking opens the doc in Google Docs in a new tab. |
| **Chat** | A full chat interface scoped to this Block with persistent session history. Same AI capabilities as the Block-level chat prompt. |

**Sort and filter controls** appear at the top of every tab except Chat:

- **Status** — Requires action / Unresolved / Resolved (default: requires action first)
- **Date** — Newest first / Oldest first
- **Source** — All / by channel or ticket / DMs only (Slack tab only)

> **Chat icon vs Chat tab:** The chat icon in the Block header opens a chat panel that floats alongside whichever tab the user is on, so they can ask a question while staying in the Jira tab, without navigating away. The Chat tab is for longer, more deliberate AI conversations where the full screen width is useful.

> **Why tabs, not a single scroll:** Each source type has a different interaction model. Slack is conversational and thread-based, Jira is structured and ticket-based, Docs are annotation-based. Tabs let a user run a focused session on one source type. Mixing everything in one scroll forces constant context switching within the Block itself: the exact problem FocusBlock is designed to eliminate.

The full Block view is designed for **intentional review sessions**, not reactive triage. A user processes the Feed using the side panel in the morning, then opens a high-priority Block's full view in the afternoon for a deliberate work session through one source tab at a time.

---

## 9. Tasks

Tasks are proactive, user-created work items that live inside a Block. They are distinct from Signals, which are reactive, created by the AI from incoming tool activity. **A Signal is something that happened to you. A Task is something you have decided to do.**

Tasks and Signals coexist in the same Block but are visually and conceptually separate. In the side panel and full Block view, Tasks appear in their own section below the Signal list — not mixed in with incoming activity.

### Task Structure

| Field | Options |
|---|---|
| Title | What needs to be done |
| Urgency | Blocking / Action Required / FYI |
| Assignee | Any Block member |
| Due date | Optional |
| Status | Todo / In Progress / Done |
| Linked Doc | Optional link to a Google Doc associated with this Block |

*In V1, Tasks can only link to Google Docs. Jira ticket linking and cross-Block task references are out of scope.*

### Creating Tasks

**Manually** — a `+ Add Task` button appears at the top of the Tasks section in both the side panel and the full Block view. A lightweight inline form collects title, urgency, assignee, due date, and optional Doc link.

**AI-suggested** — when the AI detects an actionable item in Block activity that doesn't map cleanly to an existing Signal or Jira ticket, it surfaces a Task suggestion card below the Signal list. Example: *"Suggested task: Update error state section in Technical Brief — linked to Checkout Redesign doc."* The user accepts, edits, or dismisses.

### Tasks in the Feed

Tasks with Blocking or Action Required urgency contribute to the Block's Feed card urgency indicator, the same way Signals do. The AI Feed summary paragraph may reference open Tasks when they are relevant context. Tasks with FYI urgency do not affect the Feed card.

### Tasks and Signals — V1 Boundary

In V1, Tasks and Signals are independent. Completing a Task does not resolve a Signal, and resolving a Signal does not complete a Task. Cross-linking between Tasks and Signals is a V2 consideration.

### Google Docs and Tasks

When a Task has a linked Doc:

- The Task row in the side panel and Block view shows the Doc title as a clickable link: opens the Doc in Google Docs in a new tab
- The Docs tab in the full Block view shows a Task count badge next to each linked Doc (e.g. *"2 open tasks"*)
- When a Doc comment is resolved in FocusBlock, the AI checks whether any open Task is linked to that Doc and surfaces a prompt: *"The comment on Technical Brief was resolved. Is the linked task complete?"*

> **Why Tasks are Block-level, not Signal-level:** A Signal has a natural lifecycle. It starts when someone raises something and ends when it's resolved. A Task is about intent: something that needs to happen regardless of what conversation triggered it. Attaching Tasks to Signals would make them disappear when the Signal resolves, which is the wrong behavior. Tasks belong to the Block because the Block is the persistent unit of work.

---

## 10. AI Chat

FocusBlock embeds AI chat at two levels — the Feed and the Block. Both use the same conversational interface but with different scopes of context. Chat is designed for **orientation, retrieval, and drafting**, not for replacing the Signal-based triage workflow.

### Feed-Level Chat

The Feed-level chat prompt is locked at the top of the Feed, always visible above the scrollable Block cards. It is scoped to all of the user's Blocks.

**What the Feed-level AI can do:**
- Summarize what happened today or this week across all Blocks
- Surface what requires action right now - a curated list of the highest-priority open Signals across all Blocks
- Search across all Blocks for a topic, decision, or person
- Answer questions about specific Blocks by name

> **Example:** Maya opens FocusBlock on Monday morning after a long weekend. Before scrolling the Feed she types: *"What happened while I was out?"* The AI responds with a two-paragraph summary: which Blocks had activity, what was resolved, and what is now waiting for her action. She has full orientation in 30 seconds before touching a single Feed card.

### Block-Level Chat

Block-level chat is scoped to a single Block and has access to its complete history: all Signals, all Cues, the decision log, linked Jira ticket descriptions and comments, and full Google Doc content including comment threads.

It appears in two places:
- **In the side panel** — a chat prompt between the AI summary paragraph and the Signal list, always visible when the side panel is open
- **In the full Block view** — a chat icon in the Block header opens a floating chat panel; a dedicated Chat tab is available for longer conversations

**What the Block-level AI can do:**
- Answer questions about the Block's history
- Summarize Block activity over a time period
- Surface what requires action in this Block
- Draft replies to Signals via the **Reply with AI** button (available at the Cue level inside each Signal's source drawer, not in the chat interface itself)

> **Context boundary:** Block-level chat cannot access other Blocks. This is intentional. It ensures the AI's answers are grounded in the specific Block's history and cannot hallucinate connections to unrelated work. For cross-Block questions, the user uses the Feed-level chat prompt.

### What AI Chat Is Not

AI chat is not a replacement for the Feed or the Signal triage workflow. The Feed is the primary surface for processing work. It is optimized for speed, context injection, and resolution. Chat is for questions the Feed cannot anticipate: retrieving a specific past decision, getting oriented after time away, or understanding the state of a Block before a meeting.

---

## 11. Integrations

FocusBlock is an integration-first product. All messages, comments, and documents live in the originating tool. FocusBlock reads and writes via official APIs and OAuth.

| Integration | Capabilities at Launch |
|---|---|
| **Slack** (P0) | Read: all messages in connected channels and DMs. Write: post messages and replies to threads. Actions: Slack shortcut to manually tag any message, thread, or channel to a Block. Spawn: create a new Slack channel directly from a Block. Notifications: post focus mode status on profile. |
| **Google Docs** (P0) | Read: document title, content, last edited timestamp, open comments, and (optionally) suggested edits. Write: post replies to comments, resolve comments. Signals: comments and suggested edits (toggleable via Settings: *"Create Signals from suggested edits"*). Spawn: create a new Google Doc directly from a Block, auto-associated. |
| **Jira** (P1) | Read: full ticket data: title, description, acceptance criteria, priority, status, assignee, linked tickets, and all comments. Write: post comments on Jira tickets. Actions: manually link a ticket or epic to a Block. Spawn: create a new Jira ticket or epic directly from a Block, auto-associated. Sync: ticket status changes update Block open items automatically. |
| **Gmail** (P2) | Read: email threads. Write: reply to email threads. Actions: manually tag an email thread to a Block. Lower priority: internal team communication is the primary V1 use case. |

> **API Dependency Risk:** FocusBlock Version A is entirely dependent on third-party API access. Slack, Google, and Atlassian control these APIs and can restrict or reprice access at any time. This is the primary strategic risk of the integration layer model. Mitigation: maintain developer platform relationships, monitor API ToS actively, and architect for swappable integration adapters.

---

## 12. Core Features — MVP

| Feature | How It Eliminates Context Switching | Priority | Phase |
|---|---|---|---|
| Blocks | Shared containers that aggregate all cross-tool communication for a project. Context is always present, no reconstruction. | P0 | Phase 1 |
| Unified Feed | One prioritized view for everything: Block activity, unassigned items, and AI-suggested Blocks. Sorted by urgency then Block priority. No separate inbox. | P0 | Phase 1 |
| One Feed item per Block | No matter how much activity a Block has, it surfaces as one card. The AI writes a consolidated paragraph summary across all Signals and sources. | P0 | Phase 1 |
| Slack Signal clustering | Threads are the primary Signal boundary. Top-level messages without threads are AI-clustered into Signals. DM clusters are labeled as DMs. Low-confidence boundaries surface as suggestions. | P0 | Phase 1 |
| Side panel Signal list | Tapping a Feed item opens the side panel showing each Signal individually: AI sub-summary, source drawer, per-source reply composers, and resolve button. | P0 | Phase 1 |
| Full Block view (tabbed) | Expand the side panel to a full Block view with tabs: All, Slack, Jira, Docs. Each tab shows source-specific activity with inline reply composers. | P0 | Phase 1 |
| Feed-level AI chat | Locked prompt at the top of the Feed, scoped to all Blocks. Summarizes across Blocks, surfaces what requires action, and searches Block history. | P0 | Phase 1 |
| Block-level AI chat | Chat prompt in the side panel and chat icon/tab in the full Block view. Scoped to one Block's full history. | P0 | Phase 1 |
| Reply with AI | Button below each Cue's reply text box. Generates a context-aware draft using Signal history, Block decision log, and Jira context. | P0 | Phase 1 |
| Spawn tools from a Block | Create Slack channels, Jira tickets, and Google Docs directly from a Block. Auto-associated from creation — zero tagging required. | P0 | Phase 1 |
| AI-first Block association | AI reads all tool activity and silently associates messages with the correct Block in the background. | P0 | Phase 1 |
| Unassigned Queue (in Feed) | Low-confidence items surface in the Feed labeled Unassigned. User assigns, accepts AI suggestion, or creates a new Block. | P0 | Phase 1 |
| AI-suggested new Blocks | AI detects clusters of unassociated activity and suggests a new Block directly in the Feed. | P0 | Phase 1 |
| Full Jira context injection | Surfaces ticket title, description, acceptance criteria, status, and assignee — not just comments — on every Jira-linked item. | P0 | Phase 1 |
| Full Google Doc context | When a Google Doc is linked to a Block, FocusBlock fetches and caches the full document body. The AI uses this content for Block-level chat, Signal summaries, and Reply with AI drafts — not just comments. Doc content is cached on link and refreshed in the background every 60 minutes with a live fallback if stale. | P0 | Phase 1 |
| AI Urgency Triage | Blocking / action required / FYI classification for every message. Non-urgent items never interrupt a focus session. | P0 | Phase 1 |
| AI Context Injection | Every Feed item prepends Block context, last decision, open items, and Jira ticket details. Zero reconstruction before responding. | P0 | Phase 1 |
| Cross-tool reply | Replies sent in FocusBlock post natively back to the originating tool (Slack, Jira, Docs). | P0 | Phase 1 |
| Tasks | Block-level proactive work items with urgency, assignee, due date, status, and optional Google Doc link. AI-suggested and manually created. | P0 | Phase 1 |
| Focus Mode | User-declared deep work blocks. Non-blocking items held. Team notified via Slack automatically. Feed stays quiet. | P0 | Phase 1 |
| Decision Log | AI maintains a running log of resolved outcomes per Block. Newcomers get current in under 30 seconds. | P1 | Phase 1 |
| Manual tag to Block | Slack shortcut to manually assign any message, thread, or channel to a Block. Fallback when AI confidence is low. | P1 | Phase 1 |
| Jira integration | Full ticket read/write, status sync, spawn from Block. | P1 | Phase 2 |
| Calendar sync for Focus Mode | Auto-detect focus blocks from Google or Outlook calendar. | P2 | Phase 2 |
| Gmail integration | Read, reply, and tag email threads to Blocks. | P2 | Phase 3 |
| AI confidence tuning | Per-user threshold controls for when AI auto-assigns vs. routes to Unassigned. | P2 | Phase 3 |

---

## 13. Goals & Success Metrics

| Metric | Target at 90 Days Post-Adoption |
|---|---|
| Context switches per user per day | Reduced by ≥ 40% (activity log + session survey) |
| Time to respond to a non-urgent message | < 5 minutes average from Feed delivery to reply sent |
| Time to get current on a Block cold | < 30 seconds via decision log and context injection |
| Blocking items missed during focus mode | Zero — triage must not suppress genuinely blocking items |
| Slack usage (bridge connected) | Reduced by ≥ 60% within 60 days |
| AI Block association accuracy | ≥ 85% of auto-assignments confirmed correct by users |
| AI suggested Block acceptance rate | ≥ 65% of suggested Blocks accepted or edited (not dismissed) |
| Unassigned items cleared within 24 hours | ≥ 80% of unassigned Feed items resolved same day |
| Focus Mode sessions per user per week | ≥ 3 sessions per user within 30 days |
| Focus satisfaction NPS | ≥ 50 on in-app quarterly pulse |

---

## 14. Target Users

FocusBlock is built for a specific profile — not all office workers, and explicitly not teams whose work is interrupt-driven by design.

**The target user has three defining characteristics:**

1. Their work spans days or weeks on a single problem: an engineer building a feature, a designer iterating on a prototype, a PM writing a spec. The quality of their output depends on sustained, uninterrupted focus.
2. Their collaboration is asynchronous. They need input from others, but not in real time. A 4-hour delay on a non-blocking question costs nothing. A 4-hour interruption mid-deep work costs the whole afternoon.
3. Prior context materially affects their current work. When they return to a task, they need to know what was decided last time — not just what the ticket says today.

**In practice:** engineers, product managers, designers, analysts, writers, and strategists at companies between 10 and 200 people.

### Who This Is NOT For

Customer support agents, sales reps, and operations coordinators whose job is rapid switching between many short, independent interactions. For those roles, speed of response is the primary value and FocusBlock's batching model would hurt performance. Companies over 200 people are out of scope for V1 due to compliance, SSO, and data access complexity.

---

## 15. Risks & Mitigations

| Risk | Mitigation |
|---|---|
| Slack / Google / Atlassian restrict or reprice API access | Core strategic risk. Mitigation: maintain developer platform relationships, monitor API ToS actively, and architect for swappable integration adapters from day one. |
| AI auto-assigns a message to the wrong Block — erodes trust | Conservative confidence threshold in V1. Misassignments are easy to correct (one tap to reassign in the side panel). User feedback loop retrains the model. Accuracy is a core KPI. |
| AI suppresses a blocking item — user misses something critical | Triage errs toward delivery in V1. Urgency thresholds tunable per Block. Override always available. SLA: zero P0 suppressions in first 90 days. |
| Feed becomes overwhelming — just a prettier version of Slack | Strict grouping by Signal means 10 related messages appear as 1 Feed item. Feed is capped at showing 15 items at a time; older non-urgent items collapse automatically. |
| Spawning tools from a Block creates Slack/Jira clutter | Spawned channels and tickets are labeled with the Block name. Archiving a Block optionally archives its spawned channels and tickets. |
| Block membership gets stale — wrong people seeing wrong things | Creators are reminded monthly to review active Block membership. Inactive members (no activity in 30 days) are flagged for review, not auto-removed. |

---

## 16. Open Questions

1. **Confidence threshold calibration:** What is the right default auto-assignment confidence cutoff? Too low and the unassigned queue fills the Feed. Too high and the AI makes wrong groupings. How do we tune this during beta?

2. **Block granularity:** Should a Block map 1:1 to a Jira epic, or can Blocks cut across multiple epics? What happens when a Jira ticket is relevant to two Blocks?

3. **Cross-tool grouping logic:** How does the AI determine that a Slack message and a Jira comment are about the same Signal? What signals — shared participants, linked ticket IDs, keyword overlap — drive this, and what is the false-positive tolerance?

4. **Retroactive association:** When a user creates a new Block, should FocusBlock scan prior Slack and Jira history to backfill context, or only surface activity going forward?

5. **AI decision log quality:** How does the AI know when a decision has been reached vs. when a thread is still open? What signals indicate resolution?
