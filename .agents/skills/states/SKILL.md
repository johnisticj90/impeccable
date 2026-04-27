# States Skill

Handles interactive and component states — hover, focus, active, disabled, loading, error, success — with consistent visual feedback patterns.

## Concepts

### State Layers
Material Design-inspired state layer model. Each interaction adds a semi-transparent overlay at a defined opacity:

| State    | Opacity |
|----------|---------|
| hover    | 0.08    |
| focus    | 0.12    |
| active   | 0.16    |
| dragged  | 0.16    |
| selected | 0.08    |
| disabled | 0.38    |

### State Tokens
States map to design tokens rather than hard-coded values, so they adapt across themes.

### Focus Visibility
Focus styles must be visible for keyboard users. WCAG 2.4.11 (AA) requires focus indicators with:
- Minimum area: perimeter of unfocused component × 1px
- Contrast ratio ≥ 3:1 between focused and unfocused states

## Functions

### `getStateLayer(state, baseColor)`
Returns an rgba color string representing the state overlay.

```js
function getStateLayer(state, baseColor) {
  const opacities = {
    hover: 0.08,
    focus: 0.12,
    active: 0.16,
    dragged: 0.16,
    selected: 0.08,
    disabled: 0.38,
  };

  const opacity = opacities[state];
  if (opacity === undefined) return null;

  // baseColor expected as [r, g, b] 0-255
  const [r, g, b] = baseColor;
  return `rgba(${r}, ${g}, ${b}, ${opacity})`;
}
```

**Examples:**
```js
getStateLayer('hover', [0, 0, 0])     // 'rgba(0, 0, 0, 0.08)'
getStateLayer('focus', [98, 0, 238])  // 'rgba(98, 0, 238, 0.12)'
getStateLayer('disabled', [0, 0, 0]) // 'rgba(0, 0, 0, 0.38)'
```

---

### `isValidState(state)`
Returns true if the given state string is a recognised interactive state.

```js
function isValidState(state) {
  const validStates = [
    'default', 'hover', 'focus', 'active',
    'visited', 'disabled', 'loading', 'error',
    'success', 'selected', 'dragged', 'checked',
    'indeterminate', 'readonly',
  ];
  return validStates.includes(state);
}
```

---

### `checkFocusIndicator(indicator)`
Audits a focus indicator definition for WCAG 2.4.11 compliance.

`indicator` shape:
```ts
{
  outlineWidth: number,      // px
  outlineOffset: number,     // px
  outlineColor: string,      // hex
  backgroundColor: string,   // hex — surface behind the outline
}
```

Returns an audit result:
```ts
{
  passes: boolean,
  contrastRatio: number,
  minimumContrast: 3,
  outlineWidth: number,
  issues: string[],
}
```

```js
// Uses contrastRatio() from contrast skill
function checkFocusIndicator(indicator) {
  const issues = [];
  const ratio = contrastRatio(indicator.outlineColor, indicator.backgroundColor);

  if (ratio < 3) {
    issues.push(
      `Focus outline contrast is ${ratio.toFixed(2)}:1 — must be at least 3:1 (WCAG 2.4.11)`
    );
  }

  if (indicator.outlineWidth < 2) {
    issues.push(
      `Outline width is ${indicator.outlineWidth}px — recommend at least 2px for visibility`
    );
  }

  return {
    passes: issues.length === 0,
    contrastRatio: ratio,
    minimumContrast: 3,
    outlineWidth: indicator.outlineWidth,
    issues,
  };
}
```

---

### `getDisabledStyles()`
Returns a consistent set of CSS-in-JS style properties for disabled components.

```js
function getDisabledStyles() {
  return {
    opacity: 0.38,
    cursor: 'not-allowed',
    pointerEvents: 'none',
    userSelect: 'none',
  };
}
```

---

### `getLoadingStyles()`
Returns style properties appropriate for a loading state — keeps layout stable, signals activity.

```js
function getLoadingStyles() {
  return {
    cursor: 'wait',
    pointerEvents: 'none',
    // Caller should layer in a spinner or skeleton; opacity kept at 1
    // so layout does not shift.
  };
}
```

---

### `describeState(state)`
Returns a human-readable description of what a state means visually and semantically. Useful for documentation generation.

```js
function describeState(state) {
  const descriptions = {
    default:       'Resting state — no interaction in progress.',
    hover:         'Pointer is over the component. Subtle background lift.',
    focus:         'Component has keyboard focus. Visible outline required.',
    active:        'Component is being pressed or clicked.',
    visited:       'Link has been visited by the user.',
    disabled:      'Component is unavailable. Reduced opacity, no pointer events.',
    loading:       'Async operation in progress. Cursor is wait, interactions blocked.',
    error:         'Validation or system error. Use error color tokens.',
    success:       'Operation completed successfully. Use success color tokens.',
    selected:      'Item is chosen within a group (e.g. tab, chip).',
    dragged:       'Item is being dragged. Elevated shadow, state layer applied.',
    checked:       'Boolean control (checkbox, toggle) is on.',
    indeterminate: 'Boolean control is in a mixed/partial state.',
    readonly:      'Value is visible but not editable.',
  };

  return descriptions[state] ?? `Unknown state: "${state}"`;
}
```

## Usage Notes

- Always pair `disabled` state with `aria-disabled="true"` — do not rely on CSS alone.
- `loading` state should be announced via `aria-busy="true"` on the container.
- `error` and `success` states should include an `aria-live` region for screen readers.
- Never remove focus outlines without providing an equivalent custom focus indicator.
- State layers stack — a selected+hover item gets both the selected (0.08) and hover (0.08) layers, totalling 0.16 opacity.

## Related Skills

- **contrast** — `contrastRatio()` is used by `checkFocusIndicator()`
- **motion** — transitions between states should use motion scale tokens
- **colorize** — error/success/warning state colors come from semantic color tokens
- **elevation** — `dragged` state typically increases elevation level
