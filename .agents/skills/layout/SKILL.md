# Layout Skill

Analyzes and improves layout decisions including grid systems, alignment, whitespace distribution, and visual hierarchy.

## Purpose

Helps ensure layouts are consistent, balanced, and follow established design principles. Works alongside the spacing skill for precise measurements and the typography skill for text-based layout concerns.

## Functions

### `detectGridSystem(elements)`

Infers the underlying grid system from a set of element positions and widths.

```javascript
/**
 * Attempt to detect the column count and gutter size of an implicit grid
 * from a list of laid-out elements.
 *
 * @param {Array<{x: number, width: number, containerWidth: number}>} elements
 * @returns {{ columns: number, gutter: number, confidence: number }}
 */
function detectGridSystem(elements) {
  if (!elements.length) return { columns: 12, gutter: 16, confidence: 0 };

  const containerWidth = elements[0].containerWidth;
  const candidateColumns = [4, 6, 8, 12, 16, 24];
  let best = { columns: 12, gutter: 16, confidence: 0 };

  for (const cols of candidateColumns) {
    const colWidth = containerWidth / cols;
    let totalError = 0;

    for (const el of elements) {
      // How many columns does this element span (nearest integer)?
      const spanRaw = el.width / colWidth;
      const spanRounded = Math.round(spanRaw);
      const error = Math.abs(spanRaw - spanRounded) / spanRounded;
      totalError += error;
    }

    const avgError = totalError / elements.length;
    const confidence = Math.max(0, 1 - avgError * 4);

    if (confidence > best.confidence) {
      // Estimate gutter from leftover space
      const totalSpan = elements.reduce((sum, el) => {
        return sum + Math.round(el.width / colWidth);
      }, 0);
      const gutterCount = Math.max(1, totalSpan - 1);
      const usedWidth = elements.reduce((sum, el) => sum + el.width, 0);
      const gutter = Math.round((containerWidth - usedWidth) / gutterCount);

      best = { columns: cols, gutter: Math.max(0, gutter), confidence };
    }
  }

  return best;
}
```

### `checkAlignment(elements)`

Checks whether elements are aligned to a consistent set of axes.

```javascript
/**
 * Identify misaligned elements by looking for near-matches on x/y axes
 * that fall just outside a tolerance threshold.
 *
 * @param {Array<{id: string, x: number, y: number, width: number, height: number}>} elements
 * @param {number} [tolerance=2] - Pixel tolerance for alignment snapping
 * @returns {Array<{ id: string, axis: 'x'|'y', offset: number, nearestAxis: number }>}
 */
function checkAlignment(elements, tolerance = 2) {
  const xAxes = [...new Set(elements.map((el) => el.x))];
  const yAxes = [...new Set(elements.map((el) => el.y))];
  const misaligned = [];

  for (const el of elements) {
    const nearestX = xAxes.reduce((prev, curr) =>
      Math.abs(curr - el.x) < Math.abs(prev - el.x) ? curr : prev
    );
    const nearestY = yAxes.reduce((prev, curr) =>
      Math.abs(curr - el.y) < Math.abs(prev - el.y) ? curr : prev
    );

    const xOffset = Math.abs(el.x - nearestX);
    const yOffset = Math.abs(el.y - nearestY);

    // Only flag if it's close but not exact — exact matches are intentional
    if (xOffset > 0 && xOffset <= tolerance * 3 && xOffset > tolerance) {
      misaligned.push({ id: el.id, axis: 'x', offset: xOffset, nearestAxis: nearestX });
    }
    if (yOffset > 0 && yOffset <= tolerance * 3 && yOffset > tolerance) {
      misaligned.push({ id: el.id, axis: 'y', offset: yOffset, nearestAxis: nearestY });
    }
  }

  return misaligned;
}
```

### `scoreVisualBalance(elements, containerWidth, containerHeight)`

Scores how visually balanced a layout is by comparing the visual weight distribution across quadrants.

```javascript
/**
 * Compute a balance score (0–1) for a layout by comparing visual weight
 * across the four quadrants of the container.
 *
 * Visual weight is approximated by element area.
 *
 * @param {Array<{x: number, y: number, width: number, height: number}>} elements
 * @param {number} containerWidth
 * @param {number} containerHeight
 * @returns {number} Score from 0 (unbalanced) to 1 (perfectly balanced)
 */
function scoreVisualBalance(elements, containerWidth, containerHeight) {
  const cx = containerWidth / 2;
  const cy = containerHeight / 2;

  const quadrants = { topLeft: 0, topRight: 0, bottomLeft: 0, bottomRight: 0 };

  for (const el of elements) {
    const area = el.width * el.height;
    const elCx = el.x + el.width / 2;
    const elCy = el.y + el.height / 2;

    if (elCx <= cx && elCy <= cy) quadrants.topLeft += area;
    else if (elCx > cx && elCy <= cy) quadrants.topRight += area;
    else if (elCx <= cx && elCy > cy) quadrants.bottomLeft += area;
    else quadrants.bottomRight += area;
  }

  const values = Object.values(quadrants);
  const total = values.reduce((a, b) => a + b, 0);
  if (total === 0) return 1;

  const normalized = values.map((v) => v / total);
  const ideal = 0.25;
  const maxDeviation = normalized.reduce((sum, v) => sum + Math.abs(v - ideal), 0);

  // Maximum possible deviation is 0.75 (all weight in one quadrant)
  return Math.max(0, 1 - maxDeviation / 0.75);
}
```

## Usage Guidelines

- Use `detectGridSystem` to verify that a design is following an established grid before suggesting layout changes.
- Use `checkAlignment` to surface subtle misalignments that are easy to miss visually but noticeable subconsciously.
- Use `scoreVisualBalance` as a quick sanity check — scores below 0.6 typically indicate a layout that will feel "heavy" on one side.
- Combine with the **spacing** skill to ensure gutters and margins are on the spacing scale.
- Combine with the **contrast** skill when evaluating whether background regions provide sufficient separation between layout sections.

## Related Skills

- `spacing` — for snapping measurements to scale
- `typography` — for text-driven layout decisions
- `contrast` — for section separation and layering
