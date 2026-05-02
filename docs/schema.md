# Database Schema
Note: this doc was AI generated

FocusBlock uses Supabase (PostgreSQL) as its backend. All tables have Row Level Security (RLS) enabled — users can only access data they are authorized to see. The schema is organized around the core product concepts: Blocks, Signals, Cues, and the AI processing pipeline that connects them.

---

## Table 1 — profiles

Extends Supabase's built-in `auth.users` with FocusBlock-specific user data: AI preferences, confidence threshold for Block association, and per-user settings like whether suggested edits in Google Docs should create Signals. A trigger auto-creates a profile row on signup so no manual setup is required.

```sql
CREATE TABLE profiles (
  id           UUID PRIMARY KEY REFERENCES auth.users(id) ON DELETE CASCADE,
  display_name TEXT,
  avatar_url   TEXT,
  ai_tone      TEXT DEFAULT 'professional' CHECK (ai_tone IN ('professional','casual','concise')),
  ai_confidence_threshold FLOAT DEFAULT 0.75,
  focus_duration_default INTEGER DEFAULT 60,
  google_suggested_edits_signals BOOLEAN DEFAULT TRUE,
  created_at   TIMESTAMPTZ DEFAULT NOW(),
  updated_at   TIMESTAMPTZ DEFAULT NOW()
);

-- Auto-create profile on signup
CREATE OR REPLACE FUNCTION handle_new_user()
RETURNS TRIGGER AS $$
BEGIN
  INSERT INTO profiles (id, display_name)
  VALUES (NEW.id, NEW.raw_user_meta_data->>'full_name');
  RETURN NEW;
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;

CREATE TRIGGER on_auth_user_created
  AFTER INSERT ON auth.users
  FOR EACH ROW EXECUTE FUNCTION handle_new_user();
```

---

## Table 2 — integrations

Stores OAuth tokens for each connected tool per user. Tokens are encrypted at the application layer before insert. The `workspace_id` and `cloud_id` fields handle Slack workspace and Jira cloud identifiers respectively, which are needed for routing API calls to the correct tenant.

```sql
CREATE TABLE integrations (
  id            UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id       UUID NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,
  provider      TEXT NOT NULL CHECK (provider IN ('slack','jira','google','gmail')),
  access_token  TEXT NOT NULL,
  refresh_token TEXT,
  expires_at    TIMESTAMPTZ,
  workspace_id  TEXT,
  workspace_name TEXT,
  cloud_id      TEXT,
  scope         TEXT,
  created_at    TIMESTAMPTZ DEFAULT NOW(),
  updated_at    TIMESTAMPTZ DEFAULT NOW(),
  UNIQUE(user_id, provider)
);
```

---

## Table 3 — blocks + block_members

The core organizational unit. One row per Block. `decision_log` is a JSONB array of one-sentence AI-generated summaries, prepended every time a Signal is resolved. `block_members` is colocated here because membership is always fetched alongside the Block — it controls RLS for nearly every other table in the schema.

```sql
CREATE TABLE blocks (
  id            UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  created_by    UUID NOT NULL REFERENCES profiles(id),
  name          TEXT NOT NULL,
  description   TEXT,
  status        TEXT DEFAULT 'active' CHECK (status IN ('active','paused','archived')),
  ai_priority   TEXT DEFAULT 'normal' CHECK (ai_priority IN ('high','normal','low')),
  user_priority TEXT DEFAULT 'normal' CHECK (user_priority IN ('high','normal','low')),
  decision_log  JSONB DEFAULT '[]',
  created_at    TIMESTAMPTZ DEFAULT NOW(),
  updated_at    TIMESTAMPTZ DEFAULT NOW()
);

-- block_members: who can see each block
CREATE TABLE block_members (
  block_id   UUID NOT NULL REFERENCES blocks(id) ON DELETE CASCADE,
  user_id    UUID NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,
  role       TEXT DEFAULT 'member' CHECK (role IN ('owner','member')),
  joined_at  TIMESTAMPTZ DEFAULT NOW(),
  PRIMARY KEY (block_id, user_id)
);
```

---

## Table 4 — block_sources

Links a Block to its connected tools — Slack channels, Jira tickets, Google Docs, Gmail threads. Each row represents one source connection. The `spawned` flag indicates whether the source was created from within FocusBlock (Tier 1 association, permanent) or linked after the fact (eligible for AI re-association). Used heavily by the webhook event processor to route incoming events to the correct Block.

