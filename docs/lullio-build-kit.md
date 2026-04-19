# Lullio — Build Kit v2

## Part 1: Cursor Prompt Pack

Nine sequential prompts to build Lullio from zero to polished MVP. Run them in order — each builds on the previous.

**IMPORTANT:** Before starting, add the file `lullio-design-tokens.md` to your Cursor project root. Every prompt below references it as the single source of truth for colors, typography, spacing, icons, and dark mode. Tell Cursor: "Use lullio-design-tokens.md as the design system reference for all styling decisions."

---

### Prompt 1: Project Setup, Theme System & Navigation Shell

```
Create a new Expo (React Native) project called "Lullio" using Expo Router for file-based navigation.

FIRST: Read the file lullio-design-tokens.md — this is the design system for the entire app. All colors, fonts, spacing, and radii come from this file. Do not invent new values.

1. Create a theme system (constants/theme.ts) implementing both light and dark token sets from the design tokens doc:
   - Light mode: background #FAFAF7, foreground #1A1A18, card #FFFFFF, primary #0F3460
   - Dark mode ("Nightlight Mode"): background #12161B, foreground #F3EDE5, card #181D23, primary #D78A2D
   - Include ALL tokens from the doc: core, primary, accent, semantic, category colors with both light and dark bg tints
   - Export as lightTheme and darkTheme objects

2. Create a ThemeProvider (contexts/ThemeContext.tsx) that:
   - Detects system color scheme with useColorScheme()
   - Provides current theme object and a manual toggle
   - Wraps the entire app

3. Set up tab navigation with Expo Router:
   - Tabs: Home, Log, Briefing, History
   - Tab bar uses Lucide React Native icons (install lucide-react-native):
     Home → House, Log → SquarePen, Briefing → FileText, History → History
   - Active tab: foreground color, strokeWidth 2.2
   - Inactive tab: muted-foreground color, strokeWidth 1.8
   - Tab label: 12px, DM Sans, weight 600 active / 500 inactive
   - Tab bar background: card color, border-top: 1px solid border color

4. Install dependencies:
   - expo-router, expo-av, expo-notifications, expo-haptics, expo-secure-store
   - @supabase/supabase-js, react-native-reanimated
   - lucide-react-native
   - @expo-google-fonts/dm-sans, @expo-google-fonts/instrument-serif

5. Load fonts in _layout.tsx:
   - DM Sans (300, 400, 500, 600, 700) — body font for everything
   - Instrument Serif (400 italic) — display font for headlines and section titles only

6. App config (app.json):
   - name: "Lullio"
   - slug: "lullio"
   - scheme: "lullio"
   - iOS bundleIdentifier: "com.lullio.app"
   - Android package: "com.lullio.app"

7. Brand mark in the top bar: 🌙 emoji (22px) + "Lullio" in Instrument Serif italic (22px). The moon emoji is one of the few places emoji appears — see the icon/emoji split rules in the design tokens doc.
```

---

### Prompt 2: Supabase Auth & Data Layer

```
Set up Supabase integration for Lullio. Reference lullio-design-tokens.md for all styling.

1. Create lib/supabase.ts — initialize the Supabase client using expo-secure-store for token persistence. Use environment variables for SUPABASE_URL and SUPABASE_ANON_KEY.

2. Create AuthProvider context (contexts/AuthContext.tsx):
   - Manages auth state (user, session, loading)
   - Provides signUp, signIn, signOut functions
   - Listens to onAuthStateChange
   - Wraps the app in _layout.tsx

3. Auth screens (Sign In / Sign Up):
   - Style using theme tokens — primary color for CTAs, background for screens, card for input containers
   - Brand mark (🌙 Lullio) centered at top
   - Headlines in Instrument Serif italic, body in DM Sans
   - Sign Up fields: name, email, password, partner invite code
   - Must respect dark mode via ThemeProvider

4. TypeScript types (types/database.ts):
   - households (id, created_at)
   - profiles (id, household_id, full_name, role, avatar_url)
   - shifts (id, household_id, parent_id, started_at, ended_at, status, ai_summary, watch_for)
   - events (id, shift_id, parent_id, event_type, category, label, quantity, unit, duration_seconds, note, logged_at)
   - briefings (id, shift_id, from_parent_id, to_parent_id, summary, watch_for, stats, created_at, read_at)

5. Data hooks (hooks/useLullio.ts):
   - useCurrentShift() — get or create active shift
   - useShiftEvents(shiftId) — real-time subscription to events for a shift
   - useLatestBriefing() — get most recent unread briefing
   - useLogEvent() — mutation to insert an event
```

