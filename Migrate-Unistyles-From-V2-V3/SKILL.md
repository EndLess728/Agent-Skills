---
name: unistyles-v2-to-v3
description: Migrate react-native-unistyles from v2 to v3. Use when upgrading Unistyles, fixing "Style is not bound!" errors, converting createStyleSheet/useStyles, or merging styles with array syntax.
---

# Unistyles v2 → v3 Migration Guide

A complete reference for migrating a React Native project from `react-native-unistyles` v2 to v3.

---

## Overview of What Changed

| v2 API                             | v3 Replacement                                             |
| ---------------------------------- | ---------------------------------------------------------- |
| `UnistylesRegistry.addConfig(...)` | `StyleSheet.configure(...)`                                |
| `createStyleSheet(...)`            | `StyleSheet.create(...)`                                   |
| `useStyles(stylesheet)`            | Removed — styles used directly                             |
| `useStyles(stylesheet, variants)`  | `styles.useVariants(variants)`                             |
| `UnistylesProvider`                | Removed entirely                                           |
| `useInitialTheme`                  | `settings.initialTheme` callback in `StyleSheet.configure` |
| `UnistylesRuntime.hairlineWidth`   | `StyleSheet.hairlineWidth`                                 |
| `rt.insets.bottom` (dynamic)       | `rt.insets.ime` for keyboard-aware bottom inset            |
| Spread to merge styles             | Array syntax: `[style1, style2]`                           |

---

## Step-by-Step Migration

### Step 1 — Install v3 and run getting-started setup

