# Lullio — Product Requirements Document

**Version:** 1.0
**Last updated:** April 2026
**Owner:** Brett
**Status:** Pre-build (design & prototypes complete)

---

## 1. Summary

Lullio is an AI-powered co-parenting shift-change briefing tool for new parents. One parent logs baby care events during their shift; an AI engine synthesizes those events into a structured briefing; the other parent reads it when starting their shift.

The app is built for **sleep-deprived users holding a baby**. Every design and engineering decision is measured against a single principle:

> **One hand, one eye, three seconds.**
>
> If a user cannot complete the action with one hand, while looking at it with one eye, in three seconds or less — it fails the core use case.

---

## 2. Problem

New parents share caregiving across shifts — one sleeps while the other is "on baby," then they swap. At every handoff, the incoming parent needs to know:

- When did the baby last eat, sleep, get changed, take medication?
- How was the last stretch — fussy, content, off?
- Anything unusual worth watching for?

Today, this information is communicated via groggy verbal handoffs at 3am, scattered text messages, or spreadsheets and tracker apps that require two hands and full attention to operate. The data is either lost, incomplete, or captured at the cost of the very rest the handoff is supposed to enable.

Lullio closes this loop with minimum-friction capture during the shift and a structured, glanceable briefing at the handoff.

---

## 3. Users

### Primary persona: The on-shift parent (capture-side)
- Holding the baby, one hand free at best
- Often in the dark, often half-asleep
- Needs to log events in seconds or will not log at all
- Does not want to "write a note" — wants to dump state

### Primary persona: The off-shift parent (read-side)
- Waking up or arriving to take over
- Has ~60 seconds of attention before the baby needs them
- Needs the last shift summarized, not replayed
- Needs to know what to watch for in the next few hours

### Out of scope (for now)
- Solo parents (no handoff partner)
- Caregivers beyond the two parents (nanny, grandparent)
- Multi-child households

---

## 4. Core loop

1. **Shift starts.** Parent A taps "Start shift." Baby is now their responsibility.
2. **Events get logged.** Throughout the shift, Parent A captures events via:
   - **Quick Tags** — pre-defined taps (fed, slept, changed, meds, fussy, content)
   - **Voice memos** — transcribed via Deepgram, parsed into structured events by Claude Haiku
   - **Free text** — typed notes when needed
3. **Shift ends.** Parent A taps "End shift." Reviews auto-generated summary. Sends to partner.
4. **Briefing generated.** Claude Sonnet synthesizes events + summary into a structured briefing (last feed, last sleep, notable patterns, what to watch).
5. **Partner reads briefing.** Parent B opens Lullio, sees the briefing, starts their shift.

---

## 5. Scope

### MVP (must ship)
- Two-parent household setup (auth + partner invite)
- Shift state machine: Idle → Active → Ending/Review → Sent
- Quick Tags capture
- Voice capture with Deepgram transcription
- Free-text event capture
- AI event parsing (Haiku)
- AI briefing generation (Sonnet)
- Briefing screen (partner-facing)
- Real-time sync (partner sees shift status live)
- Push notifications (shift ended, briefing ready)
- Offline-first event capture with sync-when-online
- Dark mode ("Nightlight Mode")
- App Store IAP (one-time annual fee, positioned as baby-product purchase)

### V1 (post-launch, first 90 days)
- Pattern intelligence (feed/sleep trends over multi-day horizon)
- Shareable pediatrician export (PDF)
- Web-to-app hybrid payment flow (Stripe + deep link)
- Baby shower gifting flow

### V2 (later)
- Multi-child support
- Caregiver modes (nanny, grandparent)
- Apple Watch companion
- Smart suggestions during capture ("Looks like baby is due for a feed")

### Explicitly out of scope
- Hardware or IoT integrations (bottles, bassinets, wearables)
- Medical advice or diagnosis
- Social / community features
- Generalized note-taking beyond baby care context

---

## 6. Design principles

1. **One hand, one eye, three seconds.** The north star. Every screen passes this test or it fails review.
2. **Capture over correctness.** It's better to log a sloppy event fast than a perfect event slowly. The AI cleans up afterward.
3. **The briefing is the product.** Capture is the cost; the briefing is the value. Invest UX effort proportionally.
4. **Nightlight Mode is the default assumption.** Design for 3am first, daytime second.
5. **Lucide icons for UI, emoji for personality.** Icons carry function, emoji carry tone.

---

## 7. Tech stack