---

### Prompt 3: Quick Tags System

```
Build the Quick Tags input system for Lullio. Reference lullio-design-tokens.md for all styling, colors, and icon choices.

Design principle: "One hand, one eye, three seconds."

CRITICAL — ICON/EMOJI SPLIT: All category icons, event timeline icons, and UI elements use Lucide React Native icons — NOT emoji. Emoji only appears in conversational copy (headlines, toasts). See the "Iconography" section of lullio-design-tokens.md for the full rules.

1. QuickTagsScreen (screens/log/QuickTags.tsx):
   - Top: "Suggested right now" — 3 AI-suggested tags (hardcoded for now)
   - Category grid: 2-column grid of category cards
   - Bottom: scrollable timeline of events logged this shift

2. Tag taxonomy (constants/tags.ts) — define with Lucide icon references:
   - Feeding (Lucide: Utensils, color: #F59E0B): Bottle (qty: oz), Left Breast (timer), Right Breast (timer), Solids (note), Pumped (qty: oz)
   - Sleep (Lucide: Moon, color: #6366F1): Fell Asleep, Woke Up, Nap Started, Nap Ended, Fussy/Restless
   - Diaper (Lucide: Droplets, color: #10B981): Wet, Dirty, Both, Dry, Blowout
   - Health (Lucide: Stethoscope, color: #EF4444): Medication (note required), Temperature (qty: °F), Spit Up, Gas, Rash (note)
   - Activity (Lucide: Sparkles, color: #8B5CF6): Tummy Time (timer), Bath, Walk/Outside, Play Time
   - Mood (Lucide: Heart, color: #EC4899): Happy, Calm, Cranky, Crying, Clingy

3. Category card styling:
   - Background: category bg tint from theme (light: feedingBg #FEF3C7, dark: #2A2215, etc.)
   - Lucide icon in a 40px circle, colored with category primary
   - Border radius: xl (16px) from tokens
   - Cards must adapt to dark mode using ThemeProvider

4. Tag detail screens:
   - QuantityPicker: circular buttons for preset values, selected state uses category color
   - TimerView: large timer in DM Sans weight 300, start/pause buttons use category color
   - NoteInput: optional textarea, required only for meds/rash

5. Interaction flow:
   - Simple tags (mood, basic diaper, sleep): ONE TAP logs instantly
   - Tags with details (bottle, temp, meds): tap opens detail sheet
   - Timer tags (breastfeeding, tummy time): tap opens timer

6. After logging:
   - Insert into Supabase events table via useLogEvent()
   - Show toast: dark pill (#1A1A18 in light mode, #F3EDE5 in dark), text: "{label} logged ✓"
   - Note: the ✓ in the toast is one of the few emoji-in-conversational-voice uses
   - Haptic: expo-haptics medium impact on successful log

7. Use react-native-reanimated for toast animation (slide from top, 2.2s auto-dismiss).
```

---

### Prompt 4: Voice Memo Input

