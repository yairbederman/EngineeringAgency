# Common Validation Protocol

> **Token Efficiency**: Phase files should reference this instead of repeating each rule.
> **Usage**: `Validation: See _validation-common.md`

---

## Dependency Verification Protocol (BLOCKING)

Every claimed dependency MUST pass this verification:

### Step 1: Search for Evidence
```bash
# For HTTP dependencies
grep_search("<target-baseUrl>|<target-ServiceClient>", "${PROJECT_PATH}/src")

# For library dependencies
grep_search("import.*<package-name>", "${PROJECT_PATH}/src")
```

### Step 2: Verify Active Usage
- Config key EXISTS is not enough
- Must find actual USAGE in code (client class, fetch call, import statement)
- Dormant config = exclude from output

### Step 3: Document Evidence
```json
{
  "codeEvidence": "[file]:[line] - [what was found]",
  "verified": true
}
```

### Decision Matrix

| Evidence Found | Action |
|----------------|--------|
| File + line number | Include in output with `verified: true` |
| File only (no line) | Include with warning in `_warnings[]` |
| None found | **EXCLUDE** from output, add to `_warnings[]` |

---

## Coverage Tracking Protocol

Every output file MUST include `_coverage`:

```json
"_coverage": {
  "[category]Documented": 45,
  "[category]Total": 50,
  "percentage": 90,
  "skipped": [{ "path": "[dir]", "reason": "[valid reason]" }]
}
```

### Valid Skip Reasons
- `"configuration-only"` - File contains only config, no logic
- `"base-class"` - Abstract base class
- `"deprecated"` - Marked deprecated in code
- `"test-file"` - Test/spec file

### Invalid Skip Reasons (BLOCK workflow)
- `"not fully extracted"` → Extract now
- `"complex structure"` → Extract now, mark unresolved
- `"Pending"` → Invalid
- Blank or vague → Invalid

---

## Evidence Quality Gate

### HIGH Quality (✅ Include)
- Specific `file:line` with code snippet
- Example: `OrderService.java:76 - for-loop over items`

### LOW Quality (❌ Exclude)
- "Inferred from architecture"
- "Standard pattern"
- "Assumed based on naming"

**Rule**: If evidence uses "inferred", "assumed", or "pattern", re-search for actual code evidence or exclude.
