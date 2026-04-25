# Contrast Skill

Analyzes and improves color contrast ratios for accessibility compliance.

## Overview

This skill evaluates foreground/background color pairs against WCAG 2.1 contrast requirements and suggests accessible alternatives when needed.

## Functions

### `contrastRatio(fg, bg)`

Calculates the contrast ratio between two colors.

**Parameters:**
- `fg` — foreground color (hex, rgb, or hsl string)
- `bg` — background color (hex, rgb, or hsl string)

**Returns:** `number` — contrast ratio (1–21)

```js
contrastRatio('#000000', '#ffffff') // 21
contrastRatio('#767676', '#ffffff') // ~4.54
```

---

### `meetsWCAG(fg, bg, level, size)`

Checks whether a color pair meets WCAG 2.1 contrast requirements.

**Parameters:**
- `fg` — foreground color string
- `bg` — background color string
- `level` — `'AA'` or `'AAA'` (default: `'AA'`)
- `size` — `'normal'` or `'large'` (default: `'normal'`)

**Returns:** `boolean`

WCAG thresholds:
| Level | Normal text | Large text (18pt+ or 14pt+ bold) |
|-------|-------------|-----------------------------------|
| AA    | 4.5:1       | 3:1                               |
| AAA   | 7:1         | 4.5:1                             |

```js
meetsWCAG('#767676', '#ffffff', 'AA', 'normal') // false (4.54 < 4.5... actually passes)
meetsWCAG('#767676', '#ffffff', 'AAA', 'normal') // false
meetsWCAG('#595959', '#ffffff', 'AAA', 'normal') // true
```

---

### `suggestAccessibleColor(fg, bg, level, size)`

If the color pair fails WCAG, returns the closest passing variant of `fg` by adjusting lightness.

**Parameters:**
- `fg` — foreground color string to adjust
- `bg` — background color string (kept fixed)
- `level` — `'AA'` or `'AAA'` (default: `'AA'`)
- `size` — `'normal'` or `'large'` (default: `'normal'`)

**Returns:** `string` — hex color that passes the given WCAG level, or original if already passing

```js
suggestAccessibleColor('#aaaaaa', '#ffffff', 'AA') // '#767676' or darker
```

---

### `auditPalette(pairs, level)`

Audits an array of color pairs and returns a report.

**Parameters:**
- `pairs` — array of `{ name, fg, bg }` objects
- `level` — `'AA'` or `'AAA'` (default: `'AA'`)

**Returns:** array of audit result objects

```js
auditPalette([
  { name: 'body text', fg: '#333333', bg: '#ffffff' },
  { name: 'placeholder', fg: '#aaaaaa', bg: '#ffffff' },
], 'AA')
// [
//   { name: 'body text', ratio: 12.63, passes: true },
//   { name: 'placeholder', ratio: 2.32, passes: false, suggestion: '#767676' }
// ]
```

---

## Implementation

