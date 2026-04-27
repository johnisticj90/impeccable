# Breakpoints Skill

Handles responsive breakpoint detection, validation, and media query generation consistent with a design token system.

## Responsibilities

- Define and validate breakpoint scales
- Generate media query strings
- Detect current breakpoint from viewport width
- Snap arbitrary values to the nearest breakpoint
- Check if a value falls on the breakpoint scale

## Breakpoint Scale

Default breakpoints follow a mobile-first approach:

| Name | Min Width | Use Case |
|------|-----------|----------|
| `xs` | 0px | Mobile portrait |
| `sm` | 480px | Mobile landscape |
| `md` | 768px | Tablet |
| `lg` | 1024px | Desktop |
| `xl` | 1280px | Wide desktop |
| `2xl` | 1536px | Ultra-wide |

## Functions

### `getBreakpointScale(config?)`

Returns the full breakpoint scale, optionally merged with custom config.

```js
getBreakpointScale()
// => { xs: 0, sm: 480, md: 768, lg: 1024, xl: 1280, '2xl': 1536 }

getBreakpointScale({ custom: 900 })
// => { xs: 0, sm: 480, md: 768, custom: 900, lg: 1024, xl: 1280, '2xl': 1536 }
```

### `isOnBreakpointScale(value, config?)`

Returns `true` if the value (in px) matches a defined breakpoint.

```js
isOnBreakpointScale(768)  // => true
isOnBreakpointScale(800)  // => false
```

### `snapToBreakpoint(value, config?)`

Snaps an arbitrary px value to the nearest breakpoint.

```js
snapToBreakpoint(820)  // => { name: 'md', value: 768 }
snapToBreakpoint(1100) // => { name: 'lg', value: 1024 }
```

### `getMediaQuery(breakpoint, direction?, config?)`

Generates a CSS media query string for a given breakpoint name.

- `direction`: `'up'` (min-width, default) | `'down'` (max-width) | `'only'` (range between current and next)

```js
getMediaQuery('md')          // => '@media (min-width: 768px)'
getMediaQuery('md', 'down')  // => '@media (max-width: 767px)'
getMediaQuery('md', 'only')  // => '@media (min-width: 768px) and (max-width: 1023px)'
```

### `detectBreakpoint(viewportWidth, config?)`

Returns the active breakpoint name for a given viewport width.

```js
detectBreakpoint(900)  // => 'md'
detectBreakpoint(480)  // => 'sm'
detectBreakpoint(1600) // => 'xl'
```

### `getBreakpointRange(from, to, config?)`

Returns a media query string covering a range between two named breakpoints.

```js
getBreakpointRange('sm', 'lg')
// => '@media (min-width: 480px) and (max-width: 1023px)'
```

## Implementation

```js
const DEFAULT_BREAKPOINTS = {
  xs: 0,
  sm: 480,
  md: 768,
  lg: 1024,
  xl: 1280,
  '2xl': 1536,
};

function getBreakpointScale(config = {}) {
  const merged = { ...DEFAULT_BREAKPOINTS, ...config };
  // Sort by value ascending
  return Object.fromEntries(
    Object.entries(merged).sort(([, a], [, b]) => a - b)
  );
}

function isOnBreakpointScale(value, config = {}) {
  const scale = getBreakpointScale(config);
  return Object.values(scale).includes(value);
}

function snapToBreakpoint(value, config = {}) {
  const scale = getBreakpointScale(config);
  const entries = Object.entries(scale);
  let closest = entries[0];
  let minDiff = Math.abs(value - entries[0][1]);

  for (const [name, bp] of entries) {
    const diff = Math.abs(value - bp);
    if (diff < minDiff) {
      minDiff = diff;
      closest = [name, bp];
    }
  }

  return { name: closest[0], value: closest[1] };
}

function getMediaQuery(breakpoint, direction = 'up', config = {}) {
  const scale = getBreakpointScale(config);
  const entries = Object.entries(scale);
  const idx = entries.findIndex(([name]) => name === breakpoint);

  if (idx === -1) throw new Error(`Unknown breakpoint: "${breakpoint}"`);

  const [, minPx] = entries[idx];

  if (direction === 'up') {
    return `@media (min-width: ${minPx}px)`;
  }

  if (direction === 'down') {
    return `@media (max-width: ${minPx - 1}px)`;
  }

  if (direction === 'only') {
    const next = entries[idx + 1];
    if (!next) return `@media (min-width: ${minPx}px)`;
    return `@media (min-width: ${minPx}px) and (max-width: ${next[1] - 1}px)`;
  }

  throw new Error(`Unknown direction: "${direction}". Use 'up', 'down', or 'only'.`);
}

function detectBreakpoint(viewportWidth, config = {}) {
  const scale = getBreakpointScale(config);
  const entries = Object.entries(scale);
  let active = entries[0][0];

  for (const [name, minPx] of entries) {
    if (viewportWidth >= minPx) active = name;
  }

  return active;
}

function getBreakpointRange(from, to, config = {}) {
  const scale = getBreakpointScale(config);
  const entries = Object.entries(scale);

  const fromIdx = entries.findIndex(([name]) => name === from);
  const toIdx = entries.findIndex(([name]) => name === to);

  if (fromIdx === -1) throw new Error(`Unknown breakpoint: "${from}"`);
  if (toIdx === -1) throw new Error(`Unknown breakpoint: "${to}"`);
  if (fromIdx >= toIdx) throw new Error(`"${from}" must be smaller than "${to}"`);

  const minPx = entries[fromIdx][1];
  const nextAfterTo = entries[toIdx + 1];
  const maxPx = nextAfterTo ? nextAfterTo[1] - 1 : null;

  if (!maxPx) return `@media (min-width: ${minPx}px)`;
  return `@media (min-width: ${minPx}px) and (max-width: ${maxPx}px)`;
}
```

## Usage Notes

- All breakpoint values are in **pixels**.
- `'down'` queries subtract 1px to avoid overlap with the next breakpoint's `'up'` query.
- Custom breakpoints passed via `config` are merged and re-sorted by value, so ordering is always consistent.
- When using `detectBreakpoint` server-side, pass a known viewport width (e.g. from a user-agent hint or cookie).

## Related Skills

- **density** — density scale can vary per breakpoint
- **spacing** — spacing may be adjusted at different breakpoints
- **typography** — type scale may shift at breakpoints
- **layout** — grid systems are inherently breakpoint-aware
