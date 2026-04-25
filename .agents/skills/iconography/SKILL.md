# Skill: Iconography

Analyze and validate icon usage, sizing, optical alignment, and visual consistency across icon sets.

## Responsibilities

- Validate icon sizes against a consistent scale
- Check optical sizing adjustments for different contexts
- Detect icon/label alignment issues
- Recommend touch target sizes for interactive icons
- Verify icon weight/style consistency

## Scale

Icons should follow a discrete size scale, typically multiples of 4px:

```
12, 16, 20, 24, 32, 40, 48, 64
```

Interactive icons require a minimum touch target of 44×44px (WCAG 2.5.5).

## Functions

### `getIconScale()`

Returns the standard icon size scale.

```js
function getIconScale() {
  return [12, 16, 20, 24, 32, 40, 48, 64];
}
```

### `isOnIconScale(size)`

Checks whether a given size is on the icon scale.

```js
function isOnIconScale(size) {
  return getIconScale().includes(size);
}
```

**Example:**
```js
isOnIconScale(24); // true
isOnIconScale(22); // false
```

### `snapToIconScale(size)`

Snaps an arbitrary size to the nearest value on the icon scale.

```js
function snapToIconScale(size) {
  const scale = getIconScale();
  return scale.reduce((prev, curr) =>
    Math.abs(curr - size) < Math.abs(prev - size) ? curr : prev
  );
}
```

**Example:**
```js
snapToIconScale(22); // 24
snapToIconScale(14); // 12 or 16 depending on proximity
```

### `checkTouchTarget(iconSize, targetSize)`

Verifies that an interactive icon has an adequate touch target.

Returns an object with `passes` (boolean) and `recommendation` (string).

```js
function checkTouchTarget(iconSize, targetSize) {
  const MIN_TARGET = 44;
  const passes = targetSize >= MIN_TARGET;
  return {
    passes,
    iconSize,
    targetSize,
    recommendation: passes
      ? `Touch target meets WCAG 2.5.5 (${targetSize}px >= ${MIN_TARGET}px)`
      : `Increase touch target to at least ${MIN_TARGET}px (currently ${targetSize}px). Consider adding padding around the ${iconSize}px icon.`
  };
}
```

**Example:**
```js
checkTouchTarget(16, 32);
// {
//   passes: false,
//   iconSize: 16,
//   targetSize: 32,
//   recommendation: "Increase touch target to at least 44px..."
// }

checkTouchTarget(24, 44);
// { passes: true, ... }
```

### `opticalSize(logicalSize, context)`

Returns an optically adjusted icon size for a given rendering context.

Some contexts (dense UIs, large displays) benefit from slight size adjustments to maintain visual weight parity.

```js
/**
 * @param {number} logicalSize - The intended logical icon size in px
 * @param {'default'|'dense'|'large'} context - Rendering context
 * @returns {number} Optically adjusted size, snapped to scale
 */
function opticalSize(logicalSize, context) {
  const adjustments = {
    default: 0,
    dense: -4,
    large: +8
  };
  const delta = adjustments[context] ?? 0;
  return snapToIconScale(logicalSize + delta);
}
```

**Example:**
```js
opticalSize(24, 'dense');  // 20
opticalSize(24, 'large');  // 32
opticalSize(24, 'default'); // 24
```

### `checkIconLabelAlignment(iconSize, labelFontSize, lineHeight)`

Checks whether an icon will appear vertically centered relative to a text label.

Returns alignment quality: `'good'`, `'acceptable'`, or `'poor'`.

```js
function checkIconLabelAlignment(iconSize, labelFontSize, lineHeight) {
  const labelLineHeightPx = labelFontSize * lineHeight;
  const diff = Math.abs(iconSize - labelLineHeightPx);
  const ratio = diff / labelLineHeightPx;

  if (ratio <= 0.05) return { quality: 'good', diff };
  if (ratio <= 0.15) return { quality: 'acceptable', diff };
  return {
    quality: 'poor',
    diff,
    recommendation: `Icon size (${iconSize}px) differs from label line-height (${labelLineHeightPx}px) by ${Math.round(ratio * 100)}%. Consider using a ${snapToIconScale(labelLineHeightPx)}px icon or adjusting line-height.`
  };
}
```

**Example:**
```js
checkIconLabelAlignment(24, 16, 1.5); // line-height = 24px → good
checkIconLabelAlignment(24, 14, 1.5); // line-height = 21px → acceptable
checkIconLabelAlignment(32, 12, 1.2); // line-height = 14.4px → poor
```

## Usage Notes

- Always prefer named semantic icons over decorative ones for accessible UIs
- Pair icon sizes with the type scale from the `typography` skill for label alignment
- Use `opticalSize` when icons appear in toolbars (dense) or hero sections (large)
- Icons used without visible labels **must** have `aria-label` or `title` attributes
- Stroke-based icons may need weight adjustments at very small (<16px) or large (>48px) sizes

## Related Skills

- `typography` — for pairing icon sizes with text
- `spacing` — for padding/touch target calculations
- `contrast` — for icon color accessibility
