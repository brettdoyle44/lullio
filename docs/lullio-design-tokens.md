# Lullio — Design Tokens & System

> Single source of truth for Lullio's visual identity.  
> Use this as a reference for Cursor, Figma, Stitch, Banani, or any design tool.

---

## Brand Essence

**Voice:** A warm, calm friend who happens to know everything about your baby's day.  
**Not:** A medical dashboard. A corporate SaaS app. A clinical tracker.  
**Feels like:** A premium baby product that's genuinely useful.  
**The test:** Would a sleep-deprived parent at 3 AM feel reassured looking at this?

---

## Typography

### Font Pairing

| Role | Font | Weight | Style | Usage |
|------|------|--------|-------|-------|
| Display | Instrument Serif | 400 | Italic | Hero headlines, section titles, brand name |
| Body | DM Sans | 300–600 | Normal | Everything else |

**Why this pairing:** Instrument Serif italic gives Lullio its personality — it feels human, personal, like a handwritten note. DM Sans is warm enough to not feel clinical but clean enough to stay readable at small sizes and on tired eyes.

**Never use:** Inter, Roboto, SF Pro, or any system font as the display face. These make the app feel like a generic dashboard.

### Type Scale

| Token | Size | Weight | Font | Tracking | Leading | Usage |
|-------|------|--------|------|----------|---------|-------|
| `hero` | 32px | 400 | Display, italic | -0.5px | 1.1 | "You're on, Brett ☀️" |
| `hero-sm` | 26px | 400 | Display, italic | -0.4px | 1.15 | Screen sub-headlines |
| `section` | 20px | 400 | Display, italic | -0.3px | 1.2 | "This shift", "Watch for" |
| `card-title` | 17px | 400 | Display, italic | 0 | 1.3 | "AI Summary" |
| `body` | 15px | 400 | Body | 0 | 1.55 | Paragraphs, summary text |
| `body-sm` | 14px | 400 | Body | 0 | 1.5 | Alert text, secondary info |
| `caption` | 13px | 500 | Body | 0 | 1.4 | Event notes, timestamps |
| `label` | 12px | 600 | Body | 0.5px | 1 | Badges, tab labels, stat labels |
| `overline` | 11px | 700 | Body, uppercase | 1.2px | 1 | Section overlines (rare) |
| `stat` | 18px | 600 | Body | 0 | 1 | "6h 50m", "9oz" |

---

## Color System

### Core Palette

| Token | Hex | Usage |
|-------|-----|-------|
| `background` | `#FAFAF7` | App background — warm off-white, easy on eyes at 3 AM |
| `foreground` | `#1A1A18` | Primary text — almost-black with warm undertone |
| `card` | `#FFFFFF` | Card backgrounds |
| `card-foreground` | `#1A1A18` | Text on cards |
| `border` | `#F0EDE8` | Card borders, dividers — warm, subtle |
| `muted` | `#F1ECE9` | Inactive backgrounds, secondary fills |
| `muted-foreground` | `#9A9A94` | Secondary text, timestamps, placeholders |

### Primary — Deep Navy

| Token | Hex | Usage |
|-------|-----|-------|
| `primary` | `#0F3460` | CTAs, active states, brand accents, dark hero headers |
| `primary-foreground` | `#FFFFFF` | Text on primary |

**Why navy:** Trust, calm, nighttime association. It anchors the app without being loud.

### Accent — Warm Amber

| Token | Hex | Usage |
|-------|-----|-------|
| `accent` | `#E08A12` | Feeding category, highlight moments |
| `accent-foreground` | `#FFFFFF` | Text on accent |
| `accent-muted` | `#FEF3C7` | Light amber backgrounds |

**Role:** The amber adds energy and warmth. Used sparingly as a counterpoint to the navy. Never as a primary CTA color.

### Semantic Colors

| Token | Hex | Usage |
|-------|-----|-------|
| `success` | `#10B981` | Confirmations, "shift active" dot, send button |
| `success-muted` | `#D1FAE5` | Success badge backgrounds |
| `success-foreground` | `#053927` | Text on success backgrounds |
| `destructive` | `#EF4444` | End shift button (outline only), errors |
| `destructive-muted` | `#FEE2E2` | Error backgrounds |
| `warning` | `#FFFBEB` | Watch-for card backgrounds |
| `warning-foreground` | `#92400E` | Watch-for text |
| `warning-border` | `#FDE68A` | Watch-for card borders |

