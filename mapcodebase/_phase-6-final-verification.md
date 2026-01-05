# Phase 6: Final Verification (BLOCKING GATE)

> [!WARNING]
> **THIS PHASE IS NOT OPTIONAL.** Do NOT report completion until all verification steps pass.

## Step 1: List all generated files

For EACH target project, list the contents of `.ai-instructions/`:

```bash
find ${PROJECT_ROOT}/.ai-instructions/ -type f
```

## Step 2: Verify Required Files Exist

Check EACH required file (see `_artifacts-reference.md`):

| File | Status |
|------|--------|
| `copilot-instructions.md` | [ ] Exists |
| `analysis/techstack.md` | [ ] Exists |
| `analysis/source-structure.json` | [ ] Exists |
| `analysis/entity-contracts.json` | [ ] Exists |
| `analysis/api-contracts.json` | [ ] Exists |
| `analysis/function-registry.json` | [ ] Exists |
| `analysis/file-categorization.json` | [ ] Exists |
| `analysis/error-taxonomy.json` | [ ] Exists |
| `deep-dive/dependency-chains.md` | [ ] Exists |
| `deep-dive/data-flow.md` | [ ] Exists |

## Step 3: Check for stale files

If ANY file exists in `.ai-instructions/` that is NOT in the required or conditional lists:
- **Option A**: Regenerate it with current content
- **Option B**: Delete it (and document why in summary)

> [!IMPORTANT]
> **Since Phase 0 deleted all files**, there should be NO stale files. If stale files exist, Phase 0 was not executed correctly. Re-run from Phase 0.

## Step 4: Confirmation gate

> [!CAUTION]
> **DO NOT mark workflow complete** until:
> - [ ] All required files exist for ALL target projects
> - [ ] No stale files remain
> - [ ] Coverage thresholds were met (Phase 4.5)
