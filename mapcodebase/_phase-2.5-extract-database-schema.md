---
description: Phase 2.5 - Extract database schema contracts for migration-aware field additions
---

# Phase 2.5: Extract Database Schema Contracts

## Goal
Extract database table/collection definitions with columns, constraints, and entity mappings—enabling Tech Specs to determine if migrations are needed.

## Trigger Condition
**Execute this phase IF** `source-structure.json.detectedStack` includes:
- JPA/Hibernate annotations (`@Entity`, `@Table`, `@Column`)
- TypeORM decorators (`@Entity`, `@Column`, `@PrimaryGeneratedColumn`)
- Prisma schema (`schema.prisma`)
- Mongoose schemas (`new Schema({...})`)
- SQLAlchemy models (`Base`, `Column`, `relationship`)
- Django models (`models.Model`)

**Skip IF**: No ORM/database layer detected.

## Input
Use `source-structure.json.discoveredLocations.entities` and scan for database annotations.

## Steps

### 1. Detect Database Layer Pattern
Scan project for ORM indicators:

| Ecosystem | Detection Patterns |
|-----------|-------------------|
| JVM/Spring | `@Entity`, `@Table`, `jakarta.persistence.*` |
| Node/TypeORM | `@Entity()`, `@Column()`, `typeorm` imports |
| Node/Prisma | `schema.prisma` file exists |
| Node/Mongoose | `mongoose.Schema`, `new Schema` |
| Python/SQLAlchemy | `declarative_base()`, `Column`, `relationship` |
| Python/Django | `models.Model`, `CharField`, `ForeignKey` |

### 2. Extract Table Definitions
For EACH entity with database mapping:

| Field | Source |
|-------|--------|
| `tableName` | `@Table(name=)` or class name (snake_case convention) |
| `columns` | All `@Column` annotated fields |
| `columnType` | Database type from annotation or inferred from field type |
| `nullable` | `@Column(nullable=true)` or `optional` in schema |
| `defaultValue` | `@Column(default=)` or `@ColumnDefault` |
| `primaryKey` | `@Id`, `@PrimaryKey`, `@PrimaryGeneratedColumn` |
| `foreignKeys` | `@ManyToOne`, `@OneToMany`, `references` |

### 3. Extract Constraints and Indexes
Identify table-level constraints:

| Constraint Type | Detection |
|-----------------|-----------|
| Unique | `@UniqueConstraint`, `unique: true` |
| Index | `@Index`, `@@index` in Prisma |
| Composite Key | `@IdClass`, composite primary key |
| Check | `@Check`, check constraint |

### 4. Map Entity-to-Table Relationships
Cross-reference with `entity-contracts.json`:

```json
{
  "entityName": "[from entity-contracts]",
  "tableName": "[actual DB table]",
  "discrepancies": [
    { "field": "[fieldName]", "inEntity": true, "inTable": false }
  ]
}
```

### 5. Detect Migration Framework
Look for migration tooling:

| Framework | Detection |
|-----------|-----------|
| Flyway | `db/migration/V*__*.sql` files |
| Liquibase | `changelog*.xml` or `changelog*.yaml` |
| TypeORM | `migrations/` folder with `*.ts` |
| Prisma | `prisma/migrations/` folder |
| Alembic | `alembic/versions/` folder |
| Django | `*/migrations/*.py` files |

## Output

### `analysis/database-schema.json`
```json
{
  "detectedOrm": "[ORM name]",
  "tables": {
    "[TableName]": {
      "entity": "[EntityClass from entity-contracts]",
      "file": "[entity file path]",
      "columns": {
        "[columnName]": {
          "fieldName": "[entity field name]",
          "type": "VARCHAR(255)",
          "nullable": false,
          "primaryKey": false,
          "autoGenerate": false,
          "default": "[value or null]",
          "foreignKey": {
            "target": "[TargetTable]",
            "column": "[targetColumn]",
            "onDelete": "CASCADE"
          }
        }
      },
      "indexes": [
        {
          "name": "[indexName]",
          "columns": ["[col1]", "[col2]"],
          "unique": false
        }
      ],
      "constraints": [
        {
          "type": "unique",
          "columns": ["[col1]", "[col2]"]
        }
      ]
    }
  },
  "_migrations": {
    "framework": "[Flyway|Liquibase|Prisma|...]",
    "location": "[path to migrations]",
    "latestVersion": "[version identifier]",
    "pendingCount": 0
  },
  "_entityTableMapping": {
    "[EntityName]": {
      "table": "[TableName]",
      "missingInDb": ["[field names in entity but not in table]"],
      "missingInEntity": ["[columns in table but not in entity]"]
    }
  },
  "_coverage": {
    "entitiesWithDbMapping": 15,
    "tablesExtracted": 15,
    "columnsExtracted": 87
  }
}
```

## Critical Rules

1. **Entity-Table Sync**: Flag discrepancies between entity fields and table columns
2. **Migration Awareness**: Document migration framework so Tech Spec knows if migration is needed
3. **Type Mapping**: Resolve database types (VARCHAR, INT, TIMESTAMP) not just programming types
4. **Foreign Key Chains**: Document FK relationships for referential integrity checks
5. **Coverage**: Track entities vs tables to ensure completeness
