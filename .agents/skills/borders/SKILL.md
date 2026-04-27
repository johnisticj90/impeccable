# Borders Skill

Handles border widths, radii, and styles — ensuring consistency with a defined scale and accessibility considerations.

## Scale

Border widths follow a constrained set of values to maintain visual consistency:

| Token       | Value  |
|-------------|--------|
| none        | 0px    |
| hairline    | 0.5px  |
| thin        | 1px    |
| base        | 2px    |
| thick       | 4px    |
| heavy       | 8px    |

Border radii follow a geometric scale:

| Token       | Value  |
|-------------|--------|
| none        | 0px    |
| xs          | 2px    |
| sm          | 4px    |
| md          | 8px    |
| lg          | 12px   |
| xl          | 16px   |
| 2xl         | 24px   |
| full        | 9999px |

## Functions

### getBorderScale()
Returns the full border width scale as an object.

```js
function getBorderScale() {
  return {
    none: 0,
    hairline: 0.5,
    thin: 1,
    base: 2,
    thick: 4,
    heavy: 8,
  };
}
```

### getRadiusScale()
Returns the full border radius scale as an object.

```js
function getRadiusScale() {
  return {
    none: 0,
    xs: 2,
    sm: 4,
    md: 8,
    lg: 12,
    xl: 16,
    '2xl': 24,
    full: 9999,
  };
}
```

### isOnBorderScale(value)
Returns `true` if the given pixel value exists in the border width scale.

```js
function isOnBorderScale(value) {
  return Object.values(getBorderScale()).includes(value);
}
```

### isOnRadiusScale(value)
Returns `true` if the given pixel value exists in the radius scale.

```js
function isOnRadiusScale(value) {
  return Object.values(getRadiusScale()).includes(value);
}
```

### snapToBorderScale(value)
Snaps an arbitrary border width to the nearest scale value.

```js
function snapToBorderScale(value) {
  const scale = Object.values(getBorderScale());
  return scale.reduce((prev, curr) =>
    Math.abs(curr - value) < Math.abs(prev - value) ? curr : prev
  );
}
```

### snapToRadiusScale(value)
Snaps an arbitrary radius to the nearest scale value. Clamps at `full` (9999) for pill shapes.

```js
function snapToRadiusScale(value) {
  if (value >= 9999) return 9999;
  const scale = Object.values(getRadiusScale()).filter(v => v < 9999);
  return scale.reduce((prev, curr) =>
    Math.abs(curr - value) < Math.abs(prev - value) ? curr : prev
  );
}
```

### checkBorderContrast(borderColor, backgroundColor)
Checks whether a border provides enough contrast against its background to be perceivable. Uses a minimum contrast ratio of 3:1 per WCAG 1.4.11 (Non-text Contrast).

Returns an object with `passes` (boolean) and `ratio` (number).

```js
// Depends on: contrastRatio (from contrast/SKILL.md)
function checkBorderContrast(borderColor, backgroundColor) {
  const ratio = contrastRatio(borderColor, backgroundColor);
  return {
    passes: ratio >= 3,
    ratio: Math.round(ratio * 100) / 100,
  };
}
```

### getBorderStyle(token, color)
Returns a CSS border shorthand string for a given width token and color.

```js
function getBorderStyle(token, color) {
  const scale = getBorderScale();
  const width = scale[token];
  if (width === undefined) throw new Error(`Unknown border token: ${token}`);
  if (width === 0) return 'none';
  return `${width}px solid ${color}`;
}
```

## Usage Notes

- **Hairline borders** (0.5px) may render inconsistently across devices and pixel densities. Prefer `thin` (1px) for reliable rendering.
- **Avoid mixing** border widths within a single component family — pick one weight and apply it consistently.
- **Radius consistency**: UI surfaces at the same elevation level should share the same radius token.
- **Pill shapes**: Use `full` token rather than a large magic number like `100px`.
- **Focus rings**: Border-based focus indicators should be at least `base` (2px) wide. See `states/SKILL.md` for `checkFocusIndicator`.

## Audit Checklist

- [ ] All border widths map to a scale token
- [ ] All border radii map to a scale token
- [ ] Decorative borders meet 3:1 contrast ratio (WCAG 1.4.11)
- [ ] Focus-visible borders are ≥ 2px
- [ ] No hairline borders used in critical UI affordances
- [ ] Radius is consistent within component families
