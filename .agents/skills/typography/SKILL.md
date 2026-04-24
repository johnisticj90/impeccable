# Typography Skill

Analyze and improve typographic choices in UI designs and stylesheets.

## Purpose

Ensure consistent, readable, and aesthetically sound typography across the project by evaluating type scales, line heights, font pairing, and readability metrics.

## Functions

### `getTypeScale(base, ratio)`

Generates a modular type scale given a base size and ratio.

```javascript
/**
 * Generate a modular type scale.
 * @param {number} base - Base font size in px (typically 16)
 * @param {number} ratio - Scale ratio (e.g. 1.25 = Major Third, 1.333 = Perfect Fourth)
 * @param {number} steps - Number of steps above and below base
 * @returns {Object} Scale object with named steps
 */
function getTypeScale(base = 16, ratio = 1.25, steps = 6) {
  const scale = {};
  const names = ['xs', 'sm', 'base', 'md', 'lg', 'xl', '2xl', '3xl'];

  for (let i = -2; i <= steps; i++) {
    const size = base * Math.pow(ratio, i);
    scale[i] = Math.round(size * 100) / 100;
  }

  return scale;
}
```

### `isOnTypeScale(size, base, ratio)`

Checks whether a given font size falls on (or near) a modular type scale.

```javascript
/**
 * Check if a font size aligns with the modular type scale.
 * @param {number} size - Font size in px to check
 * @param {number} base - Base font size
 * @param {number} ratio - Scale ratio
 * @param {number} tolerance - Allowed deviation in px
 * @returns {boolean}
 */
function isOnTypeScale(size, base = 16, ratio = 1.25, tolerance = 0.5) {
  const scale = getTypeScale(base, ratio, 10);
  return Object.values(scale).some(s => Math.abs(s - size) <= tolerance);
}
```

### `recommendLineHeight(fontSize)`

Returns a recommended unitless line-height for a given font size, following readability best practices.

```javascript
/**
 * Recommend a unitless line-height for a given font size.
 * Larger text needs tighter leading; body text needs looser.
 * @param {number} fontSize - Font size in px
 * @returns {number} Recommended unitless line-height
 */
function recommendLineHeight(fontSize) {
  if (fontSize >= 48) return 1.1;
  if (fontSize >= 32) return 1.2;
  if (fontSize >= 24) return 1.3;
  if (fontSize >= 18) return 1.4;
  return 1.5; // body text default
}
```

### `measureReadability(charsPerLine)`

Evaluates whether a line length is within the optimal readability range.

```javascript
/**
 * Evaluate line length for readability.
 * Optimal range is 45–75 characters per line.
 * @param {number} charsPerLine - Average characters per line
 * @returns {{ score: string, message: string }}
 */
function measureReadability(charsPerLine) {
  if (charsPerLine < 30) {
    return { score: 'poor', message: 'Lines too short — creates choppy reading rhythm.' };
  }
  if (charsPerLine < 45) {
    return { score: 'fair', message: 'Lines slightly short — consider wider container or smaller font.' };
  }
  if (charsPerLine <= 75) {
    return { score: 'good', message: 'Line length is within the optimal readability range.' };
  }
  if (charsPerLine <= 90) {
    return { score: 'fair', message: 'Lines slightly long — may cause eye-tracking issues.' };
  }
  return { score: 'poor', message: 'Lines too long — readers will lose their place.' };
}
```

### `snapToTypeScale(size, base, ratio)`

Snaps an arbitrary font size to the nearest step on the modular type scale.

```javascript
/**
 * Snap a font size to the nearest modular scale value.
 * @param {number} size - Font size in px
 * @param {number} base - Base font size
 * @param {number} ratio - Scale ratio
 * @returns {number} Nearest scale value in px
 */
function snapToTypeScale(size, base = 16, ratio = 1.25) {
  const scale = getTypeScale(base, ratio, 10);
  const values = Object.values(scale);
  return values.reduce((closest, val) =>
    Math.abs(val - size) < Math.abs(closest - size) ? val : closest
  );
}
```

## Common Ratios

| Name            | Ratio  |
|-----------------|--------|
| Minor Second    | 1.067  |
| Major Second    | 1.125  |
| Minor Third     | 1.200  |
| Major Third     | 1.250  |
| Perfect Fourth  | 1.333  |
| Augmented Fourth| 1.414  |
| Perfect Fifth   | 1.500  |
| Golden Ratio    | 1.618  |

## Usage Notes

- Always use `getTypeScale` to establish a baseline before auditing existing font sizes.
- Cross-reference with `contrastRatio` (from `document/SKILL.md`) when evaluating small text — WCAG AA requires 4.5:1 for text under 18px.
- Pair with `snapToScale` from `spacing/SKILL.md` to ensure font sizes and spacing share the same rhythm.
- Prefer unitless line-heights to avoid inheritance issues in nested elements.

## Anti-Patterns to Flag

- Font sizes that don't align to any recognizable scale
- Line heights set in px (brittle, breaks on zoom)
- More than 4–5 distinct font sizes in a single view
- Body text smaller than 14px
- Heading hierarchy that skips levels (h1 → h3)
