# Phase 4: Unified Domain Model

> **Goal**: Merge entity definitions across projects to identify canonical entities and type conflicts.

---

## Input

- `${ANALYSIS_DIR}/project-inventory.json` (from Phase 1)
- Each project's `entity-contracts.json`
- Each project's `database-schema.json` **(NEW - if available for backend projects)**

## Output

- `${ANALYSIS_DIR}/unified-domain-model.json`

---

## Instructions

### Step 1: Extract All Entities

For each `ready` project:
- Read `entity-contracts.json`
- Extract all entities with their fields
- **NEW**: If `database-schema.json` exists:
  - Enrich entities with table mappings
  - Add column types and constraints
  - Flag entity-table discrepancies

### Step 2: Group by Entity Name

Group entities that share the same name across projects:

```
<EntityName>:
  - <project-a>/<EntityName> (<N> fields)
  - <project-b>/<EntityName> (<M> fields)
  - <project-c>/<EntityName> (<P> fields)
```

### Step 3: Identify Canonical Definitions

For each entity group, determine:

1. **Canonical Source**: The project that "owns" this entity
   - Typically the project where it's most complete
   - Or the project that creates/manages this data

2. **Derived Copies**: Other projects that use a subset
   - May have fewer fields
   - May be read-only views

3. **Conflicts**: Fields with different types across projects

### Step 4: Classify Entities

| Classification | Description |
|---------------|-------------|
| **Canonical** | Single authoritative definition |
| **Shared** | Defined once, used in multiple projects |
| **Duplicated** | Same entity defined in multiple projects |
| **Conflicting** | Same name but incompatible fields |

### Step 5: Build Unified Model

For each canonical entity:
- Merge all fields
- Note field source
- Flag conflicts

---

## Output Schema

```json
{
  "generatedAt": "<ISO-8601 timestamp>",
  "entities": [
    {
      "name": "<EntityName>",
      "classification": "<canonical | shared | duplicated | conflicting>",
      "canonicalSource": {
        "project": "<owning project>",
        "file": "<relative path to source>",
        "reason": "<why this is the canonical source>"
      },
      "unifiedFields": [
        {
          "name": "<fieldName>",
          "type": "<type>",
          "definedIn": ["<list of projects defining this field>"],
          "status": "<consistent | conflicting>"
        },
        {
          "name": "<anotherField>",
          "type": "<type>",
          "definedIn": ["<project-a>", "<project-b>"],
          "conflict": {
            "<project-a>": "<type-in-a>",
            "<project-c>": "<type-in-c>"
          },
          "status": "conflicting"
        }
      ],
      "usedIn": ["<list of projects using this entity>"]
    }
    // ... one entry per canonical entity
  ],
  "domainAreas": [
    {
      "name": "<DomainArea>",
      "entities": ["<list of entities in this domain>"],
      "ownedBy": "<owning project>"
    }
    // ... one entry per domain area
  ],
  "_conflicts": [
    {
      "entity": "<EntityName>",
      "field": "<fieldName>",
      "issue": "<description of the conflict>",
      "recommendation": "<how to resolve>"
    }
    // ... only populated if conflicts exist
  ],
  "_coverage": {
    "projectsAnalyzed": "<count>",
    "entitiesFound": "<count>",
    "canonicalEntities": "<count>",
    "duplicatedEntities": "<count>",
    "conflictingFields": "<count>"
  }
}
```

---

## Domain Area Detection

Automatically group entities into domain areas based on:

1. **Naming patterns**: `Booking*`, `Payment*`, `Search*`
2. **Package structure**: Entities in same package
3. **Usage patterns**: Entities used together in API calls

---

## Validation

| Check | Requirement |
|-------|-------------|
| All entity files read | Count matches projects × entity files |
| Canonical assigned | Every entity has a `canonicalSource` |
| Conflicts documented | No silent type mismatches |

---

## Conflict Resolution Recommendations

| Conflict Type | Recommendation |
|--------------|----------------|
| Type mismatch | Standardize to canonical source type |
| Missing field | Check if intentionally excluded or oversight |
| Extra field | May be valid extension or legacy |
| Name collision | Different concepts → rename one |