```sql
CREATE TABLE block_sources (
  id           UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  block_id     UUID NOT NULL REFERENCES blocks(id) ON DELETE CASCADE,
  provider     TEXT NOT NULL CHECK (provider IN ('slack','jira','google','gmail')),
  source_type  TEXT NOT NULL,
  -- slack: 'channel' | 'dm' | 'group_dm'
  -- jira:  'project' | 'epic' | 'ticket'
  -- google: 'doc' | 'folder'
  source_id    TEXT NOT NULL,
  source_name  TEXT,
  source_url   TEXT,
  spawned      BOOLEAN DEFAULT FALSE,
  created_at   TIMESTAMPTZ DEFAULT NOW(),
  UNIQUE(block_id, provider, source_id)
);
```

---

## Table 5 — signals

A discrete conversation thread within a Block — one question, one decision, one issue being worked through. The AI creates Signals by clustering related Cues. `ai_summary` holds the one-to-two sentence sub-summary shown in the side panel. `urgency` drives Feed ordering and Focus Mode behavior.

```sql
CREATE TABLE signals (
  id            UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  block_id      UUID NOT NULL REFERENCES blocks(id) ON DELETE CASCADE,
  title         TEXT,
  ai_summary    TEXT,
  urgency       TEXT DEFAULT 'fyi' CHECK (urgency IN ('blocking','action_required','fyi')),
  status        TEXT DEFAULT 'open' CHECK (status IN ('open','resolved')),
  resolved_by   UUID REFERENCES profiles(id),
  resolved_at   TIMESTAMPTZ,
  created_at    TIMESTAMPTZ DEFAULT NOW(),
  updated_at    TIMESTAMPTZ DEFAULT NOW()
);
```

---

## Table 6 — cues

The atomic unit of the product. One row per raw message, comment, or notification from a connected tool. Cues are the input the AI reads to create and update Signals. `is_unassigned` flags Cues that couldn't be confidently routed to a Block and should appear in the Unassigned Feed card. `thread_ts` preserves Slack thread structure for the two-tier clustering model.

```sql
CREATE TABLE cues (
  id            UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  signal_id     UUID REFERENCES signals(id) ON DELETE SET NULL,
  block_id      UUID REFERENCES blocks(id) ON DELETE CASCADE,
  provider      TEXT NOT NULL CHECK (provider IN ('slack','jira','google','gmail')),
  source_type   TEXT NOT NULL,
  source_id     TEXT NOT NULL,
  source_name   TEXT,
  source_url    TEXT,
  author_name   TEXT,
  author_id     TEXT,
  content       TEXT NOT NULL,
  content_ts    TIMESTAMPTZ,
  is_dm         BOOLEAN DEFAULT FALSE,
  thread_ts     TEXT,
  urgency       TEXT DEFAULT 'fyi' CHECK (urgency IN ('blocking','action_required','fyi')),
  is_unassigned BOOLEAN DEFAULT FALSE,
  created_at    TIMESTAMPTZ DEFAULT NOW()
);
```

---

## Table 7 — raw_events

The webhook event queue. Every incoming event from Slack, Jira, Google, or Gmail is written here first with `status = 'pending'`. A pg_cron job triggers the event processor edge function every 30 seconds to drain the queue. This decouples webhook receipt from processing — webhooks return 200 immediately, processing happens asynchronously.

```sql
CREATE TABLE raw_events (
  id           UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  provider     TEXT NOT NULL,
  event_type   TEXT NOT NULL,
  payload      JSONB NOT NULL,
  user_id      UUID REFERENCES profiles(id),
  status       TEXT DEFAULT 'pending' CHECK (status IN ('pending','processing','processed','failed')),
  retry_count  INTEGER DEFAULT 0,
  error        TEXT,
  created_at   TIMESTAMPTZ DEFAULT NOW(),
  processed_at TIMESTAMPTZ
);

CREATE INDEX idx_raw_events_pending ON raw_events(status, created_at)
  WHERE status = 'pending';
```

---

## Table 8 — block_context

Caches the full body text of Google Docs and full descriptions of Jira tickets linked to a Block. This is what makes Block-level AI chat fast and accurate — when a user asks "what does the Technical Brief say about error states?", the AI reads from this cache rather than making a live API call. Populated immediately on `block_sources` INSERT via a database trigger, and refreshed every 60 minutes by a background cron job. A live fallback fetch runs if the cache is more than 60 minutes stale at query time.

