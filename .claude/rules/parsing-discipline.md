---
description: Parsing and pattern-matching rules — loads only when editing parser or schema source
paths:
  - "src/parse/**"
  - "src/schema/**"
  - "src/**/*parse*.ts"
  - "src/**/*schema*.ts"
---

# Parsing Discipline (Context Forge)

## Lenience principle

CF parsers must degrade gracefully on partial or malformed input — never throw on missing optional fields. The resolver chain (`fileSlicePlan`, `fileSlice`, `fileTasks`) must return a best-effort result with a warning, not an exception.

```typescript
// WRONG — crashes on missing field
const phase = doc.frontmatter.phase;  // throws if undefined

// CORRECT — lenient with fallback
const phase = doc.frontmatter?.phase ?? 'unknown';
```

## Resolver quirks (known, do not regress)

- `fileSlicePlan` resolves from `user/architecture/`, not `user/slices/`
- Slice plan parser reads **numbered list format only** — tables are ignored
- `indexRange` spans multiple hundred-blocks in worktrees → band-check warning is cosmetic, not an error (see `reference_cf_band_warning_bug.md`)

## Schema changes

Any change to frontmatter field names or required/optional status must:
1. Update `src/schema/` type definitions
2. Update the lenience fallback in the relevant resolver
3. Add a test case in `tests/` for the missing-field path

## Test before staging parser changes

```bash
npm test -- --grep "parse\|schema\|resolve" 2>&1 | tail -15
```