Follow the [Getting started](https://www.unistyl.es/v3/start/getting-started) guide.
Ensure Babel plugin is updated in `babel.config.js` as required by v3.

---

### Step 2 — Replace `UnistylesRegistry` with `StyleSheet.configure`

**v2:**

```ts
import { UnistylesRegistry } from 'react-native-unistyles'

UnistylesRegistry.addConfig({
  adaptiveThemes: false,
  initialTheme: 'dark',
  plugins: [...],
  experimentalCSSMediaQueries: true,
  windowResizeDebounceTimeMs: 100,
  disableAnimatedInsets: true,
})
```

**v3:**

```ts
import { StyleSheet } from "react-native-unistyles";

StyleSheet.configure({
  themes, // same shape as v2
  breakpoints, // same shape as v2
  settings: {
    adaptiveThemes: false, // unchanged
    initialTheme: "dark", // unchanged
    // plugins → REMOVED (use plain functions to transform styles)
    // experimentalCSSMediaQueries → REMOVED (enabled by default in v3)
    // windowResizeDebounceTimeMs → REMOVED (CSS media queries, no debounce)
    // disableAnimatedInsets → REMOVED (insets no longer re-render views)
  },
});
```

> Themes and Breakpoints definitions are identical to v2 — no changes needed there.

---

### Step 3 — Replace `createStyleSheet` + `useStyles` with `StyleSheet.create`

**v2:**

```ts
import { createStyleSheet, useStyles } from 'react-native-unistyles'

const stylesheet = createStyleSheet(theme => ({
  container: {
    backgroundColor: theme.colors.background,
  },
}))

const MyComponent = () => {
  const { styles } = useStyles(stylesheet)
  return <View style={styles.container} />
}
```

**v3:**

```ts
import { StyleSheet } from 'react-native-unistyles'

const styles = StyleSheet.create(theme => ({
  container: {
    backgroundColor: theme.colors.background,
  },
}))

const MyComponent = () => {
  return <View style={styles.container} />
}
```

Key changes:

- `createStyleSheet` → `StyleSheet.create`
- `useStyles` hook is **removed** — styles are used directly
- Variable renamed from `stylesheet` to `styles` (convention)
- No hook call needed inside the component

---

### Step 4 — Migrate variants from `useStyles` second argument to `styles.useVariants`

**v2:**

```ts
const { styles } = useStyles(stylesheet, {
  variant1: "primary",
  variant2: "secondary",
});
```

**v3:**

```ts
// Call useVariants inside the component, before using styles
styles.useVariants({
  variant1: "primary",
  variant2: "secondary",
});
```

> `styles.useVariants` must be called inside the component render (it's a hook-like call tied to the styles object).

---

### Step 5 — Migrate `theme` access in components

**v2 (accessing theme inline):**

```ts
const MyButton = () => {
  const { theme } = useStyles(stylesheet)
  return <Button color={theme.colors.primary} />
}
```

**v3 option A — `withUnistyles` (recommended, no re-renders):**

```ts
import { Button } from 'react-native'
import { withUnistyles } from 'react-native-unistyles'

const UniButton = withUnistyles(Button, theme => ({
  color: theme.colors.primary,
}))

const MyButton = () => <UniButton />
```

**v3 option B — `useUnistyles` hook (quick migration path, causes re-renders):**

```ts
import { useUnistyles } from 'react-native-unistyles'

const MyButton = () => {
  const { theme } = useUnistyles()
  return <Button color={theme.colors.primary} />
}
```

> Use `withUnistyles` for new code. Use `useUnistyles` when you want a faster drop-in
> replacement during migration — it behaves closer to v2's `useStyles` but still re-renders the component.

---

### Step 6 — Remove `UnistylesProvider`

**v2:**

```tsx
import { UnistylesProvider } from "react-native-unistyles";

<UnistylesProvider>
  <App />
</UnistylesProvider>;
```

**v3:** Delete entirely. No provider is needed.

---

### Step 7 — Migrate `useInitialTheme`

**v2:**

```ts
useInitialTheme("dark");
```

**v3:** Set it synchronously in the `StyleSheet.configure` call:

```ts
StyleSheet.configure({
  settings: {
    initialTheme: () => {
      // Must be synchronous — read from MMKV/AsyncStorage sync API/etc.
      return storage.getString("preferredTheme") ?? "light";
    },
  },
});
```

---

### Step 8 — Fix `hairlineWidth` usage

**v2:**

```ts
UnistylesRuntime.hairlineWidth;
```

**v3:**

```ts
StyleSheet.hairlineWidth;
```

---

### Step 9 — Fix keyboard-aware insets

**v2:**

```ts
const style = StyleSheet.create({
  container: {
    paddingBottom: rt.insets.bottom, // was dynamic, moved with keyboard
  },
});
```

**v3:**

```ts
const styles = StyleSheet.create({
  container: {
    paddingBottom: rt.insets.ime, // use ime inset for keyboard position
  },
});
```

> `rt.insets.bottom` is no longer dynamic in v3 — use `rt.insets.ime` for keyboard-aware bottom padding.

---

### Step 10 — Replace `Display`/`Hide` for breakpoint-based visibility

**v2 (conditional rendering via breakpoint in component):**

```ts
const { breakpoint } = useStyles(stylesheet);
// manual conditional render based on breakpoint
```

**v3:**

```tsx
import { Display, Hide, mq } from 'react-native-unistyles'

<Display mq={mq.only.width(0, 400)}>
  <Text>Visible only on small screens</Text>
</Display>

<Hide mq={mq.only.width(400)}>
  <Text>Hidden on large screens</Text>
</Hide>
```

---

### Step 11 — Remove deleted `UnistylesRuntime` methods

These have been removed in v3:

```ts
// REMOVED — no plugins in v3
UnistylesRuntime.addPlugin(plugin);
UnistylesRuntime.removePlugin(plugin);

// REMOVED — deprecated on Android 15
UnistylesRuntime.statusBar.setColor(color);
UnistylesRuntime.navigationBar.setColor(color);
```

`setRootViewBackgroundColor` now accepts a single color (no separate `alpha` arg):

```ts
// v2
UnistylesRuntime.setRootViewBackgroundColor(color, alpha);

// v3
UnistylesRuntime.setRootViewBackgroundColor(color); // any RN-compatible color string
```

> Check TypeScript types for other renamed `UnistylesRuntime` methods — the compiler will guide you.

---

## Critical: Merging Styles in v3

**This is the most common source of errors after migration.**

In v3, Unistyles attaches C++ state to each style object. Spreading styles destroys this state.

### ❌ Never spread Unistyles styles

```tsx
// BROKEN — destroys C++ state, causes "Style is not bound!" error
const merged = { ...styles.container, ...styles.container2 }
<View style={merged} />

// BROKEN — same issue
<View style={{ ...styles.container, backgroundColor: 'red' }} />

// BROKEN — same issue
<View style={{ ...styles.container, ...{ backgroundColor: 'red' } }} />
```

### ✅ Always use array syntax

```tsx
// CORRECT
<View style={[styles.container, styles.container2]} />

// CORRECT — inline overrides
<View style={[styles.container, { backgroundColor: 'red' }]} />

// CORRECT — conditional styles
<View style={[styles.container, isActive && styles.active]} />
```

### Why this matters

- Spreading removes the C++ state needed for Unistyles to track which styles to recompute
- V3 will attempt to restore state but **loses merge order** — styles may apply incorrectly
- After restoration, **future style updates may not trigger at all**
- In `__DEV__` mode, Unistyles logs a warning when it detects this:
  ```
  Unistyles: We detected a style object with N Unistyles styles.
  This might cause no updates or unpredictable behavior.
  Please check the `style` prop for `View` and use array syntax instead.
  ```

**Ship zero of these warnings before release.**

### Reanimated note

If using Reanimated, upgrade to `react-native-reanimated >= 3.17.2` or `>= 4.0.0-beta.3`.
Older versions flattened style arrays, which broke Unistyles v3 multi-style passing.

---

## Quick Reference: Import Changes

```ts
// v2 imports
import {
  createStyleSheet,
  useStyles,
  UnistylesRegistry,
  UnistylesProvider,
} from "react-native-unistyles";

// v3 imports
import {
  StyleSheet,
  withUnistyles,
  useUnistyles,
  Display,
  Hide,
  mq,
} from "react-native-unistyles";
```

---

## Common Errors After Migration

| Error / Warning                                               | Cause                          | Fix                                                   |
| ------------------------------------------------------------- | ------------------------------ | ----------------------------------------------------- |
| `Style is not bound!`                                         | Spreading a Unistyle object    | Use `[style1, style2]` array syntax                   |
| `Unistyles: we detected style object with N unistyles styles` | Spreading styles               | Use array syntax                                      |
| Styles not updating after theme change                        | Spread operator used somewhere | Audit all `style` props for spreads                   |
| `useStyles is not a function`                                 | Still importing v2 API         | Switch to `StyleSheet.create` + direct `styles` usage |
| Theme not applied on startup                                  | `useInitialTheme` removed      | Use `initialTheme` callback in `StyleSheet.configure` |
| Keyboard not shifting content                                 | Using `rt.insets.bottom`       | Switch to `rt.insets.ime`                             |
