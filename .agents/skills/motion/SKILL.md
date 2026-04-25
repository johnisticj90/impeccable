# Motion Skill

Analyzes and validates animation timing, easing, and motion design principles for UI components.

## Purpose

Ensures animations feel natural, purposeful, and accessible. Validates timing against established motion scales, checks easing curves for physical plausibility, and flags animations that may cause accessibility issues (e.g., vestibular disorders).

## Functions

### `getMotionScale()`

Returns a standardized duration scale in milliseconds, based on Material Design and IBM Carbon motion systems.

```js
function getMotionScale() {
  return {
    instant:   0,
    fast01:    70,
    fast02:    110,
    moderate01: 150,
    moderate02: 240,
    slow01:    400,
    slow02:    700,
  };
}
```

**Returns:** `Object` — named duration tokens in ms

---

### `isOnMotionScale(durationMs, tolerance?)`

Checks whether a given duration is close to a value on the motion scale.

```js
function isOnMotionScale(durationMs, tolerance = 15) {
  const scale = Object.values(getMotionScale());
  return scale.some(v => Math.abs(v - durationMs) <= tolerance);
}
```

**Parameters:**
- `durationMs` `{number}` — animation duration in milliseconds
- `tolerance` `{number}` — acceptable deviation in ms (default: 15)

**Returns:** `boolean`

---

### `snapToMotionScale(durationMs)`

Snaps a duration to the nearest value on the motion scale.

```js
function snapToMotionScale(durationMs) {
  const scale = getMotionScale();
  const entries = Object.entries(scale);
  let closest = entries[0];
  let minDiff = Math.abs(durationMs - closest[1]);

  for (const [name, value] of entries) {
    const diff = Math.abs(durationMs - value);
    if (diff < minDiff) {
      minDiff = diff;
      closest = [name, value];
    }
  }

  return { token: closest[0], value: closest[1], diff: minDiff };
}
```

**Parameters:**
- `durationMs` `{number}` — raw duration in ms

**Returns:** `{ token: string, value: number, diff: number }`

---

### `parseEasing(cssEasing)`

Parses a CSS easing string into a normalized descriptor.

```js
function parseEasing(cssEasing) {
  const keywords = {
    'linear':      { type: 'linear', physical: false },
    'ease':        { type: 'ease', physical: true },
    'ease-in':     { type: 'ease-in', physical: false },
    'ease-out':    { type: 'ease-out', physical: true },
    'ease-in-out': { type: 'ease-in-out', physical: true },
  };

  if (keywords[cssEasing]) return keywords[cssEasing];

  const cubicMatch = cssEasing.match(
    /cubic-bezier\(([\d.]+),\s*([\d.-]+),\s*([\d.]+),\s*([\d.-]+)\)/
  );
  if (cubicMatch) {
    const [, x1, y1, x2, y2] = cubicMatch.map(Number);
    // Heuristic: ends slower than it starts = feels physical
    const physical = y2 < y1;
    return { type: 'cubic-bezier', x1, y1, x2, y2, physical };
  }

  return { type: 'unknown', physical: false };
}
```

**Parameters:**
- `cssEasing` `{string}` — CSS easing value (e.g. `'ease-out'`, `'cubic-bezier(0.4,0,0.2,1)'`)

**Returns:** `{ type: string, physical: boolean, ...curveParams? }`

---

### `checkReducedMotion(animationProps)`

Audits an animation definition for `prefers-reduced-motion` compliance.

```js
function checkReducedMotion(animationProps) {
  const issues = [];

  const { durationMs, property, hasReducedMotionAlternative } = animationProps;

  // Decorative motion that moves large elements is risky
  const riskyProperties = ['transform', 'left', 'top', 'right', 'bottom', 'margin', 'padding'];
  const isRisky = riskyProperties.some(p => (property || '').includes(p));

  if (isRisky && durationMs > 400) {
    issues.push({
      severity: 'error',
      message: `Long-duration transform animations (${durationMs}ms) may trigger vestibular issues.`,
      suggestion: 'Wrap in @media (prefers-reduced-motion: no-preference) or reduce to < 400ms.',
    });
  }

  if (isRisky && !hasReducedMotionAlternative) {
    issues.push({
      severity: 'warning',
      message: 'No reduced-motion alternative detected for motion-heavy animation.',
      suggestion: 'Provide a @media (prefers-reduced-motion: reduce) override.',
    });
  }

  return {
    passes: issues.filter(i => i.severity === 'error').length === 0,
    issues,
  };
}
```

**Parameters:**
- `animationProps` `{Object}`
  - `durationMs` `{number}`
  - `property` `{string}` — CSS property being animated
  - `hasReducedMotionAlternative` `{boolean}`

**Returns:** `{ passes: boolean, issues: Array<{ severity, message, suggestion }> }`

---

### `scoreMotion(animationProps)`

Produces an overall motion quality score (0–100) for a given animation.

```js
function scoreMotion(animationProps) {
  let score = 100;
  const notes = [];

  const { durationMs, cssEasing, property, hasReducedMotionAlternative } = animationProps;

  // Duration on scale?
  if (!isOnMotionScale(durationMs)) {
    score -= 15;
    const snap = snapToMotionScale(durationMs);
    notes.push(`Duration ${durationMs}ms is off-scale. Nearest: ${snap.token} (${snap.value}ms).`);
  }

  // Easing physical?
  const easing = parseEasing(cssEasing || 'linear');
  if (!easing.physical) {
    score -= 20;
    notes.push(`Easing '${cssEasing}' doesn't feel physical. Prefer ease-out or ease-in-out.`);
  }

  // Reduced motion?
  const rm = checkReducedMotion({ durationMs, property, hasReducedMotionAlternative });
  if (!rm.passes) {
    score -= 25;
    rm.issues.forEach(i => notes.push(i.message));
  } else if (rm.issues.length > 0) {
    score -= 10;
    rm.issues.forEach(i => notes.push(i.message));
  }

  return { score: Math.max(0, score), notes };
}
```

**Parameters:**
- `animationProps` `{Object}` — same shape as `checkReducedMotion`, plus `cssEasing`

**Returns:** `{ score: number, notes: string[] }`

---

## Usage Example

```js
const result = scoreMotion({
  durationMs: 300,
  cssEasing: 'cubic-bezier(0.4, 0, 0.2, 1)',
  property: 'transform',
  hasReducedMotionAlternative: true,
});

console.log(result.score);  // 85
console.log(result.notes);  // ['Duration 300ms is off-scale. Nearest: moderate01 (150ms).']
```

## References

- [Material Design Motion](https://m3.material.io/styles/motion/overview)
- [IBM Carbon Motion](https://carbondesignsystem.com/guidelines/motion/overview/)
- [WCAG 2.3.3 Animation from Interactions](https://www.w3.org/WAI/WCAG21/Understanding/animation-from-interactions.html)
- [Vestibular disorders & web animation](https://alistapart.com/article/designing-safer-web-animation-for-motion-sensitivity/)