### Category Colors

Each category gets a primary color and a light background tint.

| Category | Primary | Background | Emoji |
|----------|---------|------------|-------|
| Feeding | `#F59E0B` | `#FEF3C7` | 🍼 |
| Sleep | `#6366F1` | `#EEF2FF` | 😴 |
| Diaper | `#10B981` | `#D1FAE5` | 👶 |
| Health | `#EF4444` | `#FEE2E2` | 🩺 |
| Activity | `#8B5CF6` | `#F3E8FF` | ✨ |
| Mood | `#EC4899` | `#FCE7F3` | 🎭 |

**Usage rules:**
- Category colors appear on tag pills, event timeline dots, stat card backgrounds
- Never use more than 3 category colors on a single screen
- The light background tint is for fills; the primary color is for borders and small accents
- Event icons use the category color at 15% opacity for the circle fill, 30% for the border

### Dark Mode — "Nightlight Mode"

Triggered automatically by system preference (`useColorScheme()`), or manually via an in-app toggle. Optimized for 3 AM feedings — warm-dark tones that don't blast a parent's eyes.

**Design intent:** The dark palette skews warm blue-black rather than pure black. This avoids the cold, technical feel of most dark modes and matches the nighttime parenting context. The amber accent becomes the hero — it reads like warm candlelight on a dark surface.

#### Core Palette

| Token | Light | Dark | Notes |
|-------|-------|------|-------|
| `background` | `#FAFAF7` | `#12161B` | Deep blue-black, not pure black |
| `foreground` | `#1A1A18` | `#F3EDE5` | Warm off-white, not pure white |
| `card` | `#FFFFFF` | `#181D23` | Subtle lift from background |
| `card-foreground` | `#1A1A18` | `#F3EDE5` | Matches foreground |
| `border` | `#F0EDE8` | `#FFFFFF14` | White at 8% opacity — very subtle |
| `muted` | `#F1ECE9` | `#1B2229` | Dark slate for inactive fills |
| `muted-foreground` | `#9A9A94` | `#9AA3AC` | Slightly cool gray — readable, not bright |

#### Primary & Accent

| Token | Light | Dark | Notes |
|-------|-------|------|-------|
| `primary` | `#0F3460` | `#D78A2D` | Navy → amber. The primary flips in dark mode to avoid invisible navy-on-dark |
| `primary-foreground` | `#FFFFFF` | `#FFF8F0` | Warm white on amber |
| `accent` | `#E08A12` | `#F1C76A` | Brighter gold for visibility |
| `accent-foreground` | `#FFFFFF` | `#3B2A08` | Dark brown on gold |

**Key decision:** The primary color inverts from navy to amber in dark mode. Navy is invisible on dark backgrounds, so amber takes over as the dominant brand color. This keeps the app feeling warm and recognizable in both modes.

#### Semantic Colors

| Token | Light | Dark |
|-------|-------|------|
| `success` | `#10B981` | `#2C8C63` | Slightly muted green — less vibrant at night |
| `success-muted` | `#D1FAE5` | `#1A2A27` | Dark green tint |
| `success-foreground` | `#053927` | `#BFE8D4` | Light green text on dark green |
| `destructive` | `#EF4444` | `#C65A5A` | Softened red — less alarming at 3 AM |
| `destructive-muted` | `#FEE2E2` | `#2A1A1A` | Dark red tint |
| `warning` | `#FFFBEB` | `#3A2C1E` | Dark amber tint — warm, not alarming |
| `warning-foreground` | `#92400E` | `#FFDFA8` | Light amber text |
| `warning-border` | `#FDE68A` | `#5A4020` | Muted amber border |

#### Category Colors (Dark Mode Adjustments)

Category primary colors stay the same across modes, but their background tints shift to dark-tinted versions:

| Category | Primary (both) | Light bg | Dark bg |
|----------|----------------|----------|---------|
| Feeding | `#F59E0B` | `#FEF3C7` | `#2A2215` |
| Sleep | `#6366F1` | `#EEF2FF` | `#1A1A2E` |
| Diaper | `#10B981` | `#D1FAE5` | `#1A2A22` |
| Health | `#EF4444` | `#FEE2E2` | `#2A1A1A` |
| Activity | `#8B5CF6` | `#F3E8FF` | `#221A2E` |
| Mood | `#EC4899` | `#FCE7F3` | `#2A1A22` |

**Usage rules:**
- Event icon circles: category color at 20% opacity fill (slightly higher than light mode's 15% for visibility)
- Stat cards use the dark category bg fills
- The "shift active" dot remains full `success` green — it should always be clearly visible

#### Implementation Notes

```typescript
// Theme provider structure
const lightTheme = {
  background: '#FAFAF7',
  foreground: '#1A1A18',
  card: '#FFFFFF',
  primary: '#0F3460',
  // ...
};

const darkTheme = {
  background: '#12161B',
  foreground: '#F3EDE5',
  card: '#181D23',
  primary: '#D78A2D',  // ← flips to amber
  // ...
};
```

---

## Spacing

8px base grid. All spacing is a multiple of 4.

| Token | Value | Usage |
|-------|-------|-------|
| `xs` | 4px | Tight gaps (between icon and text in a badge) |
| `sm` | 8px | Grid gaps, stat card gap, inline spacing |
| `md` | 12px | Card internal gaps, section title to content |
| `lg` | 16px | Card padding, list item padding |
| `xl` | 20px | Screen padding (horizontal), section gaps |
| `xxl` | 24px | Large section spacing |
| `section` | 28px | Between major content sections |

---

## Border Radius

| Token | Value | Usage |
|-------|-------|-------|
| `sm` | 8px | Badges, small tags |
| `md` | 12px | Stat cards, inner containers |
| `lg` | 14px | Buttons |
| `xl` | 16px | Cards, lists, major containers |
| `pill` | 100px | Shift badge, tab pills |
| `full` | 999px | Avatars, icon circles, dots |

---

## Components

### Cards
- Background: `card`
- Border: 1px solid `border`
- Radius: `xl` (16px)
- Padding: 18px
- No shadow in light mode (borders only)

### Event List Items
- Padding: 16px
- Divider: 1px solid `border`
- Icon: 40×40px circle, category color at 15% opacity fill, 30% border
- Lucide icon centered in circle, 18px, category color, strokeWidth 2
- Name: `body` size, weight 500
- Note: `caption` size, `muted-foreground` color
- Time: `caption` size, `muted-foreground`, right-aligned

### Stat Cards
- Background: category light background color
- Radius: `md` (12px)
- Padding: 12px vertical, 8px horizontal
- Lucide icon (18px, category color) → value → label, centered vertically
- Grid: 3 columns, 8px gap

### Alert Cards (Watch For)
- Background: `warning`
- Border: 1px solid `warning-border`
- Radius: `xl`
- Padding: 16px
- Icon left (emoji, 18px), text right
- Title: `body` size, weight 600, `warning-foreground`
- Body: `body-sm` size, `warning-foreground`

### End Shift Button
- **Outline style** — NOT filled
- Border: 2px solid `destructive`
- Color: `destructive`
- Background: transparent
- Radius: `lg` (14px)
- Padding: 16px
- On press: fills `destructive`, text goes white
- Rationale: Easy to find, hard to accidentally tap

### Tab Bar
- Background: `card`
- Border top: 1px solid `border`
- Grid: 4 equal columns
- Active: `foreground` color, weight 600, strokeWidth 2.2
- Inactive: `muted-foreground` color, weight 500, strokeWidth 1.8
- Icons: Lucide (20px) with label below (12px)
- Padding: 8px top, 14px bottom (safe area consideration)

---

## Iconography — The Icon/Emoji Split

Lullio uses two visual systems with strict separation:

### Lucide Icons → All UI Elements

**Package:** `lucide-react-native` (React Native) / `lucide-react` (web)

Every repeating, structural UI element uses Lucide line icons. This ensures visual consistency, color control, and cross-platform fidelity.

| Context | Example Icons | Size | Stroke |
|---------|--------------|------|--------|
| Category buttons | `Utensils`, `Moon`, `Droplets`, `Stethoscope`, `Sparkles`, `Heart` | 20px | 2 |
| Event timeline | Same as categories, colored with category token | 18px | 2 |
| Stat cards | `Moon`, `Utensils`, `Droplets` | 18px | 2 |
| Tab bar (active) | `Home`, `SquarePen`, `FileText`, `History` | 20px | 2.2 |
| Tab bar (inactive) | Same icons | 20px | 1.8 |
| Alert cards | `TriangleAlert` | 20px | 2 |
| Coming up list | Category-matched icons | 16px | 2 |
| Navigation arrows | `ArrowRight`, `ArrowLeft` | 16px | 2.5 |
| Quick actions | `Mic`, `Tag`, `Keyboard` | 20px | 2 |

**Icon coloring rules:**
- Category buttons: icon uses category primary color on category bg circle
- Event timeline: icon uses category color at full opacity, circle fill at 15%, circle border at 30%
- Stat cards: icon uses category primary color
- Tab bar: active uses `foreground`, inactive uses `muted-foreground`
- Alerts: icon uses `warning-foreground`
- Coming up: icon uses `muted-foreground` on `muted` circle

**Category → Icon mapping:**

| Category | Lucide Icon | Color |
|----------|------------|-------|
| Feeding | `Utensils` | `#F59E0B` |
| Sleep | `Moon` | `#6366F1` |
| Diaper | `Droplets` | `#10B981` |
| Health | `Stethoscope` | `#EF4444` |
| Activity | `Sparkles` | `#8B5CF6` |
| Mood | `Heart` | `#EC4899` |

### Emoji → Conversational Voice Only

Emoji appear only where the app is *speaking to the parent* — in moments of personality, warmth, and human connection. They never appear as repeating structural elements.

| Context | Example | Rationale |
|---------|---------|-----------|
| Hero headlines | "You're on, Brett ☀️" | The app is greeting you like a friend |
| Brand mark | 🌙 Lullio | Signature identity element |
| Toast confirmations | "Bottle logged ✓" | Momentary celebration |
| Handoff confirmation | "Handed off 💚" | Emotional punctuation |
| Shift type labels | "🌙 Overnight Shift" | Setting the scene |
| Onboarding copy | "Parenting is a team sport 🤝" | First impression warmth |
| Empty states | Contextual emoji illustration | Softens "nothing here yet" |

**Why this split:** Emoji render differently across platforms — what looks warm on iOS can look flat on Android. For UI elements that repeat (event lists, stat cards, categories), inconsistent rendering breaks visual cohesion. Lucide icons are pixel-perfect across every device. But in conversational moments — headlines, confirmations, the brand mark — emoji add the human warmth that icons can't match. The split gives you consistency *and* personality.

---

## Animation

### Page Load
- Staggered fade-up reveal: opacity 0→1, translateY 12px→0
- Duration: 450ms ease
- Stagger: 50ms between elements
- Apply to all major sections on screen mount

### Shift Active Dot
- Pulse animation: scale 0.9→1.1, opacity 0.6→1
- Duration: 2s ease-in-out infinite

### Tap Feedback
- Scale to 0.97 on press, 150ms ease
- Haptic: light on tag tap, medium on log confirmation, success on handoff

### Toast Notification
- Slide down from top: translateY -80px→0
- Duration: 300ms ease
- Auto-dismiss: 2.2 seconds
- Background: `foreground` (dark), text: white, radius: `pill`

---

## Brand Elements

### Logo Treatment
- Emoji (🌙) + "Lullio" in Instrument Serif italic
- Spacing: 10px between emoji and text
- Size: emoji 22px, text 22px
- Color: `foreground` (adapts to dark mode)
- The moon emoji reflects overnight shifts — the app's core use case

### App Icon Concept
- Background: `primary` (deep navy)
- Foreground: simple white crescent moon or two overlapping speech bubbles
- Radius: iOS standard (continuous corner)

### Splash Screen
- Background: `background`
- Center: 🌙 emoji (48px) + "Lullio" in Instrument Serif italic (36px)
- Subtle fade-in on load