```
Build the voice memo logging system for Lullio. Reference lullio-design-tokens.md for styling.

1. VoiceRecorder component (components/VoiceRecorder.tsx):
   - Large circular record button — use destructive color (#EF4444 light, #C65A5A dark)
   - Pulses while recording using react-native-reanimated
   - Recording timer in DM Sans weight 300 (same style as tag timers for consistency)
   - Waveform visualization: simple amplitude bars using expo-av metering
   - Stop button replaces record button while active
   - After recording: playback controls + "Process" button (primary color)
   - All icons use Lucide: Mic, Square, Play, Pause

2. Recording flow (expo-av):
   - Request microphone permissions on first use
   - Record as m4a (AAC)
   - Max recording length: 5 minutes
   - Store temporarily in app cache

3. Processing pipeline (services/voiceProcessing.ts):
   - Upload audio to Supabase Storage (bucket: "voice-memos")
   - Mock transcription service for now (hardcoded transcript)
   - Display transcript with "Looks good" / "Try again" options

4. AI Event Parsing (services/eventParser.ts):
   - Calls Claude Haiku API to extract structured events from transcript
   - Prompt: "Extract baby care events from this parent's voice memo. Return JSON array with fields: category, label, quantity, unit, duration_seconds, note, approximate_time. Categories: feeding, sleep, diaper, health, activity, mood."
   - Display parsed events as editable cards using category colors and Lucide icons
   - "Confirm & Log All" inserts into Supabase

5. InputModeSwitcher (components/InputModeSwitcher.tsx):
   - Segmented control at bottom of Log screen: Voice | Tags | Type
   - Uses Lucide icons: Mic, Tag, Keyboard
   - Active segment: card bg with shadow, foreground text
   - Inactive: transparent, muted-foreground text
   - Smooth crossfade between modes
   - Remember last used mode in AsyncStorage

Style the recorder to feel calm — not like a podcast app. Big button, minimal UI.
```

---

### Prompt 5: Text Input (Fallback)

```
Build the text input fallback for Lullio. Reference lullio-design-tokens.md for styling.

1. TextLogScreen (screens/log/TextLog.tsx):
   - Single large text input, auto-focused on mount
   - Placeholder: "What happened? e.g., 'She ate 4oz at 2pm, been fussy since'"
   - Send button: Lucide ArrowUp icon in a primary-colored circle, bottom-right
   - Recent quick phrases above keyboard: chips with muted bg, showing last 5 unique phrases

2. On submit:
   - Same AI event parsing pipeline as voice memos (services/eventParser.ts)
   - Show parsed events as editable cards with Lucide category icons
   - "Confirm & Log All" inserts into Supabase

3. This screen shares the InputModeSwitcher so all three modes are accessible from the Log tab.

Keep it minimal — just a text box and AI on the other end.
```

---

### Prompt 6: Shift Flow (Start → Active → End → Handoff)

```
Build the shift lifecycle for Lullio. Reference lullio-design-tokens.md for ALL styling. This is the Home tab.

Four states, one screen:

STATE 1 — IDLE:
- Centered layout, calm and spacious
- Large circular "Start Shift" button: primary color bg (#0F3460 light, #D78A2D dark)
- Subtle pulsing ring animation (react-native-reanimated)
- Heading: "Ready when you are" in Instrument Serif italic
- Subtext in DM Sans: "Everything you log will be bundled into a briefing for {partnerName}"
- Bottom: partner avatar initial in a circle + "{partnerName}'s last shift ended at {time}"
- Tap creates shift in Supabase (status: 'active')

STATE 2 — ACTIVE:
- Status badge: success-muted bg, success-foreground text, pulsing green dot
- "SHIFT ACTIVE" label, elapsed time right-aligned
- Hero: "You're on, {name} ☀️" in Instrument Serif italic (emoji in headline = conversational voice, this is correct)
- Quick actions row: 3 buttons using Lucide icons (Mic, Tag, Keyboard) on category-tinted circles
- Event feed: card container with Lucide category icons in colored circles, DM Sans text
- "End Shift & Hand Off →" button: destructive outline (2px border, transparent bg). Uses Lucide ArrowRight. On press: fills destructive, text goes white.

STATE 3 — ENDING (Review & Send):
- Dark header using primary gradient
- "Hand off to {partnerName}" in Instrument Serif italic, white
- AI summary card: editable textarea pre-filled with AI output, Lucide Sparkles icon
- Watch For section: warning-colored cards with Lucide TriangleAlert icon
- Stats row: Lucide icons (Moon, Utensils, Droplets) with category bg tints
- Personal voice note option (reuse VoiceRecorder), Lucide Mic icon
- "Send to {partnerName}" button: success color, Lucide Send icon

STATE 4 — SENT:
- Checkmark animation (scale-in, react-native-reanimated)
- Lucide Check icon in success-colored circle
- "Handed off 💚" in Instrument Serif italic (emoji = conversational voice)
- "You're off duty — get some rest." in DM Sans
- Summary card with shift duration + event count
- "Back to home" text link

All states must render correctly in both light and dark mode.
```

