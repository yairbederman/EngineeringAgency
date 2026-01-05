# Generated Artifacts Reference

After successful execution, the following files **MUST** exist in `${AI_INSTRUCTIONS_ROOT}`:

## Required Files (Always Generated)

```
.ai-instructions/
├── copilot-instructions.md       # Master AI instructions (entry point)
├── analysis/
│   ├── techstack.md              # Stack detection results
│   ├── source-structure.json     # Discovered locations + file counts + detectedCapabilities
│   ├── entity-contracts.json     # Type definitions with fields
│   ├── api-contracts.json        # REST endpoints with validation
│   ├── function-registry.json    # Service dependencies
│   ├── file-categorization.json  # Files grouped by layer
│   └── error-taxonomy.json       # Error codes + response shapes (UNIVERSAL)
└── deep-dive/
    ├── dependency-chains.md      # Controller → Service → External chains
    └── data-flow.md              # Data object transformations
```

## Conditional Files (Generated When Applicable)

```
.ai-instructions/
├── analysis/
│   ├── design-tokens.json        # (Frontend only) CSS/Tailwind tokens
│   ├── component-registry.json   # (Frontend only) React/Vue components
│   ├── state-contracts.json      # (Frontend only) Redux/Zustand full action payloads
│   ├── database-schema.json      # (Backend only) Table definitions + migrations
│   ├── validation-schemas.json   # (Conditional) Zod/Yup/Joi validation rules
│   ├── inter-service-contracts.json  # (Backend only) Microservice call DTOs
│   └── external-integrations.json    # (Conditional) Third-party SDK wrappers
└── deep-dive/
    ├── component-registry.md     # (Frontend only) Component documentation
    ├── debugging-guide.md        # (If complex error handling exists)
    └── testing-strategy.md       # (If test files exist)
```

## Regeneration Rule

> [!WARNING]
> If ANY file in `.ai-instructions/` exists but is NOT in the lists above, either:
> 1. **Regenerate it** with fresh content and timestamp, OR
> 2. **Delete it** if no longer applicable (explain why)