| Layer | Choice | Why |
|---|---|---|
| Mobile framework | React Native via Expo | Brett's existing skills; EAS handles build/submit |
| Backend | Supabase | Auth, Postgres, RLS, real-time; migration already written |
| Speech-to-text | Deepgram | Fast, accurate, streaming-capable |
| Event parsing | Claude Haiku (via API) | High-volume, low-latency, cheap |
| Briefing generation | Claude Sonnet (via API) | Low-volume, high-quality output |
| AI proxy | Lightweight Node service on Railway | Keeps API keys server-side; enables logging, rate limiting, cost caps |
| Mobile OTA + builds | EAS (Expo Application Services) | Native Expo integration |
| Payments (MVP) | Apple / Google IAP | Path of least resistance for launch |
| Payments (V1) | Stripe web checkout + deep link | Web-to-app hybrid to reduce App Store commission |
| Marketing site | Next.js on Vercel | Hosts Stripe flow in V1 |
| CI | GitHub Actions | Lint, typecheck, test on PR; trigger EAS builds on merge |
| E2E testing | Maestro | RN-native, YAML flows, pairs well with Expo |
| Unit / integration testing | Jest + React Native Testing Library | Standard |
| Offline sync | TBD — spike early | WatermelonDB or SQLite queue; Supabase alone is insufficient |

---

## 8. Data model (high level)

See `supabase/migrations/` for the canonical schema. Core tables:

- **households** — one per couple; RLS-scoped root
- **users** — auth users linked to a household
- **shifts** — start/end timestamps, status, owner user_id, household_id
- **events** — timestamped baby care events (type, payload, source: tag/voice/text)
- **briefings** — generated per shift; includes raw summary + structured fields (last feed, last sleep, watch-items)
- **invitations** — partner-invite tokens

All tables RLS-scoped to household_id. Partners in the same household can read each other's shifts and events. Users in different households never see each other's data.

---

## 9. Top technical risks

| Risk | Mitigation |
|---|---|
| **AI cost runaway.** Haiku fires per-utterance during a voice-heavy shift; a chatty parent could generate thousands of calls. | Debounce/batch event parsing (every N seconds or M events). Per-shift token ceiling enforced at the proxy. Per-user cost monitoring from day one. |
| **Offline sync gaps.** Parents use Lullio at 3am with bad WiFi; Supabase alone does not handle local-first writes gracefully. | Spike WatermelonDB or a SQLite outbox queue in Phase 2. All Quick Tag writes must succeed locally before hitting the network. |
| **Briefing quality under noisy input.** If the voice transcript is garbled or events are ambiguous, Sonnet output may hallucinate or omit. | Sonnet prompt includes explicit instruction to flag uncertainty and omit rather than invent. Add a "raw events" expansion on the briefing so the reading parent can always verify source data. |
| **Race conditions in shift state.** Both parents could tap "Start shift" simultaneously; sync must resolve this cleanly. | State machine enforced server-side. Client optimistic updates rolled back on conflict. E2E test covers concurrent shift starts. |
| **App Store subscription friction.** New parents already drown in recurring charges; annual IAP must feel like a one-time purchase, not a subscription. | Position copy as "Lullio for year one" rather than "monthly plan." Single annual price point. Non-renewing by default. |
| **Voice privacy concerns.** Parents are recording in their homes, sometimes while a baby is present. | Deepgram calls encrypted in transit. No voice audio retained post-transcription. Clear privacy disclosure in onboarding. |

---

## 10. Success metrics

### MVP (first 30 days post-launch)
- % of shifts that end with a briefing sent (target: >80%)
- Median time from "End shift" tap to briefing-ready (target: <10s)
- Median capture time per event (target: <3s)
- D7 retention of activated households (target: >60%)

### V1 (first 90 days)
- Handoff completion rate (both parents engage with the briefing) (target: >70%)
- Events per shift (leading indicator of capture habit)
- AI cost per household per month (must stay under pricing margin)

---

## 11. Design system

Canonical source: `docs/design-tokens.md`

- **Color hierarchy:** Navy `#0F3460` / Amber `#D78A2D` / Green
- **Headline typography:** Instrument Serif italic
- **UI typography:** System font, tight scale
- **Icons:** Lucide for all UI; emoji reserved for conversational copy only
- **Dark mode:** "Nightlight Mode" — warm blue-black palette; primary flips from navy to amber
- **Motion:** Minimal. Nothing that delays a capture action. 3am-friendly means no celebratory animations.

---

## 12. Open questions

- Final pricing point for annual IAP (needs market comp pass)
- Onboarding: do we force partner invite before first shift, or allow solo use with partner-optional? (Leaning: optional, nudge hard)
- Deepgram vs on-device Whisper for privacy-conscious users in V1?
- Does the briefing send automatically on shift end, or require explicit tap to send? (Leaning: explicit — reviewing adds trust)

---

## 13. References

- `docs/design-tokens.md` — color, type, spacing, motion
- `docs/build-kit.md` — Cursor/Claude Code prompt pack
- `supabase/migrations/` — database schema and RLS
- Prototypes: Quick Tags, Briefing screen, Shift Flow (four states)
