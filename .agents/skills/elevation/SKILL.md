# Elevation Skill

Handles shadow systems, z-index scales, and surface elevation for consistent depth perception in UI.

## Concepts

### Elevation Scale
Elevation uses a numeric scale (0–24) inspired by Material Design, but mapped to a token-based shadow system. Each level corresponds to a specific box-shadow value that communicates depth.

```
Level 0  → no shadow (flat, ground surface)
Level 1  → subtle lift (cards, inputs)
Level 2  → low raise (dropdowns, tooltips)
Level 4  → mid raise (dialogs, drawers)
Level 8  → high raise (modals)
Level 16 → overlay (popovers, floating panels)
Level 24 → maximum (sticky headers, toasts)
```

### Shadow Anatomy
Each elevation level is composed of two shadow layers:
- **Ambient shadow**: large, soft, low-opacity — simulates indirect light
- **Key shadow**: small, crisp, higher-opacity — simulates direct light source

### Z-Index Scale
Z-index values should come from a defined scale to avoid conflicts:

```
0    → default flow
10   → raised content (sticky elements)
100  → dropdowns / tooltips
200  → modals / dialogs
300  → notifications / toasts
400  → dev overlays / debug tools
```

## Functions

### `getElevationScale()`
Returns the full elevation token map.

```js
function getElevationScale() {
  return {
    0:  { key: 'none',                                          ambient: 'none' },
    1:  { key: '0 1px 2px rgba(0,0,0,0.12)',                   ambient: '0 1px 4px rgba(0,0,0,0.06)' },
    2:  { key: '0 2px 4px rgba(0,0,0,0.12)',                   ambient: '0 2px 8px rgba(0,0,0,0.06)' },
    4:  { key: '0 4px 8px rgba(0,0,0,0.12)',                   ambient: '0 4px 16px rgba(0,0,0,0.06)' },
    8:  { key: '0 8px 16px rgba(0,0,0,0.12)',                  ambient: '0 8px 24px rgba(0,0,0,0.06)' },
    16: { key: '0 16px 24px rgba(0,0,0,0.12)',                 ambient: '0 16px 40px rgba(0,0,0,0.06)' },
    24: { key: '0 24px 32px rgba(0,0,0,0.14)',                 ambient: '0 24px 48px rgba(0,0,0,0.08)' },
  };
}
```

### `isOnElevationScale(level)`
Returns `true` if the given level exists in the elevation scale.

```js
function isOnElevationScale(level) {
  const scale = getElevationScale();
  return Object.keys(scale).map(Number).includes(level);
}
```

### `snapToElevationScale(level)`
Snaps an arbitrary level value to the nearest valid elevation scale step.

```js
function snapToElevationScale(level) {
  const steps = Object.keys(getElevationScale()).map(Number);
  return steps.reduce((prev, curr) =>
    Math.abs(curr - level) < Math.abs(prev - level) ? curr : prev
  );
}
```

### `getShadow(level)`
Returns the combined CSS `box-shadow` string for a given elevation level.

```js
function getShadow(level) {
  const scale = getElevationScale();
  const snapped = snapToElevationScale(level);
  const { key, ambient } = scale[snapped];
  if (key === 'none' && ambient === 'none') return 'none';
  return `${key}, ${ambient}`;
}
```

### `getZIndex(layer)`
Returns the z-index value for a named layer.

```js
function getZIndex(layer) {
  const zScale = {
    default:      0,
    raised:       10,
    dropdown:     100,
    modal:        200,
    notification: 300,
    debug:        400,
  };
  if (!(layer in zScale)) {
    throw new Error(`Unknown z-index layer: "${layer}". Valid layers: ${Object.keys(zScale).join(', ')}`);
  }
  return zScale[layer];
}
```

### `auditElevation(styles)`
Audits a styles object for elevation-related issues. Returns an array of findings.

```js
/**
 * @param {Object} styles - CSS-in-JS style object or parsed CSS properties
 * @returns {Array<{ type: 'warning'|'error', message: string }>}
 */
function auditElevation(styles) {
  const findings = [];

  // Check for arbitrary z-index values
  if (styles.zIndex !== undefined) {
    const validZValues = [0, 10, 100, 200, 300, 400];
    if (!validZValues.includes(Number(styles.zIndex))) {
      findings.push({
        type: 'warning',
        message: `z-index ${styles.zIndex} is not on the z-index scale. Consider using a named layer via getZIndex().`,
      });
    }
  }

  // Check for raw box-shadow not from the scale
  if (styles.boxShadow && styles.boxShadow !== 'none') {
    const scaleShadows = Object.keys(getElevationScale())
      .map(Number)
      .filter(l => l > 0)
      .map(l => getShadow(l));
    const isFromScale = scaleShadows.includes(styles.boxShadow);
    if (!isFromScale) {
      findings.push({
        type: 'warning',
        message: `box-shadow value does not match any elevation scale token. Use getShadow(level) to stay consistent.`,
      });
    }
  }

  return findings;
}
```

## Usage Examples

```js
// Get shadow for a card
const cardShadow = getShadow(1);
// → '0 1px 2px rgba(0,0,0,0.12), 0 1px 4px rgba(0,0,0,0.06)'

// Get z-index for a modal
const modalZ = getZIndex('modal'); // → 200

// Snap an off-scale level
const snapped = snapToElevationScale(6); // → 4 or 8 (nearest)

// Audit a component's styles
const issues = auditElevation({ zIndex: 999, boxShadow: '0 3px 6px black' });
// → [{ type: 'warning', message: '...' }, { type: 'warning', message: '...' }]
```

## Design Notes

- Prefer elevation tokens over raw `box-shadow` values in all components
- Never use arbitrary `z-index` values — always go through `getZIndex()`
- Dark mode surfaces often use lightness shifts instead of shadows; elevation level should still be tracked even when shadows are suppressed
- Avoid stacking more than 3 elevation levels in a single view — it reduces the clarity of the depth hierarchy