```sql
CREATE TABLE block_context (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  block_source_id UUID NOT NULL REFERENCES block_sources(id) ON DELETE CASCADE,
  block_id        UUID NOT NULL REFERENCES blocks(id) ON DELETE CASCADE,
  content_type    TEXT NOT NULL CHECK (content_type IN ('google_doc_body','jira_ticket_description')),
  content         TEXT NOT NULL,
  character_count INTEGER,
  fetched_at      TIMESTAMPTZ DEFAULT NOW(),
  UNIQUE(block_source_id, content_type)
);

CREATE INDEX idx_block_context_block
  ON block_context(block_id, content_type);

-- Index for identifying stale cache entries
CREATE INDEX idx_block_context_stale
  ON block_context(fetched_at)
  WHERE fetched_at < NOW() - INTERVAL '60 minutes';
```

---

## Table 9 — focus_sessions

Tracks Focus Mode activations per user. When a session is active (`ended_at IS NULL` and `ends_at` is in the future), the event processor holds Action Required and FYI items rather than surfacing them in the Feed. `ended_early` distinguishes manual deactivation from natural expiry.

```sql
CREATE TABLE focus_sessions (
  id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id     UUID NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,
  started_at  TIMESTAMPTZ DEFAULT NOW(),
  ends_at     TIMESTAMPTZ NOT NULL,
  ended_early BOOLEAN DEFAULT FALSE,
  ended_at    TIMESTAMPTZ
);

CREATE INDEX idx_focus_sessions_active
  ON focus_sessions(user_id, ends_at)
  WHERE ended_at IS NULL;
```

---

## Table 10 — tasks

User-created and AI-suggested proactive work items. Tasks are Block-level, not Signal-level — they persist beyond Signal resolution because they represent intent, not just a conversation thread. `linked_doc_id`, `linked_doc_url`, and `linked_doc_name` store the Google Doc association for V1 (Jira ticket linking is V2). The partial index on non-done tasks keeps Feed urgency queries fast.

```sql
CREATE TABLE tasks (
  id            UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  block_id      UUID NOT NULL REFERENCES blocks(id) ON DELETE CASCADE,
  created_by    UUID NOT NULL REFERENCES profiles(id),
  title         TEXT NOT NULL,
  urgency       TEXT DEFAULT 'action_required' CHECK (urgency IN ('blocking','action_required','fyi')),
  status        TEXT DEFAULT 'todo' CHECK (status IN ('todo','in_progress','done')),
  assignee_id   UUID REFERENCES profiles(id),
  due_date      DATE,
  linked_doc_id TEXT,
  linked_doc_url TEXT,
  linked_doc_name TEXT,
  created_at    TIMESTAMPTZ DEFAULT NOW(),
  updated_at    TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_tasks_block
  ON tasks(block_id, status, urgency)
  WHERE status != 'done';
```

---

## Table 11 — task_suggestions

Pending AI-generated Task suggestions waiting for user acceptance or dismissal. Created by the event processor when it detects an actionable item in a Google Doc comment that doesn't map to an existing Signal or Jira ticket. The UI polls this table per Block and renders suggestion cards below the Signal list. Accepted rows become rows in `tasks`; dismissed rows are soft-deleted via status update.

```sql
CREATE TABLE task_suggestions (
  id            UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  block_id      UUID NOT NULL REFERENCES blocks(id) ON DELETE CASCADE,
  linked_doc_id TEXT,
  linked_doc_name TEXT,
  title         TEXT NOT NULL,
  urgency       TEXT DEFAULT 'action_required',
  status        TEXT DEFAULT 'pending' CHECK (status IN ('pending','accepted','dismissed')),
  source_cue_id UUID REFERENCES cues(id),
  created_at    TIMESTAMPTZ DEFAULT NOW()
);

-- UI polls this table for pending suggestions per block
CREATE INDEX idx_task_suggestions_pending
  ON task_suggestions(block_id, status)
  WHERE status = 'pending';
```

---

## Indexes

Run after all tables are created. These cover the queries the app runs most frequently — Feed loading, Signal triage, webhook routing, and task badge counts.

