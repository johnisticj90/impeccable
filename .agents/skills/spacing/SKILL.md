# Spacing Skill

Analyzes and improves spacing, padding, margin, and layout rhythm in CSS/UI code.

## Overview

The `spacing` skill reviews spacing decisions in stylesheets and component code, ensuring consistent visual rhythm, appropriate whitespace usage, and adherence to a spacing scale.

## Usage

```
skill: spacing
target: <file or component>
options:
  scale: [4, 8, 16, 24, 32, 48, 64]  # base spacing scale in px
  unit: px | rem | em
  strict: true | false
```

## What It Checks

### 1. Spacing Scale Consistency
Verifies that spacing values align to a defined scale (e.g., multiples of 4 or 8).

**Violation example:**
```css
.card {
  padding: 13px 7px; /* arbitrary values */
  margin-bottom: 22px;
}
```

**Fixed:**
```css
.card {
  padding: 12px 8px; /* aligned to 4px scale */
  margin-bottom: 24px;
}
```

### 2. Magic Numbers
Flags hardcoded spacing values that don't map to a token or variable.

**Violation:**
```css
.header {
  margin-top: 37px;
}
```

**Fixed:**
```css
:root {
  --space-9: 36px;
}

.header {
  margin-top: var(--space-9);
}
```

### 3. Collapsed Margins
Detects potential margin collapse issues between block elements.

**Violation:**
```css
.section {
  margin-bottom: 32px;
}
.section + .section {
  margin-top: 32px; /* will collapse with above */
}
```

**Fixed:**
```css
.section + .section {
  margin-top: 0; /* rely on previous element's bottom margin */
}
```

### 4. Padding vs Margin Semantics
Checks that padding is used for internal spacing and margin for external spacing.

**Violation:**
```css
.button {
  margin: 0 12px; /* internal spacing should be padding */
}
```

**Fixed:**
```css
.button {
  padding: 0 12px;
}
```

### 5. Responsive Spacing
Ensures spacing scales appropriately across breakpoints.

**Violation:**
```css
.container {
  padding: 48px; /* same on mobile and desktop */
}
```

**Fixed:**
```css
.container {
  padding: 16px;
}

@media (min-width: 768px) {
  .container {
    padding: 32px;
  }
}

@media (min-width: 1200px) {
  .container {
    padding: 48px;
  }
}
```

## Spacing Scale Reference

| Token        | px  | rem    |
|--------------|-----|--------|
| `--space-1`  | 4   | 0.25   |
| `--space-2`  | 8   | 0.5    |
| `--space-3`  | 12  | 0.75   |
| `--space-4`  | 16  | 1      |
| `--space-5`  | 20  | 1.25   |
| `--space-6`  | 24  | 1.5    |
| `--space-8`  | 32  | 2      |
| `--space-10` | 40  | 2.5    |
| `--space-12` | 48  | 3      |
| `--space-16` | 64  | 4      |
| `--space-20` | 80  | 5      |
| `--space-24` | 96  | 6      |

## Helper Functions

```javascript
/**
 * Checks if a pixel value aligns to a given spacing scale.
 * @param {number} value - The pixel value to check.
 * @param {number} base - The base unit of the scale (default: 4).
 * @returns {boolean} True if the value is on-scale.
 */
function isOnScale(value, base = 4) {
  return value % base === 0;
}

/**
 * Finds the nearest on-scale value to a given pixel value.
 * @param {number} value - The pixel value to snap.
 * @param {number} base - The base unit of the scale (default: 4).
 * @returns {number} The nearest on-scale value.
 */
function snapToScale(value, base = 4) {
  return Math.round(value / base) * base;
}

/**
 * Converts a px spacing value to its rem equivalent.
 * @param {number} px - Pixel value.
 * @param {number} rootFontSize - Root font size in px (default: 16).
 * @returns {string} Rem value as a CSS string.
 */
function pxToRem(px, rootFontSize = 16) {
  return `${px / rootFontSize}rem`;
}
```

## Output Format

The skill outputs a structured report:

```json
{
  "file": "src/styles/components.css",
  "violations": [
    {
      "line": 14,
      "rule": "off-scale-value",
      "severity": "warning",
      "value": "13px",
      "suggestion": "12px"
    }
  ],
  "summary": {
    "total": 3,
    "errors": 0,
    "warnings": 3
  }
}
```

## Integration

This skill pairs well with:
- `colorize` — for full design token audits
- `audit` — for broader CSS quality checks
- `adapt` — when adjusting spacing for different screen sizes