---

### Prompt 7: Briefing Screen

```
Build the Briefing screen for Lullio. Reference lullio-design-tokens.md. This is the CORE VALUE PROP.

Layout (top to bottom):

1. HERO (dark gradient header):
   - In light mode: linear-gradient(135deg, #1A1A2E, #0F3460)
   - In dark mode: linear-gradient(135deg, #0A0E14, #12161B) — darker variant
   - Shift type: "🌙 Overnight Shift" or "☀️ Morning Shift" (emoji = conversational voice, correct here)
   - Greeting: "Good morning, {name}" in Instrument Serif italic, white
   - Info line: DM Sans, white at 60% opacity

2. AI SUMMARY (floating card overlapping hero):
   - Lucide Sparkles icon (accent color) + "AI Summary" in Instrument Serif italic
   - "Auto-generated" badge: muted bg, muted-foreground text
   - Summary text: DM Sans 15px, 1.55 line height

3. SHIFT STATS (3-column grid):
   - Lucide icons: Moon (sleep), Utensils (fed), Droplets (diapers) — colored with category tokens
   - Background: category bg tints (adapt for dark mode)
   - Values: DM Sans 18px weight 600

4. WATCH FOR:
   - Warning cards: warning bg, warning-border, warning-foreground
   - Lucide TriangleAlert icon, 20px
   - Title: DM Sans 15px weight 600

5. WHAT HAPPENED (expandable timeline):
   - Lucide category icons in colored circles (15% opacity fill, 30% border — 20% fill in dark mode)
   - Show 4 events, "Show all {n} events" expander
   - DM Sans for all text

6. COMING UP:
   - Lucide category icons in muted circles
   - DM Sans, muted-foreground for predictions
   - "Based on patterns from the last 7 days" in DM Sans italic, 11px

7. FIXED BOTTOM CTA:
   - "Got it — Starting my shift ☀️" (emoji = conversational voice)
   - Primary color bg, rounded lg
   - Tapping: marks briefing as read, starts new shift

Must work in both light and dark mode.
```

---

### Prompt 8: AI Briefing Engine

```
Build the AI briefing generation service for Lullio.

Create services/briefingEngine.ts:

1. generateBriefing(shiftId: string):

   a. Fetch all events for the shift from Supabase, ordered by logged_at

   b. Compute shift stats:
      - totalSleep: sum of sleep durations (fell asleep → woke up pairs)
      - totalFed: sum of feeding quantities
      - diaperCount: count of diaper events
      - longestStretch: longest gap between wake events
      - overallMood: most frequent mood, or "No mood logged"

   c. Call Claude Sonnet API (model: claude-sonnet-4-20250514) with system prompt:

      "You are the AI engine inside Lullio, a co-parenting app. A parent just finished their shift caring for the baby. Generate a warm, clear, concise briefing for the OTHER parent who is about to take over.

      Rules:
      - Write like a knowledgeable friend, not a medical professional
      - Lead with the most important information
      - Be specific (use times, quantities, durations)
      - Flag anything unusual or concerning
      - Keep summary to 3-4 sentences max
      - Keep watch_for items to 2-3 bullets, each one sentence
      - Predict 2-3 upcoming events based on timing patterns

      Respond in JSON:
      {
        "summary": "string",
        "watch_for": ["string", "string"],
        "predicted_next": [{"time": "string", "label": "string", "icon": "string"}],
        "shift_vibe": "smooth" | "rough" | "mixed"
      }

      NOTE: For the 'icon' field in predicted_next, use Lucide icon names (e.g., 'Moon', 'Utensils', 'Droplets'). Do NOT use emoji."

   d. Parse response, insert into briefings table

   e. Push notification: "{parentName} just handed off — tap to see the briefing"

2. Error handling:
   - AI call fails → fall back to basic stats-only summary
   - Retry once on timeout
   - Log errors to Supabase error_logs table

3. Supabase Edge Function (supabase/functions/generate-briefing/index.ts):
   - Accepts shiftId, calls generateBriefing(), returns briefing data
   - Keeps API key server-side

Use @anthropic-ai/sdk for the API call.
```

