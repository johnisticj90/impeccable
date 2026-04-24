# Document Skill

Generates, improves, or restructures documentation for code, APIs, and projects.

## Purpose

The `document` skill helps create clear, accurate, and maintainable documentation. It can produce inline comments, JSDoc annotations, README files, API references, and architectural overviews — adapting tone and depth to the intended audience.

## When to Use

- Code lacks inline comments or docstrings
- A function/module has no JSDoc or type annotations
- A README is missing, outdated, or incomplete
- An API has no reference documentation
- Onboarding new contributors requires clearer context
- A complex algorithm needs explanation

## Inputs

| Input | Description |
|-------|-------------|
| `target` | File, function, module, or project to document |
| `audience` | Who will read the docs (e.g., `contributor`, `end-user`, `api-consumer`) |
| `format` | Output format: `jsdoc`, `markdown`, `inline`, `readme`, `changelog` |
| `depth` | Level of detail: `brief`, `standard`, `comprehensive` |
| `existing` | Any existing documentation to improve or extend |

## Outputs

- Inline comments added to source code
- JSDoc blocks for functions, classes, and modules
- Markdown documentation files
- Updated README sections
- API reference tables

## Process

### 1. Understand the Subject

Before writing, fully understand what is being documented:

- Read the source code carefully
- Identify inputs, outputs, side effects, and edge cases
- Note dependencies and assumptions
- Understand the intended use case

Do not document what you don't understand. Ask for clarification first.

### 2. Identify the Audience

Documentation written for a library consumer differs from docs written for a core contributor:

- **End users / consumers**: Focus on *what* and *how*. Avoid implementation details. Prioritize examples.
- **Contributors / maintainers**: Include *why* decisions were made. Explain non-obvious logic. Reference related code.
- **API consumers**: Be precise about types, required vs optional params, return values, and error conditions.

### 3. Choose the Right Format

#### JSDoc (JavaScript/TypeScript)

Use for functions, classes, and modules:

```js
/**
 * Calculates the contrast ratio between two hex color values.
 *
 * Uses the WCAG 2.1 relative luminance formula. Returns a value
 * between 1 (no contrast) and 21 (maximum contrast).
 *
 * @param {string} hex1 - First color as a hex string (e.g. "#ffffff")
 * @param {string} hex2 - Second color as a hex string (e.g. "#000000")
 * @returns {number} Contrast ratio between 1 and 21
 *
 * @example
 * contrastRatio('#ffffff', '#000000') // => 21
 * contrastRatio('#777777', '#888888') // => 1.16
 */
function contrastRatio(hex1, hex2) { ... }
```

#### Inline Comments

Use sparingly — only when the *why* is not obvious from the code:

```js
// Clamp to [0, 255] before converting — browser APIs may return
// values slightly outside this range due to floating point errors
const clamped = Math.min(255, Math.max(0, value));
```

Avoid restating what the code already says:

```js
// BAD: increment counter
counter++;
```

#### README Sections

A good README includes:

1. **What it is** — one or two sentences
2. **Why it exists** — the problem it solves
3. **Quick start** — minimal working example
4. **Installation** — exact commands
5. **Usage** — common patterns with examples
6. **API reference** — if applicable
7. **Contributing** — how to get involved
8. **License**

### 4. Write Clearly

- Use active voice: *"Returns the parsed value"* not *"The parsed value is returned"*
- Be specific: *"Throws if `input` is null"* not *"May throw an error"*
- Use examples liberally — they communicate faster than prose
- Keep sentences short
- Define acronyms on first use
- Avoid filler phrases: *"basically"*, *"simply"*, *"just"*, *"obviously"*

### 5. Validate Accuracy

Documentation that is wrong is worse than no documentation:

- Verify parameter names match the actual function signature
- Confirm return types and shapes are accurate
- Test any code examples before including them
- Check that described behavior matches implementation
- Flag anything uncertain with a `// TODO: verify` comment

## Quality Checklist

- [ ] Every exported function/class has a JSDoc block
- [ ] All parameters and return values are typed and described
- [ ] At least one usage example per public API method
- [ ] No documentation contradicts the actual code behavior
- [ ] README answers: what, why, how to install, how to use
- [ ] Inline comments explain *why*, not *what*
- [ ] No placeholder text or unfinished sentences
- [ ] Consistent terminology throughout

## Anti-Patterns to Avoid

| Anti-pattern | Problem | Fix |
|---|---|---|
| Restating the code | Adds noise, no value | Explain intent or edge cases instead |
| Stale docs | Misleads readers | Update docs in the same commit as code changes |
| Over-documenting trivial code | Cognitive overhead | Document only non-obvious logic |
| Vague descriptions | Unhelpful | Be specific about types, constraints, behavior |
| No examples | Hard to apply | Add at least one concrete usage example |
| Docs in wrong place | Hard to find | Co-locate docs with the code they describe |

## Integration with Other Skills

- Use **audit** to identify undocumented code before running `document`
- Use **critique** to review documentation quality after writing
- Use **clarify** when the purpose of code is ambiguous before documenting it
- Use **bolder** if documentation is too timid or hedge-filled
