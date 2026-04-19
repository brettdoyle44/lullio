# CLAUDE.md

This file is the entrypoint for Claude Code working on this repo. Read it first, every session.

---

## What Lullio is

Lullio is an AI-powered co-parenting shift-change briefing tool for new parents. One parent logs baby care events during their shift; Claude synthesizes them into a structured briefing; the other parent reads it when starting their shift.

Canonical product spec: [`docs/PRD.md`](./docs/PRD.md)
Canonical build plan: [`docs/TODO.md`](./docs/TODO.md)
Design tokens: [`docs/design-tokens.md`](./docs/design-tokens.md)
Build kit / prompt pack: [`docs/build-kit.md`](./docs/build-kit.md)
Architecture decisions log: [`docs/architecture.md`](./docs/architecture.md)

If any of those conflict with this file, **those are the source of truth** — update this file to match.

---

## The north star

> **One hand, one eye, three seconds.**

Every design, UX, and engineering decision gets measured against this. If a user cannot complete an action one-handed, glancing with one eye, in three seconds or less, it fails the core use case. This is not a tagline — it's a review criterion. When you generate UI, test yourself against it before handing the diff to Brett.

Three seconds includes network round-trips. If a capture requires the server to respond, it fails. **Writes are local-first.**

---

## Stack

| Layer | Tool |
|---|---|
| Mobile | React Native via Expo (SDK 50+) + Expo Router |
| Language | TypeScript, strict mode |
| Backend | Supabase (Postgres, auth, RLS, real-time) |
| AI proxy | Node + Fastify on Railway (keeps API keys server-side) |
| Event parsing | Claude Haiku via the proxy |
| Briefing generation | Claude Sonnet via the proxy |
| Speech-to-text | Deepgram |
| Offline sync | TBD — see `docs/architecture.md` (Phase 3 spike) |
| Unit/integration tests | Jest + React Native Testing Library |
| E2E tests | Maestro |
| CI | GitHub Actions |
| Mobile builds + OTA | EAS (Expo Application Services) |
| Payments (MVP) | Apple / Google IAP |
| Payments (V1) | Stripe web checkout + deep link |

**The app does not call the Anthropic API directly.** All AI calls go through the Railway proxy. If you see a draft that imports `@anthropic-ai/sdk` in the mobile app, that's a bug — flag it.

---

## Folder layout

```
lullio/
├── app/                    # Expo Router screens (file-based routing)
├── components/             # Reusable UI components
├── hooks/                  # Custom React hooks
├── lib/                    # Third-party client singletons (Supabase, etc.)
├── services/               # Domain logic (events, shifts, briefings, ai)
│   ├── events.ts           # Event CRUD — UI never touches Supabase for events directly
│   ├── shifts.ts           # Shift state machine
│   ├── briefings.ts        # Briefing fetch/display
│   └── ai.ts               # Calls to the Railway proxy
├── types/                  # Shared TypeScript types
├── assets/                 # Fonts, icons, images
├── supabase/
│   └── migrations/         # SQL migrations (source of truth for schema)
├── docs/                   # PRD, TODO, design tokens, architecture
└── CLAUDE.md               # This file
```

**The service layer exists so the UI doesn't care where data lives.** When Phase 3 resolves offline sync (pure Supabase vs. SQLite outbox vs. WatermelonDB), only `services/` changes. Never reach into Supabase or AsyncStorage from a screen or component.

---

## Commands

```bash
# Development
npm install                  # Install deps
npm run start                # Expo dev server
npm run ios                  # Run in iOS simulator
npm run android              # Run in Android emulator

# Quality
npm run typecheck            # tsc --noEmit, must pass before commit
npm run lint                 # ESLint
npm run test                 # Jest unit + integration
npm run test:e2e             # Maestro flows (requires simulator running)

# Supabase
npm run db:migrate           # Apply migrations to local/dev
npm run db:reset             # Reset dev database (destructive)

# Builds
eas build --profile development    # Dev client build
eas build --profile preview        # Internal distribution
eas build --profile production     # App Store submission
```

If any of these commands don't exist yet, it means we're earlier in the build than this doc assumes — add the missing script to `package.json` rather than working around it.

---

## Working rules

