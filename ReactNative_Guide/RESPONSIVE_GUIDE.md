# Mobile Responsiveness Reference Guide 📱

This guide explains how to use the centralized scaling utilities in `src/constants/fonts.ts` to ensure your application looks perfect on all screen sizes (Phones, Tablets, Android, iOS).

## 1. Scaling Functions `(horizontalScale, verticalScale, moderateScale)`

These functions take a "standard" design value (based on 375x812 design) and scale it to the current device's screen.

| Function | When to Use | Example |
| :--- | :--- | :--- |
| `horizontalScale` | Widths, Left/Right Paddings, Left/Right Margins. | `width: horizontalScale(300)` |
| `verticalScale` | Heights, Top/Bottom Paddings, Top/Bottom Margins. | `height: verticalScale(20)` |
| `moderateScale` | Icon sizes, Border Radius, Card Paddings. | `borderRadius: moderateScale(10)` |

## 2. Responsive Fonts `(FONT_SIZE)`

Never use numbers for `fontSize`. Use the `FONT_SIZE` constants which are powered by `react-native-responsive-fontsize`.

- `FONT_SIZE.xs` (10)
- `FONT_SIZE.small` (12)
- `FONT_SIZE.regular` (14)
- `FONT_SIZE.medium` (16)
- `FONT_SIZE.large` (18)
- `FONT_SIZE.xl` (22)
- `FONT_SIZE.xxl` (28)
- `FONT_SIZE.xxxl` (34)

---

## 3. Real-World Example

### Dashboard Header Refactor

**Before (Non-Responsive):**
```tsx
<View style={{ width: 375, height: 100, padding: 10 }}>
  <Text style={{ fontSize: 20 }}>Title</Text>
</View>
```

**After (Fully Responsive):**
```tsx
import { horizontalScale, verticalScale, moderateScale, FONT_SIZE } from '../constants/fonts';

<View style={{ 
  width: '100%', 
  height: verticalScale(100), 
  padding: moderateScale(10) 
}}>
  <Text style={{ fontSize: FONT_SIZE.xl }}>Title</Text>
</View>
```

---

## 4. Breakpoints for Tablet vs. Phone

If you need to change the **Layout** (e.g., from 1 column to 2 columns), use the screen width.

```tsx
import { useWindowDimensions } from 'react-native';

const { width } = useWindowDimensions();
const isTablet = width > 768;

const numColumns = isTablet ? 4 : 2;
```

---

## 5. Summary Checklist for Every Page
1. [ ] Wrap the screen in `SafeAreaView`.
2. [ ] Use imports from `fonts.ts` for all dimensions.
3. [ ] Use `flex: 1` to fill remaining space.
4. [ ] For horizontal alignment, use `%` widths with `mx-auto` or `moderateScale` for consistency.
5. [ ] Test on a small phone and a large tablet emulator.