---

### Prompt 9: Polish & Onboarding

```
Final polish pass on Lullio. Reference lullio-design-tokens.md for everything.

1. ONBOARDING (3 screens, first launch):
   - Screen 1: "Parenting is a team sport" — Lucide icons (Users, ArrowRightLeft) as illustration elements. One sentence: "Log your shift. AI builds the briefing. Your partner knows everything."
   - Screen 2: "Three ways to log" — Lucide icons: Mic, Tag, Keyboard with one-line descriptions
   - Screen 3: "Invite your co-parent" — email input + "Skip for now"
   - Headlines in Instrument Serif italic, body in DM Sans
   - Dot indicators, horizontal paging
   - Store onboarding_complete in AsyncStorage
   - Must work in dark mode

2. PUSH NOTIFICATIONS (expo-notifications):
   - Request permission during onboarding
   - Register device token in Supabase profiles
   - Triggers: briefing received, partner started shift, 2+ hours no events reminder

3. EMPTY STATES:
   - Use Lucide icons as gentle illustrations (FileText for no briefings, SquarePen for no events, History for no history)
   - Warm copy: "No briefings yet — check back after {partnerName}'s next shift"
   - DM Sans, muted-foreground color

4. HAPTIC FEEDBACK (expo-haptics):
   - Light: tag tap
   - Medium: event logged confirmation
   - Success: shift handoff sent

5. APP ICON & SPLASH:
   - Icon: deep navy (#0F3460) background, white crescent moon
   - Splash: background color from theme, 🌙 + "Lullio" in Instrument Serif italic centered
   - Configure in app.json

6. DARK MODE — "NIGHTLIGHT MODE":
   - Already implemented via ThemeProvider from Prompt 1
   - Verify ALL screens render correctly in dark mode
   - Key checks:
     a. Category bg tints use dark variants (e.g., feedingBg: #2A2215 not #FEF3C7)
     b. Event icon circles use 20% opacity (not 15%) for visibility
     c. Primary flips from navy to amber (#D78A2D)
     d. Destructive red softens to #C65A5A
     e. Warning cards use #3A2C1E bg with #FFDFA8 text
     f. Border uses white at 8% opacity (#FFFFFF14)
   - Add a manual toggle in Settings (optional) in addition to system detection

7. FINAL CONSISTENCY CHECK:
   - All spacing on 8px grid
   - All card radius: 16px (xl)
   - All pill radius: 100px
   - All headlines: Instrument Serif italic
   - All body text: DM Sans
   - All UI icons: Lucide (never emoji)
   - All conversational copy: emoji allowed (headlines, toasts, confirmations, brand mark)
   - Test every screen in both light and dark mode
```

---

## Part 2: Design Tool Prompts

### Banani Prompt

