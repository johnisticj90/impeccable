# Skill: Design Tokens

Manage, validate, and transform design tokens across formats and scales.

## Overview

Design tokens are the atomic values that define a design system — colors, spacing, typography, motion, elevation, and more. This skill provides utilities to parse, validate, transform, and export tokens in a consistent, interoperable format.

## Core Concepts

### Token Categories
- **Primitive tokens**: Raw values (e.g., `blue-500: #3B82F6`)
- **Semantic tokens**: Contextual aliases (e.g., `color-action: {blue-500}`)
- **Component tokens**: Component-scoped overrides (e.g., `button-bg: {color-action}`)

### Token Types
- `color` — hex, rgb, hsl
- `dimension` — px, rem, em
- `fontFamily` — string
- `fontWeight` — number or keyword
- `duration` — ms, s
- `cubicBezier` — easing function
- `number` — unitless
- `shadow` — box-shadow shorthand

## Functions

### `parseToken(raw)`
Parse a raw token value and detect its type.

```js
/**
 * Parse a raw token value and infer its type.
 * @param {string|number} raw - The raw token value
 * @returns {{ value: string|number, type: string, unit?: string }}
 */
function parseToken(raw) {
  const str = String(raw).trim();

  if (/^#([0-9a-fA-F]{3,8})$/.test(str)) {
    return { value: str, type: 'color' };
  }
  if (/^rgba?\(/.test(str) || /^hsla?\(/.test(str)) {
    return { value: str, type: 'color' };
  }
  if (/^-?[\d.]+px$/.test(str)) {
    return { value: parseFloat(str), type: 'dimension', unit: 'px' };
  }
  if (/^-?[\d.]+rem$/.test(str)) {
    return { value: parseFloat(str), type: 'dimension', unit: 'rem' };
  }
  if (/^-?[\d.]+em$/.test(str)) {
    return { value: parseFloat(str), type: 'dimension', unit: 'em' };
  }
  if (/^[\d.]+ms$/.test(str)) {
    return { value: parseFloat(str), type: 'duration', unit: 'ms' };
  }
  if (/^[\d.]+s$/.test(str)) {
    return { value: parseFloat(str) * 1000, type: 'duration', unit: 'ms' };
  }
  if (/^cubic-bezier\(/.test(str)) {
    return { value: str, type: 'cubicBezier' };
  }
  if (typeof raw === 'number' || /^-?[\d.]+$/.test(str)) {
    return { value: Number(raw), type: 'number' };
  }
  return { value: str, type: 'string' };
}
```

### `resolveAlias(token, tokens)`
Resolve a token alias reference like `{color.blue.500}` to its final value.

```js
/**
 * Resolve an alias reference to its concrete value.
 * @param {string} token - Token value, possibly an alias like "{color.blue.500}"
 * @param {object} tokens - Flat or nested token map
 * @returns {string|number|null} Resolved value or null if unresolvable
 */
function resolveAlias(token, tokens) {
  const aliasPattern = /^\{([^}]+)\}$/;
  const match = String(token).match(aliasPattern);
  if (!match) return token;

  const path = match[1].split('.');
  let current = tokens;
  for (const key of path) {
    if (current == null || typeof current !== 'object') return null;
    current = current[key];
  }

  // Unwrap W3C token format
  if (current && typeof current === 'object' && '$value' in current) {
    return resolveAlias(current.$value, tokens);
  }

  return current != null ? resolveAlias(current, tokens) : null;
}
```

### `flattenTokens(tokens, prefix)`
Flatten a nested token object into a flat key-value map.

```js
/**
 * Flatten a nested token tree into dot-separated keys.
 * @param {object} tokens - Nested token object
 * @param {string} [prefix=''] - Key prefix for recursion
 * @returns {Record<string, any>} Flat token map
 */
function flattenTokens(tokens, prefix = '') {
  const result = {};
  for (const [key, value] of Object.entries(tokens)) {
    const fullKey = prefix ? `${prefix}.${key}` : key;
    if (value && typeof value === 'object' && '$value' in value) {
      result[fullKey] = value.$value;
    } else if (value && typeof value === 'object' && !Array.isArray(value)) {
      Object.assign(result, flattenTokens(value, fullKey));
    } else {
      result[fullKey] = value;
    }
  }
  return result;
}
```

### `tokenToCssVar(key, value)`
Convert a token key/value pair to a CSS custom property declaration.

```js
/**
 * Convert a token to a CSS custom property string.
 * @param {string} key - Dot-separated token key (e.g., "color.blue.500")
 * @param {string|number} value - Resolved token value
 * @returns {string} CSS variable declaration (e.g., "--color-blue-500: #3B82F6;")
 */
function tokenToCssVar(key, value) {
  const varName = '--' + key.replace(/\./g, '-');
  return `${varName}: ${value};`;
}
```

### `exportToCss(tokens)`
Export a full token set as a CSS `:root` block.

```js
/**
 * Export a token set as a CSS :root block.
 * @param {object} tokens - Nested or flat token object
 * @returns {string} Full CSS string with :root block
 */
function exportToCss(tokens) {
  const flat = flattenTokens(tokens);
  const lines = Object.entries(flat).map(([key, value]) =>
    '  ' + tokenToCssVar(key, value)
  );
  return `:root {\n${lines.join('\n')}\n}`;
}
```

### `validateToken(key, raw)`
Validate a token value and return any issues.

```js
/**
 * Validate a single token value.
 * @param {string} key - Token key for error messages
 * @param {string|number} raw - Raw token value
 * @returns {{ valid: boolean, issues: string[] }}
 */
function validateToken(key, raw) {
  const issues = [];
  const { type, value } = parseToken(raw);

  if (type === 'color') {
    // Basic hex length check
    if (/^#/.test(String(raw)) && ![4, 5, 7, 9].includes(String(raw).length)) {
      issues.push(`"${key}": invalid hex color length`);
    }
  }

  if (type === 'dimension' && value === 0 && String(raw) !== '0') {
    issues.push(`"${key}": zero dimension should be bare 0 without unit`);
  }

  if (type === 'string' && String(raw).startsWith('{')) {
    issues.push(`"${key}": unresolved alias reference "${raw}"`);
  }

  return { valid: issues.length === 0, issues };
}
```

## Usage Examples

```js
// Parse a token value
parseToken('#3B82F6');     // { value: '#3B82F6', type: 'color' }
parseToken('1.5rem');      // { value: 1.5, type: 'dimension', unit: 'rem' }
parseToken('200ms');       // { value: 200, type: 'duration', unit: 'ms' }

// Resolve aliases
const tokens = { color: { blue: { 500: '#3B82F6' } } };
resolveAlias('{color.blue.500}', tokens); // '#3B82F6'

// Export to CSS
const myTokens = { color: { primary: '#3B82F6' }, spacing: { md: '1rem' } };
exportToCss(myTokens);
// :root {
//   --color-primary: #3B82F6;
//   --spacing-md: 1rem;
// }
```

## Integration Notes

- Tokens should follow the [W3C Design Token Community Group](https://design-tokens.github.io/community-group/format/) format where possible, using `$value` and `$type` keys.
- Use `resolveAlias` before passing values to contrast, spacing, or typography skill functions.
- `flattenTokens` output can be diffed across versions to detect breaking token changes.
- Pair with the `colorize` skill to validate color tokens against palette rules.
