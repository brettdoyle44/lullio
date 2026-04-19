# Lullio — Build Todo List

A phased checklist from empty repo to App Store launch. Designed to be worked top-to-bottom. Each phase ends in a state where you could stop and still have something useful; each phase is also a natural branch-and-merge boundary for Claude Code sessions.

Legend: `[ ]` todo · `[~]` spike/exploration · `[!]` blocker risk

---

## Phase 0 — Repo prep & foundations

Goal: Claude Code has everything it needs on day one.

- [ ] Create GitHub repo `lullio`
- [ ] Initialize Expo app (`npx create-expo-app --template`) with TypeScript template
- [ ] Set up `main` branch protection, require PR, require CI green
- [ ] Add `.gitignore`, `.editorconfig`, `.nvmrc`
- [ ] Drop canonical docs into `docs/`:
  - [ ] `docs/PRD.md` (this doc)
  - [ ] `docs/design-tokens.md` (from build kit)
  - [ ] `docs/build-kit.md` (the nine-prompt Cursor pack)
  - [ ] `docs/architecture.md` (stub — grows as we build)
- [ ] Create `CLAUDE.md` at repo root — Claude Code reads this first every session. Include: the "one hand, one eye, three seconds" principle, stack summary, where to find docs, commands to run tests.
- [ ] Set up Linear (or GitHub Issues) project for Lullio; import this todo as milestones
- [ ] Provision empty Supabase project (dev)
- [ ] Provision empty Supabase project (prod) — don't touch it yet, just reserve
- [ ] Create Anthropic API key, Deepgram API key, store in 1Password or similar
- [ ] Create Expo / EAS account, link to repo

---

## Phase 1 — Scaffold (one Claude Code session, ~2 hours)

Goal: the app boots, navigates, and looks like Lullio. No logic yet.

- [ ] Install core deps: `expo-router`, `@supabase/supabase-js`, `react-native-reanimated`, `react-native-gesture-handler`
- [ ] Set up Expo Router with file-based routing
- [ ] Create folder structure: `app/`, `components/`, `lib/`, `hooks/`, `services/`, `types/`, `assets/`
- [ ] Implement theme provider wired to design tokens (light + Nightlight Mode)
- [ ] Set up Lucide icon library; confirm rendering
- [ ] Load Instrument Serif italic font via `expo-font`
- [ ] Create screen shells (empty but navigable):
  - [ ] Onboarding / auth
  - [ ] Shift — Idle
  - [ ] Shift — Active
  - [ ] Shift — Ending / Review
  - [ ] Shift — Sent
  - [ ] Briefing (partner-facing)
  - [ ] Settings
- [ ] Add basic bottom-tab or stacked nav between the main screens
- [ ] Commit: `chore: scaffold lullio app`

---

## Phase 2 — Supabase + auth (one session)

Goal: households exist, partners can be invited, RLS holds.

- [ ] Apply the existing SQL migration to dev Supabase project
- [ ] Verify RLS policies via the Supabase SQL editor with two test users
- [ ] Build Supabase client singleton in `lib/supabase.ts`
- [ ] Implement auth flow:
  - [ ] Email + magic link (simplest; no passwords to manage)
  - [ ] Or Apple Sign-In + Google Sign-In (better UX on mobile; pick one path and commit)
- [ ] Build onboarding: new user → create household → generate partner invite link
- [ ] Build invite-acceptance flow: deep link → join existing household
- [ ] Test: two devices, one household, both can read shared data; a third device outside the household cannot.
- [ ] `[!]` Get auth watertight before moving on. Every later feature depends on it.

---

## Phase 3 — Offline sync spike

Goal: know now whether Supabase alone is enough, or whether we need WatermelonDB / SQLite outbox.

- [ ] `[~]` Spike: implement Quick Tag write with pure Supabase, kill network mid-write, confirm behavior
- [ ] `[~]` Spike: same test with a local SQLite outbox queue (simplest option)
- [ ] `[~]` Spike: evaluate WatermelonDB as a heavier-weight alternative
- [ ] Decide: pure Supabase / SQLite outbox / WatermelonDB. Document decision in `docs/architecture.md`
- [ ] Implement chosen approach as a service layer (`services/events.ts`) so the rest of the app doesn't care how persistence works

---

## Phase 4 — Quick Tags capture (one session)

Goal: a parent can log events in under 3 seconds.

- [ ] Port Quick Tags prototype to React Native
- [ ] Wire taps to event service; writes succeed offline
- [ ] Haptic feedback on tap (subtle — 3am-friendly)
- [ ] Undo affordance for the last 5 seconds (easy to mis-tap one-handed)
- [ ] Visual confirmation that the event was captured (no modal — inline)
- [ ] Test against the core principle: one-handed, dim screen, sub-3-second capture
- [ ] E2E test: open app → start shift → log 5 events → see them in shift state