Use in [app.banani.co](https://app.banani.co). Generate light and dark versions separately.

```
Design a mobile app called "Lullio" — an AI-powered co-parenting shift-change briefing app for new parents.

DESIGN SYSTEM:
- Background: #FAFAF7 (light) / #12161B (dark)
- Foreground: #1A1A18 (light) / #F3EDE5 (dark)
- Cards: #FFFFFF (light) / #181D23 (dark)
- Primary: #0F3460 (light navy) / #D78A2D (dark amber — primary flips in dark mode)
- Accent: #E08A12 / #F1C76A
- Success: #10B981 / #2C8C63
- Borders: #F0EDE8 (light) / #FFFFFF14 (dark)
- Warning: #FFFBEB + #FDE68A border (light) / #3A2C1E + #5A4020 border (dark)
- Category colors: Feeding #F59E0B, Sleep #6366F1, Diaper #10B981, Health #EF4444, Activity #8B5CF6, Mood #EC4899

TYPOGRAPHY:
- Headlines and section titles: serif italic (warm, personal)
- Body text: clean sans-serif (DM Sans style)
- Never use Inter or system fonts for display text

ICONOGRAPHY — CRITICAL:
- ALL UI icons use Lucide line icons (consistent stroke weight, colored with category tokens)
- Category icons: Utensils (feeding), Moon (sleep), Droplets (diaper), Stethoscope (health), Sparkles (activity), Heart (mood)
- Navigation: House, SquarePen, FileText, History
- Do NOT use emoji as UI elements. Emoji only appears in headline text like "You're on, Brett ☀️"

COMPONENTS:
- Cards: 1px border, no shadows, 12-16px radius
- Event list items: 40px icon circles with category color at 15% opacity fill
- Stat cards: category bg tint with Lucide icon + value + label
- Alert cards: warning bg + border, Lucide TriangleAlert
- End shift button: red OUTLINE only (2px border, transparent bg)
- Tab bar: Lucide icons, active = foreground, inactive = muted

GENERATE 6 SCREENS:
1. SHIFT START: Centered start button, serif italic "Ready when you are", tab bar
2. ACTIVE SHIFT: Green badge, serif italic "You're on, Brett ☀️", AI summary + stats, quick log grid with Lucide icons, event feed, red outline end-shift button
3. QUICK TAGS: 2-col category grid with Lucide icons, smart suggestions, input mode switcher
4. BRIEFING: Dark hero, serif italic greeting, AI summary card, stats with Lucide icons, watch-for cards, timeline, "Got it — Starting my shift ☀️" CTA
5. TAG DETAIL (Bottle): Utensils icon + "Bottle", circular oz selectors, note field, amber log button
6. HANDOFF SENT: Green checkmark, serif italic "Handed off 💚", shift summary
```

---

### Google Stitch Prompt

Use in [stitch.withgoogle.com](https://stitch.withgoogle.com).

```
Design a mobile app called "Lullio" — an AI-powered co-parenting shift-change briefing app. Generate 6 screens for iPhone 15 Pro.

Design direction: warm, calm, reassuring. Serif italic headlines, clean sans-serif body. Off-white background (#FAFAF7), navy primary (#0F3460), amber accent (#E08A12), green success (#10B981). Use Lucide-style line icons for all UI elements — NOT emoji. Emoji only in headline text. Category colors: Feeding amber, Sleep indigo, Diaper green, Health red, Activity purple, Mood pink. Cards have 1px borders, no shadows, 12-16px radius.

Screen 1 — SHIFT START: Large circular start button (navy), "Ready when you are" serif italic, partner info, 4-tab bar with line icons.
Screen 2 — ACTIVE SHIFT: Green "Shift active" badge, "You're on, Brett ☀️" serif italic, AI summary with sparkle icon + 3-col stats, quick log grid with line icons, event feed, red outline end-shift button.
Screen 3 — QUICK TAGS: 2-col category grid with line icons on colored circles, smart suggestions, mode switcher.
Screen 4 — BRIEFING: Dark gradient hero, serif italic greeting, floating AI summary, stat cards with line icons, amber watch-for cards, timeline, "Got it — Starting my shift ☀️" CTA.
Screen 5 — BOTTLE DETAIL: Utensils icon + "Bottle", circular oz selectors, note field, amber log button.
Screen 6 — HANDOFF SENT: Green checkmark, "Handed off 💚" serif italic, shift summary.

Consistent tab bar on all screens, line icons only, warm premium aesthetic.
```

---

### Tips for Design Tools

1. **Generate light mode first**, then ask for dark mode variants with the dark palette
2. **Banani** lets you upload screenshots — use our React prototypes as starting input
3. **Stitch** is best for quick multi-screen generation to see all screens side by side
4. **Export to Figma** from either tool to finalize with precise token values
5. **Check dark mode category tints** — light pastels need to become dark-tinted versions, not just darkened pastels