### Before you write code
1. Read `docs/PRD.md` if you haven't this session.
2. Check `docs/TODO.md` to confirm which phase we're in. Don't do Phase 12 work during Phase 4.
3. If the task touches the database, read the latest migration in `supabase/migrations/`.
4. If the task touches AI, read `services/ai.ts` and the relevant prompt in `services/ai/prompts/`.

### While you write code
- **TypeScript strict mode.** No `any` without a comment explaining why.
- **One feature per branch.** Branch naming: `phase-N/short-description` (e.g. `phase-4/quick-tags`).
- **Tests live next to source.** `events.ts` → `events.test.ts`. Not in a `__tests__/` directory.
- **Prompts are data, not code.** AI prompts live in `services/ai/prompts/*.ts` as exported strings, versioned, with fixture tests.
- **No new dependencies without a reason written in the PR.** Bundle size matters on mobile.
- **Copy is part of the design.** If you write user-facing strings, pause and ask whether they sound like Lullio (warm, concise, not cute) or like generic SaaS.

### Before you hand over the diff
- [ ] Does it pass `npm run typecheck`?
- [ ] Does it pass `npm run lint`?
- [ ] Does it pass `npm run test`?
- [ ] If it touches UI, have you applied the three-second test?
- [ ] If it touches AI, have you added a cost estimate comment (rough tokens per call)?
- [ ] If it touches RLS or auth, have you tested with two separate households?

---

## Non-negotiables

These are the things that break Lullio if you get them wrong. When in doubt, ask Brett rather than guess.

1. **Capture is local-first.** A Quick Tag tap must succeed and feel instant even with no network. The sync happens after.
2. **The Anthropic API key never touches the mobile app.** Always go through the Railway proxy.
3. **No voice audio is retained server-side post-transcription.** Deepgram transcribes, we store the transcript, the audio goes away. This is a privacy commitment, not a nice-to-have.
4. **RLS is the security model.** Every table is household-scoped. If you write a query that could leak cross-household data, that's a P0 bug even if no one notices.
5. **AI calls have cost ceilings.** Per-shift, per-household, per-day. The proxy enforces them; the client respects them gracefully.
6. **The briefing never invents events.** Sonnet's prompt says this explicitly, and the UI always makes raw events accessible so the reader can verify.
7. **Nightlight Mode is not an afterthought.** Design dark first. Most shifts happen at 3am.

---

## How to handle ambiguity

If you're uncertain about a product or architecture decision:

1. **Check `docs/architecture.md`** — past decisions are logged there.
2. **Check the open questions section at the end of `docs/PRD.md`** — if it's listed there, flag it in your PR rather than guessing.
3. **Ask Brett.** Lullio is opinionated software. Guessing at opinionated software dilutes it. Better to pause than to paper over.

If you make a decision that will shape the codebase going forward, add an entry to `docs/architecture.md` (date, decision, rationale, alternatives considered). Future-you will thank present-you.

---

## Things Lullio is not

Mentioning these because past confusion cost us real time:

- **Not a hardware product.** No IoT, no bottles, no wearables, no bassinet integrations. Software only.
- **Not a general-purpose tracker.** Baby care handoff specifically; not a journaling app, not a habit tracker.
- **Not medical software.** We don't diagnose, dose, or advise. We summarize what the parent told us.
- **Not social.** No feeds, no sharing, no community. Two parents, one household.
- **Not a subscription in how it feels.** Annual IAP positioned as a baby-product purchase. Copy reflects that.

---

## Useful references

- Expo Router docs: https://docs.expo.dev/router/introduction/
- Supabase RLS guide: https://supabase.com/docs/guides/auth/row-level-security
- Anthropic API (for the proxy, not the app): https://docs.claude.com
- Deepgram streaming API: https://developers.deepgram.com/docs/live-streaming-audio
- EAS Build: https://docs.expo.dev/build/introduction/
- Maestro: https://maestro.mobile.dev/

---

## Session kickoff template

When starting a new Claude Code session, say something like:

> We're working on Phase N (see `docs/TODO.md`). Today's task is [X]. Read `CLAUDE.md` and `docs/PRD.md` first, confirm the plan, then proceed.

This forces context loading up front and catches drift early.