---

## Phase 5 — Shift state machine (one session)

Goal: Idle → Active → Ending → Sent works correctly, including across two devices.

- [ ] Model shift state server-side (Supabase row with status field)
- [ ] Client optimistic updates with server reconciliation
- [ ] Prevent concurrent shifts in same household (server-enforced)
- [ ] "Start shift" and "End shift" actions
- [ ] Ending/Review state: show all events from shift, allow quick edits
- [ ] Sent state: confirmation screen, option to start new shift immediately
- [ ] E2E test: two devices, shift started on A, B sees "A is on shift" live
- [ ] E2E test: concurrent "start shift" taps; only one succeeds

---

## Phase 6 — Voice capture + Deepgram (one session)

Goal: parent speaks; transcript appears; transcript flows into event pipeline.

- [ ] Integrate `expo-av` for audio recording
- [ ] Tap-and-hold or tap-to-toggle UI (decide; lean toggle — easier one-handed)
- [ ] Stream audio to Deepgram (or batch upload — start batch, optimize later)
- [ ] Handle transcription errors gracefully (show raw audio duration, allow retry)
- [ ] Transcript saved to event row with `source: 'voice'` and `raw_transcript` field
- [ ] Verify: no voice audio is retained server-side post-transcription
- [ ] E2E test: record 10s, transcript appears in event list

---

## Phase 7 — AI proxy service on Railway (one session)

Goal: AI calls go through our service, not direct from app.

- [ ] Create `lullio-ai-proxy` repo (or monorepo package)
- [ ] Minimal Node + Fastify service; one endpoint per purpose: `/parse-event`, `/generate-briefing`
- [ ] Anthropic SDK, server-side API key
- [ ] Auth: Supabase JWT verification — only authenticated Lullio users can call
- [ ] Rate limiting per household (per minute, per shift, per day)
- [ ] Per-shift token ceiling enforced
- [ ] Structured logging: request, household, model, tokens, cost, latency
- [ ] Deploy to Railway with staging + prod environments
- [ ] Wire app to proxy URLs via env-scoped config

---

## Phase 8 — Event parsing (Haiku) (one session)

Goal: voice transcripts and free text become structured events.

