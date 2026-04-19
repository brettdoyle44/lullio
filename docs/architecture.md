# Architecture Decisions

A log of significant technical decisions, their rationale, and alternatives considered.

Newest entries at the top. When you make a decision that will shape the codebase, add an entry here.

## Format

### YYYY-MM-DD — Short title of decision

**Decision:** What we decided, in one sentence.

**Context:** Why this came up now.

**Alternatives considered:**

- Option A — why not
- Option B — why not

**Consequences:** What this commits us to, what it rules out, what we'll need to revisit.

---

## Decisions

### 2026-04-18 — Mobile framework: React Native via Expo

**Decision:** Build Lullio as an Expo-managed React Native app.

**Context:** Need to ship iOS + Android from a single codebase. Brett has React experience from prior projects (Voyage Board, ClientPact).

**Alternatives considered:**

- Native iOS/Android — rejected: 2x the build and maintenance cost for a solo founder.
- Flutter — rejected: unfamiliar stack, weaker Supabase/Deepgram SDK support.
- Bare React Native — rejected: Expo's EAS build + OTA updates save meaningful time; can eject later if needed.

**Consequences:** Committed to the Expo SDK release cadence. Some native modules may require prebuild. EAS becomes the path for builds and submission.

---

### 2026-04-18 — Backend: Supabase

**Decision:** Supabase for auth, database, RLS, and real-time.

**Context:** Need auth, a relational store, row-level security, and real-time sync in one service for a two-sided (partner) app.

**Alternatives considered:**

- Firebase — rejected: weaker SQL story, Lullio's data is relational.
- Custom Postgres + custom auth — rejected: too much undifferentiated backend work for MVP.
- Convex — rejected: promising but less mature, weaker mobile SDK story.

**Consequences:** Committed to Postgres. RLS is the security model — every table must be household-scoped. Supabase's mobile SDK quirks (especially around real-time reconnection) will need handling.

---

### 2026-04-18 — AI calls proxied through Railway, not direct from app

**Decision:** All Anthropic API calls go through a Node service on Railway. The mobile app never holds the Anthropic API key.

**Context:** Direct-from-client calls would expose the API key and make rate limiting and cost ceilings impossible.

**Alternatives considered:**

- Direct from app — rejected: key exposure, no cost control.
- Supabase Edge Functions — considered, deferred: viable but adds a second runtime to reason about; a dedicated Node service is simpler for MVP.

**Consequences:** Extra deploy target (Railway). Adds a network hop to every AI call. Gets us logging, per-household rate limits, per-shift token ceilings, and caching for free.

---

<!-- Add new decisions above this line -->
