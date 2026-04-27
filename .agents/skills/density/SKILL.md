# Density Skill

Handles UI density scaling — the relationship between component sizing, spacing, and information density across compact, default, and comfortable modes.

## Concepts

**Density** controls how much information fits on screen and how much breathing room elements have. Three tiers:

- **Compact** (-1): Tighter spacing, smaller touch targets, more information visible. Good for data-heavy UIs, power users.
- **Default** (0): Balanced. The baseline everything else is measured against.
- **Comfortable** (+1): More padding, larger targets, less cognitive load. Good for consumer apps, accessibility.

Density multipliers apply to spacing and sizing tokens, not to typography (type scale stays fixed across densities).

## Scale

```
Density Level | Multiplier | Use Case
-1 (compact)  |    0.75    | dashboards, data tables, dev tools
 0 (default)  |    1.00    | general purpose
+1 (comfort)  |    1.25    | consumer, a11y-first, touch-primary
```

Component-specific density adjustments:

```
Component     | Compact padding | Default padding | Comfortable padding
Button        | 4px 8px         | 8px 16px        | 12px 24px
Input         | 6px 8px         | 8px 12px        | 12px 16px
List item     | 4px 16px        | 8px 16px        | 12px 16px
Table row     | 4px 8px         | 8px 8px         | 12px 8px
Card          | 12px            | 16px            | 24px
```

## Functions

### getDensityScale()

Returns the full density scale definition.

```js
function getDensityScale() {
  return {
    compact:     { level: -1, multiplier: 0.75, label: 'Compact' },
    default:     { level:  0, multiplier: 1.00, label: 'Default' },
    comfortable: { level: +1, multiplier: 1.25, label: 'Comfortable' },
  };
}
```

### getDensityMultiplier(density)

Returns the numeric multiplier for a given density key or level.

```js
function getDensityMultiplier(density) {
  const scale = getDensityScale();

  // Accept string key
  if (typeof density === 'string') {
    const entry = scale[density];
    if (!entry) throw new Error(`Unknown density: "${density}". Use compact, default, or comfortable.`);
    return entry.multiplier;
  }

  // Accept numeric level (-1, 0, 1)
  if (typeof density === 'number') {
    const entry = Object.values(scale).find(d => d.level === density);
    if (!entry) throw new Error(`Unknown density level: ${density}. Use -1, 0, or 1.`);
    return entry.multiplier;
  }

  throw new TypeError('density must be a string key or numeric level');
}
```

### applyDensity(value, density)

Scales a spacing or sizing value by the density multiplier. Returns a rounded pixel value.

```js
function applyDensity(value, density) {
  const multiplier = getDensityMultiplier(density);
  // Round to nearest 2px to stay on grid
  return Math.round((value * multiplier) / 2) * 2;
}
```

**Example:**
```js
applyDensity(16, 'compact')     // → 12
applyDensity(16, 'default')     // → 16
applyDensity(16, 'comfortable') // → 20
```

### getComponentDensity(component, density)

Returns padding values for a known component at the given density.

```js
function getComponentDensity(component, density) {
  const components = {
    button:   { compact: '4px 8px',   default: '8px 16px',  comfortable: '12px 24px' },
    input:    { compact: '6px 8px',   default: '8px 12px',  comfortable: '12px 16px' },
    listItem: { compact: '4px 16px',  default: '8px 16px',  comfortable: '12px 16px' },
    tableRow: { compact: '4px 8px',   default: '8px 8px',   comfortable: '12px 8px'  },
    card:     { compact: '12px',      default: '16px',       comfortable: '24px'      },
  };

  const key = typeof density === 'number'
    ? Object.values(getDensityScale()).find(d => d.level === density)?.label.toLowerCase()
    : density;

  if (!components[component]) {
    throw new Error(`Unknown component: "${component}". Known: ${Object.keys(components).join(', ')}`);
  }

  if (!components[component][key]) {
    throw new Error(`Unknown density: "${key}" for component "${component}".`);
  }

  return components[component][key];
}
```

### checkMinTouchTarget(size, density)

Checks whether a touch target meets the minimum size requirement at the given density. Compact mode relaxes the minimum slightly for pointer-primary contexts.

```js
function checkMinTouchTarget(size, density) {
  // WCAG 2.5.5 recommends 44x44px. Compact allows 32px for pointer-primary.
  const minimums = {
    compact:     32,
    default:     44,
    comfortable: 44,
  };

  const key = typeof density === 'number'
    ? Object.values(getDensityScale()).find(d => d.level === density)?.label.toLowerCase()
    : density;

  const min = minimums[key];
  if (min === undefined) throw new Error(`Unknown density: "${key}"`);

  return {
    passes: size >= min,
    size,
    minimum: min,
    delta: size - min,
  };
}
```

## Usage Examples

```js
// Scale a spacing value
const padding = applyDensity(16, 'compact'); // 12

// Get button padding for comfortable mode
const btnPadding = getComponentDensity('button', 'comfortable'); // '12px 24px'

// Check touch target in compact mode
const result = checkMinTouchTarget(36, 'compact');
// { passes: true, size: 36, minimum: 32, delta: 4 }

// Get multiplier by level
const m = getDensityMultiplier(-1); // 0.75
```

## Notes

- Typography does **not** change with density. Only spacing and sizing tokens scale.
- Icon sizes follow their own scale (see iconography skill) but touch targets respect density minimums.
- When density is `compact`, always flag if touch targets drop below 32px — compact is for pointer devices, not touch.
- Density multipliers compose with spacing scale: first snap to spacing scale, then apply density multiplier.