```js
/**
 * Convert a hex color string to linear RGB [0..1] components.
 * Supports 3- and 6-digit hex.
 */
function hexToLinearRgb(hex) {
  const clean = hex.replace('#', '');
  const full = clean.length === 3
    ? clean.split('').map(c => c + c).join('')
    : clean;

  const r = parseInt(full.slice(0, 2), 16) / 255;
  const g = parseInt(full.slice(2, 4), 16) / 255;
  const b = parseInt(full.slice(4, 6), 16) / 255;

  return [r, g, b].map(c =>
    c <= 0.03928 ? c / 12.92 : Math.pow((c + 0.055) / 1.055, 2.4)
  );
}

/**
 * Calculate relative luminance of a color per WCAG 2.1.
 */
function relativeLuminance(hex) {
  const [r, g, b] = hexToLinearRgb(hex);
  return 0.2126 * r + 0.7152 * g + 0.0722 * b;
}

/**
 * Calculate contrast ratio between two hex colors.
 * Returns a value between 1 and 21.
 */
function contrastRatio(fg, bg) {
  const l1 = relativeLuminance(fg);
  const l2 = relativeLuminance(bg);
  const lighter = Math.max(l1, l2);
  const darker = Math.min(l1, l2);
  return (lighter + 0.05) / (darker + 0.05);
}

/** WCAG minimum contrast thresholds */
const WCAG_THRESHOLDS = {
  AA:  { normal: 4.5, large: 3.0 },
  AAA: { normal: 7.0, large: 4.5 },
};

/**
 * Check whether a color pair meets WCAG contrast requirements.
 */
function meetsWCAG(fg, bg, level = 'AA', size = 'normal') {
  const ratio = contrastRatio(fg, bg);
  const threshold = WCAG_THRESHOLDS[level][size];
  return ratio >= threshold;
}

/**
 * Adjust hex lightness by a delta (-100 to +100).
 * Converts through HSL space.
 */
function adjustLightness(hex, delta) {
  const [r, g, b] = hexToLinearRgb(hex).map(c =>
    c <= 0.0031308 ? c * 12.92 : 1.055 * Math.pow(c, 1 / 2.4) - 0.055
  );

  const max = Math.max(r, g, b);
  const min = Math.min(r, g, b);
  let h, s;
  let l = (max + min) / 2;

  if (max === min) {
    h = s = 0;
  } else {
    const d = max - min;
    s = l > 0.5 ? d / (2 - max - min) : d / (max + min);
    switch (max) {
      case r: h = ((g - b) / d + (g < b ? 6 : 0)) / 6; break;
      case g: h = ((b - r) / d + 2) / 6; break;
      case b: h = ((r - g) / d + 4) / 6; break;
    }
  }

  l = Math.min(1, Math.max(0, l + delta / 100));

  // HSL back to hex
  const hue2rgb = (p, q, t) => {
    if (t < 0) t += 1;
    if (t > 1) t -= 1;
    if (t < 1/6) return p + (q - p) * 6 * t;
    if (t < 1/2) return q;
    if (t < 2/3) return p + (q - p) * (2/3 - t) * 6;
    return p;
  };

  let nr, ng, nb;
  if (s === 0) {
    nr = ng = nb = l;
  } else {
    const q = l < 0.5 ? l * (1 + s) : l + s - l * s;
    const p = 2 * l - q;
    nr = hue2rgb(p, q, h + 1/3);
    ng = hue2rgb(p, q, h);
    nb = hue2rgb(p, q, h - 1/3);
  }

  return '#' + [nr, ng, nb]
    .map(c => Math.round(c * 255).toString(16).padStart(2, '0'))
    .join('');
}

/**
 * Suggest the nearest accessible foreground color by darkening or
 * lightening until the pair meets the given WCAG level.
 */
function suggestAccessibleColor(fg, bg, level = 'AA', size = 'normal') {
  if (meetsWCAG(fg, bg, level, size)) return fg;

  const bgLum = relativeLuminance(bg);
  // Darken if bg is light, lighten if bg is dark
  const direction = bgLum > 0.5 ? -1 : 1;

  for (let step = 1; step <= 100; step++) {
    const candidate = adjustLightness(fg, direction * step);
    if (meetsWCAG(candidate, bg, level, size)) return candidate;
  }

  return fg; // fallback — should not reach here
}

/**
 * Audit an array of { name, fg, bg } color pairs.
 * Returns results with ratio, pass/fail, and suggestions.
 */
function auditPalette(pairs, level = 'AA') {
  return pairs.map(({ name, fg, bg }) => {
    const ratio = contrastRatio(fg, bg);
    const passes = meetsWCAG(fg, bg, level);
    const result = {
      name,
      fg,
      bg,
      ratio: Math.round(ratio * 100) / 100,
      passes,
    };
    if (!passes) {
      result.suggestion = suggestAccessibleColor(fg, bg, level);
    }
    return result;
  });
}
```

## Usage Notes

- All color inputs should be **6-digit hex strings** (e.g. `#ffffff`). 3-digit hex is also supported.
- "Large text" per WCAG means ≥18pt regular or ≥14pt bold (~24px / ~18.67px respectively).
- `suggestAccessibleColor` adjusts lightness only — hue and saturation are preserved.
- For UI auditing workflows, combine with the `colorize` skill to generate full palette reports.