- [ ] Design Haiku prompt for event extraction (returns JSON: type, timestamp, confidence, raw_text)
- [ ] Debounce: batch events every N seconds rather than per-utterance
- [ ] Handle multi-event transcripts ("fed at 3, then nap at 3:45")
- [ ] Low-confidence events flagged rather than dropped
- [ ] Cost monitoring per parse call (log to proxy's metrics)
- [ ] Unit tests with fixture transcripts covering happy path, ambiguous input, garbled input

---

## Phase 9 — Briefing generation (Sonnet) (one session)

Goal: shift ends → structured briefing exists.

- [ ] Design Sonnet prompt:
  - Inputs: shift events, prior shift's briefing (for continuity), time of day
  - Output: structured JSON with `last_feed`, `last_sleep`, `total_feeds`, `total_sleep_hours`, `notable_patterns`, `watch_items`, `headline`
- [ ] Prompt explicitly instructs: flag uncertainty; never invent events; omit rather than fabricate
- [ ] Briefing triggered on shift-end action, stored in `briefings` table
- [ ] Raw events accessible from briefing (expandable "source data" section)
- [ ] Cost monitoring per generation
- [ ] Unit tests with fixture shifts (typical, sparse, chaotic)

---

## Phase 10 — Briefing screen (one session)

Goal: partner opens app; sees the briefing; it passes the 3-second test.

- [ ] Port Briefing prototype to React Native
- [ ] Top of screen: headline + "what to watch"
- [ ] Middle: last feed, last sleep, next likely need
- [ ] Bottom: expand for full event list
- [ ] Uncertainty flags rendered distinctly (not buried)
- [ ] Timestamps in relative time ("32 min ago")
- [ ] Pull-to-refresh
- [ ] Test against the core principle: passing parent, 3am, glance

---

## Phase 11 — Real-time sync (one session)

Goal: partner's app updates live during the other parent's shift.

- [ ] Supabase real-time subscription on `shifts` + `events` for current household
- [ ] Live indicator when partner is on shift ("Partner started shift 23 min ago")
- [ ] New events appear in partner's view without refresh (useful for handoff-in-progress)
- [ ] Subscription cleanup on logout / app background
- [ ] Test: two devices, real-time updates within ~1s

---

## Phase 12 — Push notifications (one session)

Goal: partner knows when shift ended and briefing is ready.

- [ ] Configure Expo push notifications
- [ ] Register device token per user, store in Supabase
- [ ] Trigger push on shift-end + briefing-ready (from Railway proxy after generation)
- [ ] Quiet hours respect (user-configurable — important for sleeping parent)
- [ ] Tapping notification deep-links to briefing screen
- [ ] Test on real iOS and Android devices

---

## Phase 13 — Onboarding polish (one session)

Goal: first-run experience does not feel like software.

- [ ] Welcome screen with Instrument Serif headline
- [ ] Privacy disclosure (voice, data) — clear, short, not a wall of legal text
- [ ] Household creation → partner invite flow
- [ ] "Walk through a fake shift" interactive demo (optional skip)
- [ ] Empty states designed (no shifts yet, no briefing yet, partner not joined yet)

---

## Phase 14 — Payments: App Store IAP (one session)

Goal: the app is monetized before TestFlight goes wide.

- [ ] Use `react-native-iap` or `expo-in-app-purchases`
- [ ] Single product: "Lullio — Year One" annual purchase
- [ ] Position as one-time annual; non-renewing if possible
- [ ] Paywall screen (post-onboarding; grace period for dogfooding)
- [ ] Restore-purchase flow
- [ ] Receipt verification server-side
- [ ] Test with App Store sandbox accounts

---

## Phase 15 — Settings & account management

- [ ] Settings screen: household, partner, notifications, quiet hours, theme
- [ ] Sign out
- [ ] Delete account (GDPR / App Store requirement — cascade to household if sole member)
- [ ] Manage subscription (deep link to App Store)
- [ ] Support contact (email link)

---

## Phase 16 — Adversarial review pass

Goal: fresh Claude Code session as reviewer; find what we missed.

- [ ] Prompt: "You are reviewing Lullio adversarially. Find: race conditions in shift state, RLS policy holes, AI cost explosion vectors, accessibility failures against one-hand/one-eye/three-seconds, privacy leaks, offline sync bugs, payment flow edge cases."
- [ ] Triage findings: real vs noise
- [ ] Fix all "real" findings
- [ ] Repeat with a second review focused specifically on the payment + privacy surface

---

## Phase 17 — Testing infrastructure

- [ ] Jest + React Native Testing Library configured in CI
- [ ] Maestro flows for core paths:
  - [ ] Onboarding → first shift → briefing
  - [ ] Partner receives briefing
  - [ ] Offline event capture → sync recovery
  - [ ] Concurrent shift-start rejection
- [ ] GitHub Actions: typecheck, lint, test on every PR
- [ ] GitHub Actions: EAS build triggered on merge to `main`

---

## Phase 18 — Dogfooding

Goal: we (Brett and partner) use Lullio for real shifts, two weeks minimum.

- [ ] EAS internal build to Brett's device
- [ ] Brett's partner installs via TestFlight / EAS internal distribution
- [ ] Use it for every real shift for 14 days
- [ ] Maintain a dogfood log — what broke, what felt wrong, what surprised us
- [ ] Fix every P0 / P1 before going wider
- [ ] `[!]` Do not skip this phase. You are the target user with a newborn. This is your unfair advantage.

---

## Phase 19 — TestFlight beta (closed)

- [ ] Recruit 10-20 new-parent households (friends, Ideabrowser community, local parent groups)
- [ ] TestFlight build with feedback prompt built in
- [ ] Weekly check-ins with beta users
- [ ] Track: activation rate, D7 retention, shifts per household per week, events per shift
- [ ] Iterate based on feedback for 4 weeks minimum

---

## Phase 20 — App Store submission & launch

- [ ] App Store Connect metadata: name, description, screenshots, keywords
- [ ] Privacy policy + terms (hosted on marketing site)
- [ ] App Review: be ready to explain the voice/AI pipeline to reviewers
- [ ] Screenshots that demo the product honestly (not motion-designed fake content)
- [ ] Google Play equivalent (if Android launch is concurrent)
- [ ] Launch day plan: Ideabrowser relaunch, social posts, Product Hunt if appropriate

---

## Post-launch backlog (V1 candidates)

- [ ] Pattern intelligence (trend surfacing across days/weeks)
- [ ] Pediatrician export (PDF summary of last 2 weeks)
- [ ] Web-to-app hybrid payment flow (Stripe + deep link)
- [ ] Baby shower gifting flow
- [ ] Marketing site on Vercel
- [ ] Referral mechanics (gifted month to referred household)

---

## Cross-cutting concerns (revisit every phase)

- **One hand, one eye, three seconds:** every new screen tested against this
- **AI cost:** every new AI call logged and budgeted
- **Privacy:** every new data capture reviewed for minimum-necessary principle
- **Accessibility:** screen reader support, dynamic type, contrast in both themes
- **Copy quality:** tone review every phase — Lullio's voice is warm, not clinical; concise, not cute
