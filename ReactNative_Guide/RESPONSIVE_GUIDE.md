# RESPONSIVE_SYSTEM.md
### Complete Responsive Design Reference — React Native / Expo

> Drop this file in your project root. Every developer on the team should read it before touching layouts or typography.

---

## Table of Contents

1. [Reference: `fonts.ts` Source Code](#1-reference-fontsts-source-code)
2. [Usage Guide: Scaling Functions](#2-usage-guide-scaling-functions)
3. [Typography: FONT_SIZE Constants](#3-typography-font_size-constants)
4. [Before vs. After: Responsive Refactors](#4-before-vs-after-responsive-refactors)
5. [Tablet Breakpoints with `useWindowDimensions`](#5-tablet-breakpoints-with-usewindowdimensions)
6. [Summary Checklist](#6-summary-checklist)

---

## 1. Reference: `fonts.ts` Source Code

> **Location:** `src/constants/fonts.ts`
> Copy this entire file into a new project to get started immediately.

```typescript
import { RFValue } from "react-native-responsive-fontsize";
import { Dimensions, PixelRatio } from "react-native";

const { width: SCREEN_WIDTH, height: SCREEN_HEIGHT } = Dimensions.get("window");

// Base dimensions for scaling (iPhone 11 / standard modern screen)
const guidelineBaseWidth  = 375;
const guidelineBaseHeight = 812;

// ── Layout Scaling ─────────────────────────────────────────────────────────

export const horizontalScale = (size: number) =>
  (SCREEN_WIDTH / guidelineBaseWidth) * size;

export const verticalScale = (size: number) =>
  (SCREEN_HEIGHT / guidelineBaseHeight) * size;

export const moderateScale = (size: number, factor = 0.5) =>
  size + (horizontalScale(size) - size) * factor;

// ── Font Sizing ────────────────────────────────────────────────────────────

export const FONT_SIZE = {
  xs:      RFValue(10, 812),
  small:   RFValue(12, 812),
  regular: RFValue(14, 812),
  medium:  RFValue(16, 812),
  large:   RFValue(18, 812),
  xl:      RFValue(22, 812),
  xxl:     RFValue(28, 812),
  xxxl:    RFValue(34, 812),
};

export const FONT_WEIGHT = {
  regular:  "400" as const,
  medium:   "500" as const,
  semibold: "600" as const,
  bold:     "700" as const,
};

export const LINE_HEIGHT = {
  tight:  1.2,
  normal: 1.5,
  loose:  1.8,
};

// ── Composite Text Styles ──────────────────────────────────────────────────

export const TEXT_STYLE = {
  title: {
    fontSize:   FONT_SIZE.xl,
    fontWeight: FONT_WEIGHT.bold,
  },
  subtitle: {
    fontSize:   FONT_SIZE.large,
    fontWeight: FONT_WEIGHT.medium,
  },
  body: {
    fontSize:   FONT_SIZE.regular,
    fontWeight: FONT_WEIGHT.regular,
  },
  caption: {
    fontSize:   FONT_SIZE.small,
    fontWeight: FONT_WEIGHT.regular,
  },
};
```

### How the Math Works

| Constant | Formula | Reference Base |
|----------|---------|---------------|
| `horizontalScale(size)` | `(SCREEN_WIDTH / 375) × size` | iPhone 11 width: 375pt |
| `verticalScale(size)` | `(SCREEN_HEIGHT / 812) × size` | iPhone 11 height: 812pt |
| `moderateScale(size, f)` | `size + (horizontalScale(size) - size) × f` | Blended scale (default 50%) |
| `RFValue(size, 812)` | Screen-height-relative font scaling | 812pt standard screen height |

> **Why iPhone 11?** At 375×812pt logical pixels it represents the most common mid-range design target. Designs from Figma at this artboard size translate 1:1 without any mental math.

---

## 2. Usage Guide: Scaling Functions

### Quick Decision Chart

```
Need to scale something?
│
├── Is it a FONT SIZE?
│   └── ✅ Use FONT_SIZE.* or RFValue()  →  Never use raw numbers for fonts
│
├── Is it HORIZONTAL? (width, paddingLeft, paddingRight, marginLeft, marginRight, left, right)
│   └── ✅ Use horizontalScale()
│
├── Is it VERTICAL? (height, paddingTop, paddingBottom, marginTop, marginBottom, top, bottom)
│   └── ✅ Use verticalScale()
│
├── Is it BOTH / AMBIGUOUS? (borderRadius, gap, icon size, general padding, elevation)
│   └── ✅ Use moderateScale()  →  Safe default when unsure
│
└── Is it a FLEX ratio or PERCENTAGE? ('100%', flex: 1)
    └── ✅ Use as-is  →  These are already responsive
```

---

### `horizontalScale(size)` — Horizontal Dimensions

**Use when:** The value maps to the **X axis** — widths, left/right padding, left/right margins, and `left`/`right` absolute positions.

```typescript
import { horizontalScale } from '@/constants/fonts';

const styles = StyleSheet.create({
  container: {
    paddingHorizontal: horizontalScale(16),  // ✅ Left/right padding
    marginLeft:        horizontalScale(12),  // ✅ Left margin
  },
  card: {
    width: horizontalScale(160),             // ✅ Fixed-width card
    // ⚠️ Prefer width: '48%' for fluid cards — use horizontalScale
    //    only when the card MUST be a specific point size
  },
  divider: {
    width: horizontalScale(48),              // ✅ Short horizontal rule
    height: 1,
  },
});
```

**Do NOT use for:**
- Font sizes → use `FONT_SIZE.*`
- Heights → use `verticalScale()`
- Symmetric padding (top+bottom+left+right equally) → use `moderateScale()`

---

### `verticalScale(size)` — Vertical Dimensions

**Use when:** The value maps to the **Y axis** — heights, top/bottom padding, top/bottom margins, and `top`/`bottom` absolute positions.

```typescript
import { verticalScale } from '@/constants/fonts';

const styles = StyleSheet.create({
  header: {
    height:         verticalScale(56),   // ✅ Header bar height
    paddingTop:     verticalScale(8),    // ✅ Top padding
    paddingBottom:  verticalScale(8),    // ✅ Bottom padding
  },
  listItem: {
    marginBottom:   verticalScale(12),   // ✅ Vertical spacing between rows
    minHeight:      verticalScale(48),   // ✅ Touch target minimum height
  },
  floatingButton: {
    bottom:         verticalScale(32),   // ✅ Absolute position from bottom
  },
});
```

**Why it matters:** A compact phone (667pt tall) vs a Pro Max (926pt tall) differ by ~39% in height. Without `verticalScale`, a `marginBottom: 120` that looks great on Pro Max will crowd content on an SE.

---

### `moderateScale(size, factor?)` — The Safe Default

**Use when:** The value is ambiguous (applies equally in both axes), OR you want to **soften** the scaling effect so it doesn't grow too aggressively on large screens.

```typescript
import { moderateScale } from '@/constants/fonts';

const styles = StyleSheet.create({
  card: {
    borderRadius:  moderateScale(14),     // ✅ Equal in both axes
    padding:       moderateScale(16),     // ✅ All-sides symmetric padding
    gap:           moderateScale(10),     // ✅ Gap between flex children
  },
  icon: {
    width:         moderateScale(24),     // ✅ Square icon
    height:        moderateScale(24),     // ✅ Square icon (same value)
  },
  avatar: {
    width:         moderateScale(48),
    height:        moderateScale(48),
    borderRadius:  moderateScale(24),     // ✅ Circle = half of size
  },
  badge: {
    paddingHorizontal: moderateScale(8),
    paddingVertical:   moderateScale(4),
    borderRadius:      moderateScale(6),
  },
});
```

#### The `factor` Parameter

The optional `factor` argument controls how aggressively scaling applies:

| Factor | Behavior | Best For |
|--------|---------|----------|
| `0` | No scaling — returns original `size` | Effectively disabled |
| `0.25` | Very subtle scaling | Borders, thin lines |
| `0.5` *(default)* | Moderate — halfway between fixed and fully scaled | General use |
| `0.75` | More aggressive | Cards, containers |
| `1.0` | Equivalent to `horizontalScale` | Full horizontal scaling |

```typescript
moderateScale(16)       // → 16 + (horizontalScale(16) - 16) × 0.5
moderateScale(16, 0.25) // → 16 + (horizontalScale(16) - 16) × 0.25  (subtler)
moderateScale(16, 0.75) // → 16 + (horizontalScale(16) - 16) × 0.75  (stronger)
```

---

### At-a-Glance Reference Table

| What you're styling | Function to use | Example |
|---------------------|----------------|---------|
| `width` (fixed) | `horizontalScale` | `width: horizontalScale(200)` |
| `height` (fixed) | `verticalScale` | `height: verticalScale(56)` |
| `paddingHorizontal` / `paddingLeft` / `paddingRight` | `horizontalScale` | `paddingHorizontal: horizontalScale(16)` |
| `paddingVertical` / `paddingTop` / `paddingBottom` | `verticalScale` | `paddingVertical: verticalScale(12)` |
| `padding` (all sides equal) | `moderateScale` | `padding: moderateScale(16)` |
| `marginHorizontal` / left / right | `horizontalScale` | `marginHorizontal: horizontalScale(8)` |
| `marginVertical` / top / bottom | `verticalScale` | `marginBottom: verticalScale(20)` |
| `borderRadius` | `moderateScale` | `borderRadius: moderateScale(12)` |
| `gap` | `moderateScale` | `gap: moderateScale(10)` |
| Icon / avatar size (square) | `moderateScale` | `width: moderateScale(32)` |
| `fontSize` | `FONT_SIZE.*` | `fontSize: FONT_SIZE.medium` |
| `flex` ratio | *(none — already responsive)* | `flex: 1` |
| `width: '%'` | *(none — already responsive)* | `width: '90%'` |

---

## 3. Typography: FONT_SIZE Constants

### The Scale

| Token | `RFValue` | Approx. on 812pt screen | Use Case |
|-------|----------|------------------------|---------|
| `FONT_SIZE.xs` | `RFValue(10)` | ~10pt | Micro labels, badges, timestamps |
| `FONT_SIZE.small` | `RFValue(12)` | ~12pt | Captions, helper text, hints |
| `FONT_SIZE.regular` | `RFValue(14)` | ~14pt | Body text, list items, descriptions |
| `FONT_SIZE.medium` | `RFValue(16)` | ~16pt | Emphasized body, input text |
| `FONT_SIZE.large` | `RFValue(18)` | ~18pt | Subheadings, card titles |
| `FONT_SIZE.xl` | `RFValue(22)` | ~22pt | Screen titles, primary headings |
| `FONT_SIZE.xxl` | `RFValue(28)` | ~28pt | Hero headings, onboarding titles |
| `FONT_SIZE.xxxl` | `RFValue(34)` | ~34pt | Display type, splash screens |

### Rules for Typography

**Rule 1 — Never hardcode a fontSize number.**

```typescript
// ❌ Never do this
fontSize: 16

// ✅ Always do this
fontSize: FONT_SIZE.medium
```

**Rule 2 — Use `TEXT_STYLE` presets for common patterns.**

```typescript
import { TEXT_STYLE } from '@/constants/fonts';

const styles = StyleSheet.create({
  heading:  TEXT_STYLE.title,    // xl + bold
  subhead:  TEXT_STYLE.subtitle, // large + medium weight
  body:     TEXT_STYLE.body,     // regular + regular weight
  caption:  TEXT_STYLE.caption,  // small + regular weight
});
```

**Rule 3 — Compose `TEXT_STYLE` with spread for overrides.**

```typescript
const styles = StyleSheet.create({
  errorText: {
    ...TEXT_STYLE.caption,      // Inherit base caption style
    color: '#E53935',           // Override with error color
  },
  boldBody: {
    ...TEXT_STYLE.body,
    fontWeight: FONT_WEIGHT.bold,
  },
});
```

**Rule 4 — Use `LINE_HEIGHT` multipliers, not fixed lineHeight values.**

```typescript
import { FONT_SIZE, LINE_HEIGHT } from '@/constants/fonts';

const styles = StyleSheet.create({
  paragraph: {
    fontSize:   FONT_SIZE.regular,
    lineHeight: FONT_SIZE.regular * LINE_HEIGHT.normal,  // 14 × 1.5 = 21
  },
  compactLabel: {
    fontSize:   FONT_SIZE.small,
    lineHeight: FONT_SIZE.small * LINE_HEIGHT.tight,     // 12 × 1.2 = 14.4
  },
});
```

**Rule 5 — Cap system font scaling on UI-critical text.**

```tsx
// Prevent accessibility large-text setting from breaking card layouts
<Text
  style={styles.cardTitle}
  maxFontSizeMultiplier={1.3}
  numberOfLines={2}
>
  {title}
</Text>
```

---

## 4. Before vs. After: Responsive Refactors

### Example A — Profile Card

```tsx
// ❌ BEFORE — Fixed, breaks on small phones and tablets

import { View, Text, Image, StyleSheet } from 'react-native';

export function ProfileCard() {
  return (
    <View style={styles.card}>
      <Image source={{ uri: avatarUrl }} style={styles.avatar} />
      <View style={styles.info}>
        <Text style={styles.name}>Jane Doe</Text>
        <Text style={styles.role}>Senior Engineer</Text>
      </View>
    </View>
  );
}

const styles = StyleSheet.create({
  card: {
    flexDirection: 'row',
    alignItems: 'center',
    padding: 16,           // ❌ Fixed — crowds on SE, too tight on tablet
    borderRadius: 12,      // ❌ Fixed
    marginBottom: 20,      // ❌ Fixed
    backgroundColor: '#fff',
  },
  avatar: {
    width: 56,             // ❌ Fixed — tiny on tablet, large on SE
    height: 56,            // ❌ Fixed
    borderRadius: 28,      // ❌ Fixed
    marginRight: 14,       // ❌ Fixed
  },
  name: {
    fontSize: 18,          // ❌ Hardcoded — never do this
    fontWeight: '700',
  },
  role: {
    fontSize: 13,          // ❌ Hardcoded
    color: '#666',
  },
});
```

```tsx
// ✅ AFTER — Fully responsive

import { View, Text, StyleSheet } from 'react-native';
import {
  horizontalScale, verticalScale, moderateScale,
  FONT_SIZE, FONT_WEIGHT, TEXT_STYLE,
} from '@/constants/fonts';

export function ProfileCard() {
  return (
    <View style={styles.card}>
      <View style={styles.avatar} />
      <View style={styles.info}>
        <Text style={styles.name} maxFontSizeMultiplier={1.2}>Jane Doe</Text>
        <Text style={styles.role} maxFontSizeMultiplier={1.2}>Senior Engineer</Text>
      </View>
    </View>
  );
}

const styles = StyleSheet.create({
  card: {
    flexDirection: 'row',
    alignItems: 'center',
    padding:         moderateScale(16),      // ✅ All-sides padding
    borderRadius:    moderateScale(12),      // ✅ Corner radius
    marginBottom:    verticalScale(20),      // ✅ Vertical gap between cards
    backgroundColor: '#fff',
  },
  avatar: {
    width:           moderateScale(56),      // ✅ Square — same scale both axes
    height:          moderateScale(56),
    borderRadius:    moderateScale(28),      // ✅ Stays circular
    marginRight:     horizontalScale(14),    // ✅ Horizontal gap
    backgroundColor: '#ccc',
  },
  name: {
    ...TEXT_STYLE.subtitle,                  // ✅ large + medium weight preset
  },
  role: {
    fontSize:   FONT_SIZE.small,             // ✅ Responsive token
    fontWeight: FONT_WEIGHT.regular,
    color:      '#666',
    marginTop:  verticalScale(2),            // ✅ Tiny vertical nudge
  },
});
```

---

### Example B — Action Button

```tsx
// ❌ BEFORE

const styles = StyleSheet.create({
  button: {
    width: 320,            // ❌ Too wide on SE, too narrow on tablet
    height: 52,            // ❌ Fixed height
    borderRadius: 10,      // ❌ Fixed
    paddingHorizontal: 24, // ❌ Fixed
    alignItems: 'center',
    justifyContent: 'center',
    backgroundColor: '#4F8EF7',
  },
  label: {
    fontSize: 16,          // ❌ Hardcoded
    fontWeight: '600',
    color: '#fff',
  },
});
```

```tsx
// ✅ AFTER

const styles = StyleSheet.create({
  button: {
    width:             '88%',              // ✅ Fluid — works on all widths
    height:            verticalScale(52),  // ✅ Scales with screen height
    borderRadius:      moderateScale(10),  // ✅ Proportional corners
    paddingHorizontal: horizontalScale(24),// ✅ Horizontal padding
    alignItems:        'center',
    justifyContent:    'center',
    alignSelf:         'center',           // ✅ Centers the 88% width
    backgroundColor:   '#4F8EF7',
  },
  label: {
    fontSize:   FONT_SIZE.medium,          // ✅ Token
    fontWeight: FONT_WEIGHT.semibold,      // ✅ Typed constant
    color:      '#fff',
  },
});
```

---

### Example C — Section Header with Icon

```tsx
// ❌ BEFORE
const styles = StyleSheet.create({
  row:   { flexDirection: 'row', alignItems: 'center', marginBottom: 12, paddingHorizontal: 16 },
  icon:  { width: 22, height: 22, marginRight: 8 },
  title: { fontSize: 20, fontWeight: '700' },
});

// ✅ AFTER
const styles = StyleSheet.create({
  row: {
    flexDirection:    'row',
    alignItems:       'center',
    marginBottom:     verticalScale(12),
    paddingHorizontal: horizontalScale(16),
  },
  icon: {
    width:       moderateScale(22),     // ✅ Square icon
    height:      moderateScale(22),
    marginRight: horizontalScale(8),    // ✅ Horizontal gap
  },
  title: {
    ...TEXT_STYLE.title,                // ✅ Preset: xl + bold
  },
});
```

---

## 5. Tablet Breakpoints with `useWindowDimensions`

### Why `useWindowDimensions` (not `Dimensions.get`)

`Dimensions.get('window')` is evaluated **once at module load time** and does not update when the device rotates or when a tablet goes split-screen. `useWindowDimensions` is a React hook that re-renders your component automatically whenever the screen dimensions change.

```typescript
// ❌ Static — won't update on rotation or split-screen
const { width } = Dimensions.get('window');

// ✅ Reactive — re-renders on any dimension change
const { width, height } = useWindowDimensions();
```

---

### Defining Breakpoints

```typescript
// src/constants/breakpoints.ts

export const BREAKPOINTS = {
  smallPhone: 360,   // iPhone SE, budget Androids
  phone:      430,   // Standard phones (iPhone 14, Pixel 7)
  phablet:    600,   // Large phones / small tablets
  tablet:     768,   // iPad Mini, Android tablets
  largeTablet: 1024, // iPad Pro 12.9", large Android tablets
} as const;

// Convenience hook
import { useWindowDimensions } from 'react-native';
import { BREAKPOINTS } from './breakpoints';

export function useBreakpoints() {
  const { width, height } = useWindowDimensions();
  return {
    width,
    height,
    isSmallPhone:   width < BREAKPOINTS.smallPhone,
    isPhone:        width < BREAKPOINTS.phablet,
    isTablet:       width >= BREAKPOINTS.tablet,
    isLargeTablet:  width >= BREAKPOINTS.largeTablet,
    isLandscape:    width > height,
  };
}
```

---

### Responsive Grid Layout

```tsx
import React from 'react';
import { View, FlatList, StyleSheet } from 'react-native';
import { useBreakpoints } from '@/constants/breakpoints';
import { horizontalScale, verticalScale, moderateScale } from '@/constants/fonts';

const ITEMS = Array.from({ length: 20 }, (_, i) => ({ id: String(i) }));

export default function ProductGrid() {
  const { width, isTablet, isLargeTablet } = useBreakpoints();

  // Column count based on screen width
  const columns      = isLargeTablet ? 4 : isTablet ? 3 : 2;
  const hPadding     = horizontalScale(16);
  const gap          = moderateScale(12);
  const cardWidth    = (width - hPadding * 2 - gap * (columns - 1)) / columns;

  return (
    <FlatList
      data={ITEMS}
      numColumns={columns}
      key={columns}                                // Re-key forces re-render on column change
      keyExtractor={(item) => item.id}
      contentContainerStyle={[
        styles.list,
        { paddingHorizontal: hPadding },
      ]}
      columnWrapperStyle={columns > 1 ? { gap } : undefined}
      renderItem={() => (
        <View style={[styles.card, { width: cardWidth }]} />
      )}
    />
  );
}

const styles = StyleSheet.create({
  list: {
    paddingTop: verticalScale(16),
  },
  card: {
    aspectRatio:     3 / 4,
    borderRadius:    moderateScale(12),
    marginBottom:    verticalScale(12),
    backgroundColor: '#e8eaf6',
  },
});
```

---

### Conditional Layout Direction

```tsx
import { View, StyleSheet } from 'react-native';
import { useBreakpoints } from '@/constants/breakpoints';

export default function DetailLayout({ sidebar, content }) {
  const { isTablet } = useBreakpoints();

  return (
    <View style={[
      styles.container,
      isTablet && styles.containerTablet,   // Side-by-side on tablet
    ]}>
      <View style={[
        styles.sidebar,
        isTablet && styles.sidebarTablet,
      ]}>
        {sidebar}
      </View>
      <View style={styles.content}>
        {content}
      </View>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    flexDirection: 'column',         // Phone: stacked
  },
  containerTablet: {
    flexDirection: 'row',            // Tablet: side-by-side
  },
  sidebar: {
    width: '100%',                   // Phone: full width
  },
  sidebarTablet: {
    width: 280,                      // Tablet: fixed sidebar
  },
  content: {
    flex: 1,
  },
});
```

---

### Font Size Adjustments for Tablets

`RFValue` already scales with screen height, but for tablets you may want to apply an additional cap or step up:

```tsx
import { useBreakpoints } from '@/constants/breakpoints';
import { FONT_SIZE } from '@/constants/fonts';

export function useAdaptiveFontSize() {
  const { isTablet, isLargeTablet } = useBreakpoints();

  if (isLargeTablet) {
    return {
      heading: FONT_SIZE.xxxl,
      body:    FONT_SIZE.large,
      caption: FONT_SIZE.regular,
    };
  }

  if (isTablet) {
    return {
      heading: FONT_SIZE.xxl,
      body:    FONT_SIZE.medium,
      caption: FONT_SIZE.small,
    };
  }

  return {
    heading: FONT_SIZE.xl,
    body:    FONT_SIZE.regular,
    caption: FONT_SIZE.xs,
  };
}
```

---

## 6. Summary Checklist

Run through this checklist before marking any screen or component as **done**.

---

### ✅ Step 1 — Zero Hardcoded Layout Numbers

Search the file for raw number values in `StyleSheet`:

```
width: 300     ❌  →  width: horizontalScale(300) or width: '85%'
height: 200    ❌  →  height: verticalScale(200)
padding: 16    ❌  →  padding: moderateScale(16)
marginTop: 24  ❌  →  marginTop: verticalScale(24)
borderRadius:8 ❌  →  borderRadius: moderateScale(8)
```

**Pass criterion:** No bare numbers in any dimension-related style property.

---

### ✅ Step 2 — Zero Hardcoded Font Sizes

```
fontSize: 14   ❌  →  fontSize: FONT_SIZE.regular
fontSize: 20   ❌  →  fontSize: FONT_SIZE.xl
fontWeight:'700' ⚠️ →  fontWeight: FONT_WEIGHT.bold
```

**Pass criterion:** Every `fontSize` value is a `FONT_SIZE.*` token. Every `fontWeight` is a `FONT_WEIGHT.*` constant.

---

### ✅ Step 3 — Correct Scale Function for Each Axis

Review each scaled value and confirm the right function was chosen:

| Check | Expected |
|-------|---------|
| `paddingHorizontal`, `marginLeft`, `marginRight`, `left`, `right`, fixed `width` | `horizontalScale()` |
| `paddingVertical`, `marginTop`, `marginBottom`, `top`, `bottom`, fixed `height` | `verticalScale()` |
| `borderRadius`, `gap`, icon sizes, equal `padding` | `moderateScale()` |

**Pass criterion:** No `verticalScale` used for horizontal properties and vice versa.

---

### ✅ Step 4 — Orientation & Tablet Tested

- [ ] Rotate to **landscape** — does layout reflow correctly?
- [ ] Test on a **tablet** (width ≥ 768) — do columns/panels adapt?
- [ ] Is `useWindowDimensions` used instead of `Dimensions.get` for any reactive layout?
- [ ] Are `FlatList` components re-keyed when `numColumns` changes?

**Pass criterion:** Screen renders correctly in landscape and on a 768pt-wide device.

---

### ✅ Step 5 — Safe Area & Accessibility Verified

- [ ] Screen is wrapped in `<SafeAreaView>` from `react-native-safe-area-context`
- [ ] Bottom content (buttons, tabs) respects `insets.bottom`
- [ ] All `<Text>` nodes that may overflow have `numberOfLines` and/or `maxFontSizeMultiplier`
- [ ] Minimum touch targets are `moderateScale(44)` × `moderateScale(44)` (Apple/Google HIG)
- [ ] Tested with OS accessibility **Large Text** setting enabled

**Pass criterion:** No content clipped by notch/home indicator. No layout breakage with large text.

---

### Quick Sign-Off Table

Copy this block into your PR description:

```
## Responsive UI Checklist

- [ ] Step 1: No hardcoded layout numbers
- [ ] Step 2: No hardcoded font sizes
- [ ] Step 3: Correct scale function per axis
- [ ] Step 4: Orientation + tablet tested
- [ ] Step 5: Safe area + accessibility verified
```

---

*Last updated: auto-generated from `src/constants/fonts.ts` — keep in sync when adding new tokens.*