```sql
-- Blocks: user's active blocks ordered by priority
CREATE INDEX idx_blocks_member_status
  ON block_members(user_id)
  INCLUDE (block_id);

CREATE INDEX idx_blocks_status_priority
  ON blocks(status, user_priority, ai_priority, updated_at DESC)
  WHERE status = 'active';

-- Signals: open signals per block ordered by urgency
CREATE INDEX idx_signals_block_status
  ON signals(block_id, status, urgency, updated_at DESC);

-- Cues: cues per signal
CREATE INDEX idx_cues_signal
  ON cues(signal_id, created_at DESC);

-- Cues: unassigned cues per user
CREATE INDEX idx_cues_unassigned
  ON cues(block_id, is_unassigned, created_at DESC)
  WHERE is_unassigned = TRUE;

-- Block sources: find block by source ID (for webhook routing)
CREATE INDEX idx_block_sources_lookup
  ON block_sources(provider, source_id, block_id);

-- Tasks: open tasks per block ordered by urgency
CREATE INDEX idx_tasks_block_open
  ON tasks(block_id, status, urgency, due_date)
  WHERE status != 'done';

-- Tasks: tasks linked to a specific doc (for task count badges)
CREATE INDEX idx_tasks_doc
  ON tasks(linked_doc_id, status)
  WHERE linked_doc_id IS NOT NULL AND status != 'done';

-- Task suggestions: pending suggestions per block (UI polls this)
CREATE INDEX idx_task_suggestions_block_pending
  ON task_suggestions(block_id, created_at DESC)
  WHERE status = 'pending';
```

---

## Row Level Security

RLS is enabled on every table. The core pattern: users can only see data belonging to Blocks they are a member of. `block_members` is the authority — nearly every policy resolves to a subquery against it.

```sql
-- Enable RLS on all tables
ALTER TABLE profiles         ENABLE ROW LEVEL SECURITY;
ALTER TABLE integrations     ENABLE ROW LEVEL SECURITY;
ALTER TABLE blocks           ENABLE ROW LEVEL SECURITY;
ALTER TABLE block_members    ENABLE ROW LEVEL SECURITY;
ALTER TABLE block_sources    ENABLE ROW LEVEL SECURITY;
ALTER TABLE signals          ENABLE ROW LEVEL SECURITY;
ALTER TABLE cues             ENABLE ROW LEVEL SECURITY;
ALTER TABLE raw_events       ENABLE ROW LEVEL SECURITY;
ALTER TABLE focus_sessions   ENABLE ROW LEVEL SECURITY;
ALTER TABLE block_context    ENABLE ROW LEVEL SECURITY;

-- profiles: users can only read and update their own profile
CREATE POLICY "profiles_own" ON profiles
  FOR ALL USING (auth.uid() = id);

-- integrations: users can only access their own tokens
CREATE POLICY "integrations_own" ON integrations
  FOR ALL USING (auth.uid() = user_id);

-- blocks: visible to members only
CREATE POLICY "blocks_members" ON blocks
  FOR SELECT USING (
    id IN (SELECT block_id FROM block_members WHERE user_id = auth.uid())
  );
CREATE POLICY "blocks_insert" ON blocks
  FOR INSERT WITH CHECK (auth.uid() = created_by);
CREATE POLICY "blocks_update" ON blocks
  FOR UPDATE USING (
    id IN (SELECT block_id FROM block_members WHERE user_id = auth.uid() AND role = 'owner')
  );

-- block_members: visible to members of the same block
CREATE POLICY "block_members_read" ON block_members
  FOR SELECT USING (
    block_id IN (SELECT block_id FROM block_members WHERE user_id = auth.uid())
  );

-- block_sources: visible to block members
CREATE POLICY "block_sources_members" ON block_sources
  FOR ALL USING (
    block_id IN (SELECT block_id FROM block_members WHERE user_id = auth.uid())
  );

-- signals: visible to block members
CREATE POLICY "signals_members" ON signals
  FOR ALL USING (
    block_id IN (SELECT block_id FROM block_members WHERE user_id = auth.uid())
  );

-- cues: visible to block members
CREATE POLICY "cues_members" ON cues
  FOR ALL USING (
    block_id IN (SELECT block_id FROM block_members WHERE user_id = auth.uid())
  );

-- raw_events: users see only their own events
CREATE POLICY "raw_events_own" ON raw_events
  FOR ALL USING (auth.uid() = user_id);

-- focus_sessions: users see only their own sessions
CREATE POLICY "focus_sessions_own" ON focus_sessions
  FOR ALL USING (auth.uid() = user_id);

-- tasks: visible to block members
CREATE POLICY "tasks_members" ON tasks
  FOR ALL USING (
    block_id IN (SELECT block_id FROM block_members WHERE user_id = auth.uid())
  );

-- task_suggestions: visible to block members
CREATE POLICY "task_suggestions_members" ON task_suggestions
  FOR ALL USING (
    block_id IN (SELECT block_id FROM block_members WHERE user_id = auth.uid())
  );

-- block_context: visible to block members
CREATE POLICY "block_context_members" ON block_context
  FOR ALL USING (
    block_id IN (SELECT block_id FROM block_members WHERE user_id = auth.uid())
  );
```
