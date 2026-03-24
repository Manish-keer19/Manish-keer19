# 📱 Responsive UI in React Native with Expo — Complete Guide

> A practical, beginner-to-intermediate guide to building UIs that look great on every screen.

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [Core Concepts](#2-core-concepts)
3. [Screen Dimensions Handling](#3-screen-dimensions-handling)
4. [Scaling Techniques](#4-scaling-techniques)
5. [Responsive Typography System](#5-responsive-typography-system)
6. [Layout Techniques](#6-layout-techniques)
7. [Safe Area Handling](#7-safe-area-handling)
8. [Scroll Handling](#8-scroll-handling)
9. [Image Responsiveness](#9-image-responsiveness)
10. [Testing Strategy](#10-testing-strategy)
11. [Performance Considerations](#11-performance-considerations)
12. [Common Mistakes](#12-common-mistakes)
13. [Best Practices Checklist](#13-best-practices-checklist)
14. [Complete Example](#14-complete-example)

---

## 1. Introduction

### What is Responsive UI in React Native?

A **responsive UI** adapts its layout, typography, and spacing to look and function correctly across a wide variety of screen sizes and resolutions — from small phones (320px wide) to large tablets (1024px+), and across both portrait and landscape orientations.

In React Native, you're building for a **native environment**, not a browser. That means CSS media queries don't exist — instead, you use Flexbox, the `Dimensions` API, hooks, and scaling utilities to achieve the same goal.

### Why Does Responsiveness Matter?

| Challenge | Details |
|-----------|---------|
| **Screen Sizes** | Devices range from 4" phones to 13" tablets |
| **Pixel Density (DPI)** | A "1px" on a Retina screen ≠ 1px on a standard screen |
| **Orientation** | Portrait and landscape require different layouts |
| **Platform Differences** | iOS and Android handle spacing/fonts slightly differently |
| **Accessibility** | Users may increase system font size |

Building a responsive UI from the start saves hours of debugging and makes your app feel professional and polished on every device.

---

## 2. Core Concepts

### 2.1 Flexbox

React Native uses **Flexbox** by default for layout. Unlike CSS, the default `flexDirection` in React Native is `column` (not `row`).

```jsx
import { View, Text, StyleSheet } from 'react-native';

export default function FlexExample() {
  return (
    <View style={styles.container}>
      <View style={styles.box}><Text>Box 1</Text></View>
      <View style={styles.box}><Text>Box 2</Text></View>
      <View style={styles.box}><Text>Box 3</Text></View>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,                      // Takes up all available space
    flexDirection: 'row',         // Arrange children horizontally
    justifyContent: 'space-between', // Space out along main axis
    alignItems: 'center',         // Center along cross axis
    padding: 16,
  },
  box: {
    flex: 1,                      // Each box takes equal share
    margin: 4,
    backgroundColor: '#4F8EF7',
    padding: 12,
    borderRadius: 8,
    alignItems: 'center',
  },
});
```

#### Key Flexbox Properties

| Property | Options | Description |
|----------|---------|-------------|
| `flex` | number | How much space the child takes relative to siblings |
| `flexDirection` | `row`, `column`, `row-reverse`, `column-reverse` | Direction of children |
| `justifyContent` | `flex-start`, `flex-end`, `center`, `space-between`, `space-around`, `space-evenly` | Alignment along **main** axis |
| `alignItems` | `flex-start`, `flex-end`, `center`, `stretch`, `baseline` | Alignment along **cross** axis |
| `flexWrap` | `wrap`, `nowrap` | Whether children wrap to next line |

---

### 2.2 Percentage-Based Dimensions

Instead of fixed pixel values, use percentages so your UI scales naturally:

```jsx
import { View, StyleSheet } from 'react-native';

export default function PercentageExample() {
  return (
    <View style={styles.container}>
      <View style={styles.card} />
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    alignItems: 'center',
    justifyContent: 'center',
  },
  card: {
    width: '90%',    // 90% of parent width — adapts to screen
    height: '30%',   // 30% of parent height
    backgroundColor: '#4F8EF7',
    borderRadius: 12,
  },
});
```

> ✅ Percentages are relative to the **parent container**, not the screen. Make sure the parent has a defined size (e.g., `flex: 1`).

---

### 2.3 Avoiding Fixed Sizes

**Fixed pixel values are the #1 enemy of responsive design.**

```jsx
// ❌ BAD — Looks fine on iPhone 13, broken on small/large devices
const styles = StyleSheet.create({
  button: {
    width: 300,
    height: 50,
  },
});

// ✅ GOOD — Adapts to screen width
import { Dimensions } from 'react-native';
const { width } = Dimensions.get('window');

const styles = StyleSheet.create({
  button: {
    width: width * 0.85, // 85% of screen width
    height: 50,          // Height is okay as fixed if intentional
  },
});
```

---

### 2.4 Aspect Ratio

Use `aspectRatio` to maintain proportions without specifying both dimensions:

```jsx
import { View, StyleSheet } from 'react-native';

const styles = StyleSheet.create({
  videoContainer: {
    width: '100%',
    aspectRatio: 16 / 9,   // Maintains 16:9 ratio regardless of width
    backgroundColor: '#000',
  },
  avatar: {
    width: 80,
    aspectRatio: 1,        // Makes it a perfect square
    borderRadius: 40,
    backgroundColor: '#ccc',
  },
});
```

---

## 3. Screen Dimensions Handling

### 3.1 Dimensions API

The `Dimensions` API gives you the screen width and height at a single point in time.

```jsx
import { Dimensions, View, Text, StyleSheet } from 'react-native';

const { width, height } = Dimensions.get('window');
// Use 'screen' to include status bar height on Android

console.log(`Width: ${width}, Height: ${height}`);

const styles = StyleSheet.create({
  container: {
    width: width,
    paddingHorizontal: width * 0.05,  // 5% padding on each side
  },
});
```

> ⚠️ **Limitation:** `Dimensions.get()` is called once at load time. It does **not** automatically update on orientation change.

---

### 3.2 `useWindowDimensions` Hook

This hook **re-renders automatically** when the screen size changes (e.g., rotation):

```jsx
import { useWindowDimensions, View, Text, StyleSheet } from 'react-native';

export default function ResponsiveCard() {
  const { width, height } = useWindowDimensions();

  const isLandscape = width > height;
  const cardWidth = width > 600 ? width * 0.5 : width * 0.9;

  return (
    <View style={[styles.card, { width: cardWidth }]}>
      <Text>Mode: {isLandscape ? 'Landscape' : 'Portrait'}</Text>
      <Text>Screen: {Math.round(width)} × {Math.round(height)}</Text>
    </View>
  );
}

const styles = StyleSheet.create({
  card: {
    alignSelf: 'center',
    padding: 16,
    backgroundColor: '#f0f4ff',
    borderRadius: 12,
  },
});
```

---

### 3.3 Handling Orientation Changes

Combine `useWindowDimensions` with conditional layouts:

```jsx
import { useWindowDimensions, View, Text } from 'react-native';

export default function OrientationLayout() {
  const { width, height } = useWindowDimensions();
  const isLandscape = width > height;

  return (
    <View style={{
      flex: 1,
      flexDirection: isLandscape ? 'row' : 'column',
      padding: 16,
    }}>
      <View style={{ flex: 1, backgroundColor: '#4F8EF7', margin: 8, borderRadius: 8 }}>
        <Text style={{ color: 'white', padding: 16 }}>Panel 1</Text>
      </View>
      <View style={{ flex: 1, backgroundColor: '#F74F8E', margin: 8, borderRadius: 8 }}>
        <Text style={{ color: 'white', padding: 16 }}>Panel 2</Text>
      </View>
    </View>
  );
}
```

> ✅ To enable orientation support in Expo, set `"orientation": "default"` in your `app.json`.

---

## 4. Scaling Techniques

### 4.1 Manual Scaling

A simple and effective approach — calculate scale factors based on a base design width:

```jsx
import { Dimensions } from 'react-native';

const BASE_WIDTH = 375; // iPhone 13 Mini — common design base
const { width: SCREEN_WIDTH, height: SCREEN_HEIGHT } = Dimensions.get('window');

const widthScale = SCREEN_WIDTH / BASE_WIDTH;
const heightScale = SCREEN_HEIGHT / 812; // Base height (iPhone 13)

// Scales a size proportionally to screen width
export const scale = (size) => Math.round(size * widthScale);

// Scales a size proportionally to screen height
export const verticalScale = (size) => Math.round(size * heightScale);

// Moderate scaling — less aggressive, great for fonts and padding
export const moderateScale = (size, factor = 0.5) =>
  Math.round(size + (scale(size) - size) * factor);
```

**Usage:**

```jsx
import { scale, verticalScale, moderateScale } from './scaling';

const styles = StyleSheet.create({
  container: {
    paddingHorizontal: scale(16),
    paddingVertical: verticalScale(12),
  },
  title: {
    fontSize: moderateScale(20),
  },
});
```

---

### 4.2 `react-native-responsive-fontsize`

Install:

```bash
npx expo install react-native-responsive-fontsize
```

Usage:

```jsx
import { RFValue, RFPercentage } from 'react-native-responsive-fontsize';

const styles = StyleSheet.create({
  heading: {
    fontSize: RFValue(24),        // Scales based on a 680px reference height
  },
  subheading: {
    fontSize: RFValue(16),
  },
  label: {
    fontSize: RFPercentage(2.5), // 2.5% of screen height
  },
});
```

> `RFValue(size, standardScreenHeight?)` — default standard height is 680.

---

### 4.3 `react-native-size-matters`

Install:

```bash
npx expo install react-native-size-matters
```

Usage:

```jsx
import { scale, verticalScale, moderateScale, moderateVerticalScale } from 'react-native-size-matters';

const styles = StyleSheet.create({
  container: {
    padding: moderateScale(16),
    marginVertical: verticalScale(10),
  },
  box: {
    width: scale(120),
    height: verticalScale(60),
    borderRadius: moderateScale(8),
  },
  text: {
    fontSize: moderateScale(14),
  },
});
```

| Function | Best For |
|----------|----------|
| `scale(size)` | Horizontal dimensions (width, padding) |
| `verticalScale(size)` | Vertical dimensions (height, margin) |
| `moderateScale(size, factor?)` | Fonts and subtle scaling (default factor: 0.5) |
| `moderateVerticalScale` | Vertical + moderate (cards, containers) |

---

## 5. Responsive Typography System

### 5.1 Why Consistent Font Sizes Matter

Hardcoding font sizes in every component leads to inconsistency. On a tablet, `fontSize: 14` might be too small. On a phone with large system fonts, it might overflow. A centralized font system solves this.

### 5.2 Creating a Global Font System

**`constants/fonts.js`**

```js
import { RFValue } from 'react-native-responsive-fontsize';

export const FONT_SIZE = {
  xs:      RFValue(10),
  small:   RFValue(12),
  regular: RFValue(14),
  medium:  RFValue(16),
  large:   RFValue(20),
  xl:      RFValue(24),
  xxl:     RFValue(32),
};

export const FONT_WEIGHT = {
  regular: '400',
  medium:  '500',
  semibold: '600',
  bold:    '700',
};

export const LINE_HEIGHT = {
  tight:   1.2,
  normal:  1.5,
  loose:   1.8,
};
```

### 5.3 Usage in Components

```jsx
import { Text, StyleSheet } from 'react-native';
import { FONT_SIZE, FONT_WEIGHT } from '../constants/fonts';

export default function Heading({ children }) {
  return <Text style={styles.heading}>{children}</Text>;
}

const styles = StyleSheet.create({
  heading: {
    fontSize: FONT_SIZE.xl,
    fontWeight: FONT_WEIGHT.bold,
    color: '#1a1a2e',
  },
});
```

### 5.4 Supporting System Font Scaling

```jsx
import { Text } from 'react-native';

// Prevent system font from scaling beyond 1.3x
<Text maxFontSizeMultiplier={1.3} style={styles.body}>
  Your content here
</Text>
```

---

## 6. Layout Techniques

### 6.1 Using Flexbox Effectively

```jsx
// Full-width card with content on left, icon on right
const styles = StyleSheet.create({
  card: {
    flexDirection: 'row',
    alignItems: 'center',
    justifyContent: 'space-between',
    paddingHorizontal: scale(16),
    paddingVertical: verticalScale(12),
    backgroundColor: '#fff',
    borderRadius: moderateScale(12),
    marginBottom: verticalScale(8),
    shadowColor: '#000',
    shadowOpacity: 0.06,
    shadowRadius: 6,
    elevation: 3,
  },
  content: {
    flex: 1,     // Takes remaining space, leaving room for icon
    marginRight: scale(12),
  },
  icon: {
    width: scale(40),
    height: scale(40),
  },
});
```

---

### 6.2 Conditional Layouts — Mobile vs Tablet

```jsx
import { useWindowDimensions, View } from 'react-native';

export default function AdaptiveGrid() {
  const { width } = useWindowDimensions();

  const isTablet = width > 600;
  const columns = isTablet ? 3 : 2;
  const cardWidth = (width - 32 - (columns - 1) * 12) / columns;

  const items = [1, 2, 3, 4, 5, 6];

  return (
    <View style={{ flexDirection: 'row', flexWrap: 'wrap', padding: 16, gap: 12 }}>
      {items.map((item) => (
        <View
          key={item}
          style={{
            width: cardWidth,
            height: cardWidth * 0.75,
            backgroundColor: '#4F8EF7',
            borderRadius: 12,
          }}
        />
      ))}
    </View>
  );
}
```

---

### 6.3 Breakpoints Logic

Define reusable breakpoints for cleaner conditional logic:

```js
// constants/breakpoints.js
import { Dimensions } from 'react-native';

const { width } = Dimensions.get('window');

export const BREAKPOINTS = {
  sm: 360,
  md: 600,
  lg: 768,
  xl: 1024,
};

export const isSmallPhone  = width < BREAKPOINTS.sm;
export const isPhone       = width < BREAKPOINTS.md;
export const isTablet      = width >= BREAKPOINTS.md;
export const isLargeTablet = width >= BREAKPOINTS.lg;
```

**Usage:**

```jsx
import { isTablet, isSmallPhone } from '../constants/breakpoints';

const numColumns   = isTablet ? 3 : 2;
const headerHeight = isSmallPhone ? 50 : 70;
const fontSize     = isTablet ? 18 : 14;
```

> For dynamic breakpoints (orientation-aware), use `useWindowDimensions()` and compute inline.

---

## 7. Safe Area Handling

### 7.1 Installing `react-native-safe-area-context`

This is included in Expo by default. Wrap your app:

```jsx
// App.js
import { SafeAreaProvider } from 'react-native-safe-area-context';
import MainNavigator from './navigation/MainNavigator';

export default function App() {
  return (
    <SafeAreaProvider>
      <MainNavigator />
    </SafeAreaProvider>
  );
}
```

---

### 7.2 Using `SafeAreaView`

```jsx
import { SafeAreaView } from 'react-native-safe-area-context';
import { StyleSheet, Text } from 'react-native';

export default function HomeScreen() {
  return (
    <SafeAreaView style={styles.container} edges={['top', 'left', 'right']}>
      <Text style={styles.title}>Safe Content</Text>
    </SafeAreaView>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#fff',
  },
  title: {
    fontSize: 22,
    fontWeight: 'bold',
    padding: 16,
  },
});
```

### 7.3 Using `useSafeAreaInsets`

For finer control (e.g., adding bottom padding for home indicator):

```jsx
import { useSafeAreaInsets } from 'react-native-safe-area-context';
import { View, StyleSheet } from 'react-native';

export default function BottomNav() {
  const insets = useSafeAreaInsets();

  return (
    <View style={[styles.nav, { paddingBottom: insets.bottom || 16 }]}>
      {/* Nav items */}
    </View>
  );
}

const styles = StyleSheet.create({
  nav: {
    flexDirection: 'row',
    backgroundColor: '#fff',
    paddingTop: 12,
    borderTopWidth: 1,
    borderTopColor: '#eee',
  },
});
```

---

## 8. Scroll Handling

### 8.1 `ScrollView` vs `FlatList`

| Feature | `ScrollView` | `FlatList` |
|---------|-------------|------------|
| Renders all items | ✅ Yes (all at once) | ❌ No (lazy rendering) |
| Best for | Short, static content | Long or dynamic lists |
| Memory efficient | ❌ For long lists | ✅ Yes |
| Pull to refresh | ✅ | ✅ |
| Horizontal scroll | ✅ | ✅ |

**ScrollView — Short Forms/Content:**

```jsx
import { ScrollView, StyleSheet } from 'react-native';

export default function FormScreen() {
  return (
    <ScrollView
      style={styles.scroll}
      contentContainerStyle={styles.content}
      keyboardShouldPersistTaps="handled"
      showsVerticalScrollIndicator={false}
    >
      {/* Form fields, cards, etc. */}
    </ScrollView>
  );
}

const styles = StyleSheet.create({
  scroll: { flex: 1 },
  content: {
    paddingHorizontal: 16,
    paddingBottom: 40, // Prevent content hiding behind bottom bar
  },
});
```

**FlatList — Dynamic Long Lists:**

```jsx
import { FlatList, View, Text, StyleSheet, useWindowDimensions } from 'react-native';

const DATA = Array.from({ length: 50 }, (_, i) => ({ id: String(i), title: `Item ${i + 1}` }));

export default function ListScreen() {
  const { width } = useWindowDimensions();
  const numColumns = width > 600 ? 3 : 2;

  return (
    <FlatList
      data={DATA}
      keyExtractor={(item) => item.id}
      numColumns={numColumns}
      key={numColumns} // ← Force re-render when columns change
      contentContainerStyle={styles.list}
      renderItem={({ item }) => (
        <View style={[styles.card, { width: (width - 48) / numColumns }]}>
          <Text>{item.title}</Text>
        </View>
      )}
    />
  );
}

const styles = StyleSheet.create({
  list: { padding: 16 },
  card: {
    backgroundColor: '#f0f4ff',
    margin: 4,
    padding: 16,
    borderRadius: 10,
  },
});
```

> ⚠️ Always provide `key={numColumns}` when changing `numColumns` on `FlatList` — it forces a full re-render to avoid layout bugs.

---

## 9. Image Responsiveness

### 9.1 Using `resizeMode`

```jsx
import { Image, StyleSheet, View } from 'react-native';

export default function ImageExample() {
  return (
    <View style={styles.container}>
      <Image
        source={{ uri: 'https://example.com/photo.jpg' }}
        style={styles.image}
        resizeMode="cover"   // cover | contain | stretch | center | repeat
      />
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    width: '100%',
    aspectRatio: 16 / 9,  // ← Key: let height adapt to width
  },
  image: {
    width: '100%',
    height: '100%',
  },
});
```

| `resizeMode` | Behavior |
|-------------|---------|
| `cover` | Fills box, may crop edges |
| `contain` | Fits inside box, may letterbox |
| `stretch` | Stretches to fill, may distort |
| `center` | Centers without resizing |

---

### 9.2 Avoiding Fixed Image Sizes

```jsx
// ❌ BAD
<Image source={img} style={{ width: 350, height: 200 }} />

// ✅ GOOD — Percentage + aspectRatio
<View style={{ width: '100%', aspectRatio: 16/9 }}>
  <Image source={img} style={{ width: '100%', height: '100%' }} resizeMode="cover" />
</View>
```

---

### 9.3 Avatar / Profile Images

```jsx
const styles = StyleSheet.create({
  avatar: {
    width: moderateScale(56),
    aspectRatio: 1,              // Always square
    borderRadius: moderateScale(28),
    backgroundColor: '#ddd',
  },
});
```

---

## 10. Testing Strategy

### 10.1 Android Emulator — Multiple Devices

Test on a range of virtual devices in Android Studio:

- **Small phone:** Pixel 4a (5.8", 1080×2340)
- **Standard phone:** Pixel 7 (6.3", 1080×2400)
- **Large phone:** Pixel 7 Pro (6.7")
- **Tablet:** Pixel Tablet (10.95")

Change orientation with `Ctrl+F11` (Windows/Linux) or `Fn+Ctrl+F11` (Mac).

---

### 10.2 iOS Simulator (macOS only)

- Use Xcode Simulator: `Hardware > Device` to switch devices
- Test iPhone SE (small), iPhone 15 (standard), iPad Pro 12.9" (tablet)
- Toggle orientation: `Device > Rotate Left/Right`

---

### 10.3 Expo Go — Real Device Testing

```bash
npx expo start
```

- Scan the QR code with Expo Go app
- Test on real Android and iOS devices simultaneously
- Rotate your device to check orientation handling

---

### 10.4 Responsive Testing Checklist

```
□ Tested on small phone (320–375px width)
□ Tested on standard phone (390–430px width)
□ Tested on large phone/phablet (>430px)
□ Tested on tablet (600px+)
□ Tested in portrait orientation
□ Tested in landscape orientation
□ Tested with large system font size (Accessibility settings)
□ Tested on both iOS and Android
```

---

## 11. Performance Considerations

### 11.1 Avoid Unnecessary Re-renders

```jsx
import React, { memo, useCallback } from 'react';

// Memoize list items to prevent re-renders when parent updates
const ListItem = memo(({ item, onPress }) => (
  <TouchableOpacity onPress={() => onPress(item.id)} style={styles.item}>
    <Text>{item.title}</Text>
  </TouchableOpacity>
));

// Memoize callbacks passed to memoized components
export default function List({ data }) {
  const handlePress = useCallback((id) => {
    console.log('Pressed:', id);
  }, []); // ← No dependencies = stable reference

  return (
    <FlatList
      data={data}
      keyExtractor={(item) => item.id}
      renderItem={({ item }) => <ListItem item={item} onPress={handlePress} />}
    />
  );
}
```

---

### 11.2 Optimize Images

```jsx
// ✅ Use Expo Image for better performance and caching
import { Image } from 'expo-image';

<Image
  source={{ uri: 'https://example.com/large-photo.jpg' }}
  style={styles.image}
  contentFit="cover"
  placeholder={blurhash}        // Show placeholder while loading
  cachePolicy="memory-disk"     // Cache in memory and on disk
  recyclingKey={item.id}        // Reuse image views in FlatList
/>
```

Install:

```bash
npx expo install expo-image
```

---

### 11.3 Avoid Heavy Inline Styles

```jsx
// ❌ BAD — New object created every render
<View style={{ flex: 1, backgroundColor: '#fff', padding: width * 0.05 }}>

// ✅ GOOD — Compute once outside component
const PADDING = width * 0.05;
const styles = StyleSheet.create({
  container: { flex: 1, backgroundColor: '#fff', padding: PADDING },
});
```

---

### 11.4 Use `windowSize` in FlatList

```jsx
<FlatList
  data={data}
  renderItem={renderItem}
  windowSize={5}           // Render 5 viewports worth of items
  initialNumToRender={10}  // Render 10 items on first paint
  maxToRenderPerBatch={5}  // Render 5 items per batch
  removeClippedSubviews    // Remove off-screen items (Android)
/>
```

---

## 12. Common Mistakes

### ❌ Mistake 1: Fixed Width and Height

```jsx
// ❌ Breaks on different screen sizes
style={{ width: 300, height: 500 }}

// ✅ Use flex, percentages, or scaled values
style={{ width: '85%', flex: 1 }}
```

---

### ❌ Mistake 2: Misusing Absolute Positioning

```jsx
// ❌ Hardcoded position — breaks on different screen sizes
style={{ position: 'absolute', bottom: 30, right: 20 }}

// ✅ Account for safe area insets
const insets = useSafeAreaInsets();
style={{ position: 'absolute', bottom: insets.bottom + 16, right: 16 }}
```

---

### ❌ Mistake 3: Ignoring Small Devices

Small phones (iPhone SE, budget Androids) have widths as small as 320px. Always test the minimum breakpoint:

```jsx
// ❌ Text overflow on SE
<Text style={{ fontSize: 22, padding: 24 }}>Very Long Card Title Here</Text>

// ✅ Scale font and padding
<Text style={{ fontSize: moderateScale(18), padding: moderateScale(16) }}>
  Very Long Card Title Here
</Text>
```

---

### ❌ Mistake 4: Using `Dimensions` Without Rotation Support

```jsx
// ❌ Won't update on rotation
const { width } = Dimensions.get('window');

// ✅ Updates automatically on rotation
const { width } = useWindowDimensions();
```

---

### ❌ Mistake 5: Missing `flex: 1` on Containers

```jsx
// ❌ Container collapses — child won't render
<View style={{ backgroundColor: '#fff' }}>
  <Text>Hello</Text>
</View>

// ✅ Container expands to fill available space
<View style={{ flex: 1, backgroundColor: '#fff' }}>
  <Text>Hello</Text>
</View>
```

---

### ❌ Mistake 6: Not Testing on Real Devices

Emulators are good, but real devices reveal:
- Touch target issues
- Font rendering differences
- Performance bottlenecks
- Actual safe area behavior

---

## 13. Best Practices Checklist

```
LAYOUT
□ Use flex instead of fixed width/height wherever possible
□ Use percentages for widths on full-width elements
□ Use aspectRatio to maintain proportions
□ Wrap root screen in SafeAreaView or use useSafeAreaInsets

TYPOGRAPHY
□ Use RFValue() for all font sizes
□ Define a global FONT_SIZE constant file
□ Use maxFontSizeMultiplier on Text components
□ Never hardcode fontSize: 16 directly in components

SCALING
□ Use moderateScale for padding/margin
□ Use scale for horizontal dimensions
□ Use verticalScale for vertical dimensions
□ Define a BASE_WIDTH constant matching your design specs

IMAGES
□ Never use fixed image width/height without aspectRatio
□ Use resizeMode="cover" for background-style images
□ Use expo-image for performance-critical image lists

ORIENTATION
□ Use useWindowDimensions() (not Dimensions.get)
□ Adjust layout direction based on width vs height comparison
□ Re-key FlatList when numColumns changes

PERFORMANCE
□ Memo-ize FlatList renderItem components
□ Use useCallback for event handlers passed to children
□ Set windowSize and initialNumToRender on FlatList
□ Compute StyleSheet values outside component body

TESTING
□ Tested on 320px width (small phone minimum)
□ Tested on 600px+ (tablet)
□ Tested landscape orientation
□ Tested with accessibility large text
□ Tested on real device (iOS + Android)
```

---

## 14. Complete Example

A fully responsive screen featuring a header, search bar, category chips, and a dynamic product grid.

```jsx
// screens/HomeScreen.js
import React, { useState, useCallback, memo } from 'react';
import {
  View,
  Text,
  TextInput,
  FlatList,
  TouchableOpacity,
  StyleSheet,
  useWindowDimensions,
} from 'react-native';
import { SafeAreaView, useSafeAreaInsets } from 'react-native-safe-area-context';
import { Image } from 'expo-image';
import { moderateScale, scale, verticalScale } from 'react-native-size-matters';
import { RFValue } from 'react-native-responsive-fontsize';

// ── Constants ─────────────────────────────────────────────────────────────────

const COLORS = {
  primary:    '#4F8EF7',
  background: '#F7F9FC',
  card:       '#FFFFFF',
  text:       '#1A1A2E',
  muted:      '#8A8FAB',
  border:     '#E8EBF5',
};

const FONT = {
  xs:      RFValue(10),
  sm:      RFValue(12),
  base:    RFValue(14),
  md:      RFValue(16),
  lg:      RFValue(20),
  xl:      RFValue(24),
};

const CATEGORIES = ['All', 'Popular', 'New', 'Sale', 'Featured'];

const PRODUCTS = Array.from({ length: 12 }, (_, i) => ({
  id: String(i + 1),
  title: `Product ${i + 1}`,
  price: `$${(9.99 + i * 5).toFixed(2)}`,
  color: ['#4F8EF7', '#F74F8E', '#4FF7A0', '#F7C94F'][i % 4],
}));

// ── Sub-components ────────────────────────────────────────────────────────────

const CategoryChip = memo(({ label, active, onPress }) => (
  <TouchableOpacity
    onPress={onPress}
    style={[styles.chip, active && styles.chipActive]}
    activeOpacity={0.7}
  >
    <Text style={[styles.chipText, active && styles.chipTextActive]}>
      {label}
    </Text>
  </TouchableOpacity>
));

const ProductCard = memo(({ item, cardWidth }) => (
  <View style={[styles.card, { width: cardWidth }]}>
    <View style={[styles.cardImage, { backgroundColor: item.color }]}>
      {/* Replace with <Image /> for real photos */}
    </View>
    <View style={styles.cardBody}>
      <Text style={styles.cardTitle} numberOfLines={1}>{item.title}</Text>
      <Text style={styles.cardPrice}>{item.price}</Text>
    </View>
    <TouchableOpacity style={styles.addButton} activeOpacity={0.8}>
      <Text style={styles.addButtonText}>Add</Text>
    </TouchableOpacity>
  </View>
));

// ── Main Screen ───────────────────────────────────────────────────────────────

export default function HomeScreen() {
  const { width } = useWindowDimensions();
  const insets    = useSafeAreaInsets();

  const [query,    setQuery]    = useState('');
  const [category, setCategory] = useState('All');

  // Responsive grid: 2 columns on phones, 3 on tablets
  const isTablet  = width > 600;
  const columns   = isTablet ? 3 : 2;
  const gap       = scale(12);
  const hPadding  = scale(16);
  const cardWidth = (width - hPadding * 2 - gap * (columns - 1)) / columns;

  const handleCategory = useCallback((cat) => setCategory(cat), []);

  const renderProduct = useCallback(
    ({ item }) => <ProductCard item={item} cardWidth={cardWidth} />,
    [cardWidth]
  );

  const ListHeader = (
    <>
      {/* ── Header ── */}
      <View style={styles.header}>
        <View>
          <Text style={styles.greeting}>Good morning 👋</Text>
          <Text style={styles.headline}>Discover Products</Text>
        </View>
        <View style={styles.avatar} />
      </View>

      {/* ── Search ── */}
      <View style={styles.searchRow}>
        <TextInput
          value={query}
          onChangeText={setQuery}
          placeholder="Search products…"
          placeholderTextColor={COLORS.muted}
          style={styles.searchInput}
        />
      </View>

      {/* ── Categories ── */}
      <FlatList
        horizontal
        data={CATEGORIES}
        keyExtractor={(c) => c}
        showsHorizontalScrollIndicator={false}
        contentContainerStyle={styles.categoriesContent}
        renderItem={({ item }) => (
          <CategoryChip
            label={item}
            active={item === category}
            onPress={() => handleCategory(item)}
          />
        )}
      />

      {/* ── Section Title ── */}
      <View style={styles.sectionRow}>
        <Text style={styles.sectionTitle}>Results</Text>
        <Text style={styles.sectionCount}>{PRODUCTS.length} items</Text>
      </View>
    </>
  );

  return (
    <SafeAreaView style={styles.screen} edges={['top']}>
      <FlatList
        data={PRODUCTS}
        keyExtractor={(item) => item.id}
        numColumns={columns}
        key={columns}                         // Re-key on column count change
        ListHeaderComponent={ListHeader}
        renderItem={renderProduct}
        contentContainerStyle={[
          styles.listContent,
          { paddingBottom: insets.bottom + verticalScale(20) },
        ]}
        columnWrapperStyle={columns > 1 ? styles.row : undefined}
        showsVerticalScrollIndicator={false}
        windowSize={5}
        initialNumToRender={8}
        maxToRenderPerBatch={6}
      />
    </SafeAreaView>
  );
}

// ── Styles ────────────────────────────────────────────────────────────────────

const styles = StyleSheet.create({
  screen: {
    flex: 1,
    backgroundColor: COLORS.background,
  },
  listContent: {
    paddingHorizontal: scale(16),
  },

  // Header
  header: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    alignItems: 'center',
    paddingVertical: verticalScale(16),
  },
  greeting: {
    fontSize: FONT.sm,
    color: COLORS.muted,
    fontWeight: '500',
  },
  headline: {
    fontSize: FONT.xl,
    color: COLORS.text,
    fontWeight: '700',
    marginTop: verticalScale(2),
  },
  avatar: {
    width: moderateScale(44),
    aspectRatio: 1,
    borderRadius: moderateScale(22),
    backgroundColor: COLORS.primary,
  },

  // Search
  searchRow: {
    marginBottom: verticalScale(16),
  },
  searchInput: {
    backgroundColor: COLORS.card,
    borderRadius: moderateScale(12),
    paddingHorizontal: scale(16),
    paddingVertical: verticalScale(12),
    fontSize: FONT.base,
    color: COLORS.text,
    borderWidth: 1,
    borderColor: COLORS.border,
  },

  // Categories
  categoriesContent: {
    gap: scale(8),
    paddingBottom: verticalScale(16),
  },
  chip: {
    paddingHorizontal: scale(16),
    paddingVertical: verticalScale(8),
    borderRadius: moderateScale(20),
    backgroundColor: COLORS.card,
    borderWidth: 1,
    borderColor: COLORS.border,
  },
  chipActive: {
    backgroundColor: COLORS.primary,
    borderColor: COLORS.primary,
  },
  chipText: {
    fontSize: FONT.sm,
    color: COLORS.muted,
    fontWeight: '600',
  },
  chipTextActive: {
    color: '#fff',
  },

  // Section
  sectionRow: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    alignItems: 'center',
    marginBottom: verticalScale(12),
  },
  sectionTitle: {
    fontSize: FONT.md,
    fontWeight: '700',
    color: COLORS.text,
  },
  sectionCount: {
    fontSize: FONT.sm,
    color: COLORS.muted,
  },

  // Grid
  row: {
    gap: scale(12),
    marginBottom: verticalScale(12),
  },

  // Product Card
  card: {
    backgroundColor: COLORS.card,
    borderRadius: moderateScale(14),
    overflow: 'hidden',
    borderWidth: 1,
    borderColor: COLORS.border,
  },
  cardImage: {
    width: '100%',
    aspectRatio: 1.2,
  },
  cardBody: {
    padding: moderateScale(10),
  },
  cardTitle: {
    fontSize: FONT.sm,
    fontWeight: '600',
    color: COLORS.text,
  },
  cardPrice: {
    fontSize: FONT.sm,
    color: COLORS.primary,
    fontWeight: '700',
    marginTop: verticalScale(2),
  },
  addButton: {
    backgroundColor: COLORS.primary,
    margin: moderateScale(10),
    marginTop: 0,
    paddingVertical: verticalScale(8),
    borderRadius: moderateScale(8),
    alignItems: 'center',
  },
  addButtonText: {
    color: '#fff',
    fontSize: FONT.sm,
    fontWeight: '700',
  },
});
```

---

## Summary

| Topic | Key Takeaway |
|-------|-------------|
| Flexbox | Default layout system — master `flex`, `flexDirection`, `justifyContent`, `alignItems` |
| Dimensions | Use `useWindowDimensions()` for reactive, orientation-aware values |
| Scaling | Use `moderateScale` for fonts/padding, `scale` for widths |
| Typography | Centralize font sizes using `RFValue` in a constants file |
| Safe Area | Always wrap screens with `SafeAreaView` from `react-native-safe-area-context` |
| Images | Use `aspectRatio` + `resizeMode`, avoid fixed dimensions |
| Testing | Test on real devices across sizes and orientations |

---

*Built with ❤️ for React Native + Expo developers. Happy shipping!*
