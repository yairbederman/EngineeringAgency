# Workflow Error Codes

> **Purpose**: Structured error codes for all workflow failures.
> **Usage**: When a workflow fails, emit the error code for categorization, automation, and debugging.

---

## Error Code Format

```
[AGENT]-[PHASE/MODE]-[NUMBER]

Examples:
- MCB-P1-001  → Map Codebase Agent, Phase 1, Error #1
- ENG-TS-003  → Engineering Agent, TechSpec Mode, Error #3
- SYS-P4-002  → System Architecture Agent, Phase 4, Error #2
- MGR-BT-001  → Manager Agent, Beat Mode, Error #1
```

---

## Severity Levels

| Level | Code | Description | Action |
|-------|------|-------------|--------|
| **BLOCKING** | 🔴 | Cannot proceed. Workflow halts. | Fix before continuing |
| **WARNING** | 🟠 | Can proceed with degraded quality. | Log and continue, surface to user |
| **INFO** | 🟡 | Informational. No action required. | Log only |

---

## Map Codebase Agent (MCB)

### Phase 0: Clean Slate
| Code | Severity | Description | Resolution |
|------|----------|-------------|------------|
| `MCB-P0-001` | 🔴 BLOCKING | Failed to delete `.ai-instructions/` folder | Check file permissions, close open files |
| `MCB-P0-002` | 🔴 BLOCKING | User did not approve folder deletion | Request explicit "Yes" or "Approved" |
| `MCB-P0-003` | 🟠 WARNING | Folder already deleted (no-op) | Continue to Phase 1 |

### Phase 1: Detect Stack
| Code | Severity | Description | Resolution |
|------|----------|-------------|------------|
| `MCB-P1-001` | 🔴 BLOCKING | No recognized project files found | Verify project path, check for package.json/pom.xml/etc. |
| `MCB-P1-002` | 🔴 BLOCKING | Project not in registered projects list | Add project to `shared/configuration.md` |
| `MCB-P1-003` | 🟠 WARNING | Multiple ecosystems detected | Document primary and secondary stacks |
| `MCB-P1-004` | 🟠 WARNING | Source structure incomplete (missing directories) | List missing expected directories |
| `MCB-P1-005` | 🟡 INFO | Monorepo structure detected | Switch to multi-module extraction mode |

### Phase 2: Extract Entities
| Code | Severity | Description | Resolution |
|------|----------|-------------|------------|
| `MCB-P2-001` | 🔴 BLOCKING | Entity extraction coverage <30% | Re-run with expanded file patterns |
| `MCB-P2-002` | 🟠 WARNING | Unresolved `any` types detected | Add to `_unresolved` section |
| `MCB-P2-003` | 🟠 WARNING | Circular type references detected | Document cycle in `_notes` |
| `MCB-P2-004` | 🟠 WARNING | Generic types with complex constraints | Simplify to base type in output |
| `MCB-P2-005` | 🟡 INFO | External types (from node_modules) skipped | Expected behavior |

### Phase 2.5: Extract Database Schema (Backend Only)
| Code | Severity | Description | Resolution |
|------|----------|-------------|------------|
| `MCB-P25-001` | 🔴 BLOCKING | ORM detected but no entity mappings found | Check ORM configuration files |
| `MCB-P25-002` | 🟠 WARNING | Migration files found but not parseable | Document migration framework manually |
| `MCB-P25-003` | 🟠 WARNING | Entity-table mapping incomplete | Cross-reference with entity-contracts.json |
| `MCB-P25-004` | 🟡 INFO | Phase skipped (no ORM/database layer detected) | Expected for stateless services |

### Phase 2.6: Extract Validation Schemas (Conditional)
| Code | Severity | Description | Resolution |
|------|----------|-------------|------------|
| `MCB-P26-001` | 🟠 WARNING | Validation library detected but <80% coverage | List undocumented validation schemas |
| `MCB-P26-002` | 🟠 WARNING | Custom validators not fully extracted | Document validator names and file paths |
| `MCB-P26-003` | 🟡 INFO | Phase skipped (no validation library detected) | Expected for some projects |

### Phase 3: Extract APIs
| Code | Severity | Description | Resolution |
|------|----------|-------------|------------|
| `MCB-P3-001` | 🔴 BLOCKING | API endpoint extraction coverage <100% | List undocumented controllers/routes |
| `MCB-P3-002` | 🔴 BLOCKING | Request/response types unresolved | Cross-reference with entity-contracts.json |
| `MCB-P3-003` | 🟠 WARNING | Validation rules not extracted for endpoints | Re-extract with validation metadata |
| `MCB-P3-004` | 🟠 WARNING | Path parameters not typed | Infer from usage or mark as `string` |
| `MCB-P3-005` | 🟡 INFO | Internal-only endpoints detected | Mark with `internal: true` flag |

### Phase 3.5: Extract Design Tokens (Frontend Only)
| Code | Severity | Description | Resolution |
|------|----------|-------------|------------|
| `MCB-P35-001` | 🟠 WARNING | Tailwind config not found | Check for tailwind.config.js/ts |
| `MCB-P35-002` | 🟠 WARNING | CSS variables detected but not parseable | Extract manually from CSS files |
| `MCB-P35-003` | 🟡 INFO | Phase skipped (no design system detected) | Expected for some projects |

### Phase 3.6: Extract Component Registry (Frontend Only)
| Code | Severity | Description | Resolution |
|------|----------|-------------|------------|
| `MCB-P36-001` | 🟠 WARNING | Component coverage <70% | List undocumented components |
| `MCB-P36-002` | 🟠 WARNING | Props extraction incomplete (complex types) | Use simplified prop types |
| `MCB-P36-003` | 🟡 INFO | Phase skipped (no React/Vue/Angular detected) | Expected for non-SPA projects |

### Phase 3.7: Extract State Contracts (Frontend Only)
| Code | Severity | Description | Resolution |
|------|----------|-------------|------------|
| `MCB-P37-001` | 🔴 BLOCKING | State module coverage <100% | All state must be documented |
| `MCB-P37-002` | 🟠 WARNING | Async thunk lifecycle not fully extracted | Document pending/fulfilled/rejected |
| `MCB-P37-003` | 🟠 WARNING | Selector dependencies unclear | Mark as `_needsReview` |
| `MCB-P37-004` | 🟡 INFO | Phase skipped (no state management detected) | Expected for stateless UIs |

### Phase 4: Map Dependencies
| Code | Severity | Description | Resolution |
|------|----------|-------------|------------|
| `MCB-P4-001` | 🟠 WARNING | Service coverage <50% | List undocumented services |
| `MCB-P4-002` | 🟠 WARNING | Cross-project dependency detected but unverified | Check target project's api-contracts.json |
| `MCB-P4-003` | 🟠 WARNING | Call chain incomplete (max depth reached) | Document partial chain with `_truncated` flag |
| `MCB-P4-004` | 🟠 WARNING | External API calls without code evidence | Add `codeEvidence` field |
| `MCB-P4-005` | 🟡 INFO | Circular dependency detected | Document in dependency-chains.md |

### Phase 4.2: Extract Inter-Service Contracts (Backend Only)
| Code | Severity | Description | Resolution |
|------|----------|-------------|------------|
| `MCB-P42-001` | 🔴 BLOCKING | Target service not in registered projects | Add target to shared/configuration.md |
| `MCB-P42-002` | 🟠 WARNING | Target service has no api-contracts.json | Run /map-codebase-agent on target first |
| `MCB-P42-003` | 🟠 WARNING | Contract mismatch detected (caller != callee) | Flag for manual review |
| `MCB-P42-004` | 🟡 INFO | Phase skipped (no inter-service calls detected) | Expected for standalone services |

### Phase 4.3: Extract External Integrations (Conditional)
| Code | Severity | Description | Resolution |
|------|----------|-------------|------------|
| `MCB-P43-001` | 🟠 WARNING | Third-party SDK detected but wrapper not found | Check for service classes using the SDK |
| `MCB-P43-002` | 🟠 WARNING | Webhook handlers detected but not documented | Add to external-integrations.json |
| `MCB-P43-003` | 🟡 INFO | Phase skipped (no third-party SDKs detected) | Expected for self-contained services |

### Phase 4.4: Extract Error Taxonomy (Universal)
| Code | Severity | Description | Resolution |
|------|----------|-------------|------------|
| `MCB-P44-001` | 🔴 BLOCKING | No error handling patterns found | Verify project has custom exceptions/error classes |
| `MCB-P44-002` | 🟠 WARNING | Error codes used but not all documented | List undocumented error codes |
| `MCB-P44-003` | 🟠 WARNING | Error propagation patterns unclear | Document in error-taxonomy.json |

### Phase 4.5: Enforcement Gate
| Code | Severity | Description | Resolution |
|------|----------|-------------|------------|
| `MCB-P45-001` | 🔴 BLOCKING | Coverage threshold not met (controllers <100%) | Return to Phase 3 |
| `MCB-P45-002` | 🔴 BLOCKING | Coverage threshold not met (services <50%) | Return to Phase 4 |
| `MCB-P45-003` | 🔴 BLOCKING | Coverage threshold not met (hooks <80%) | Return to Phase 4 |
| `MCB-P45-004` | 🔴 BLOCKING | Coverage threshold not met (state <100%) | Return to Phase 3.7 |
| `MCB-P45-005` | 🔴 BLOCKING | Invalid skip reasons found ("Pending", "not extracted") | Return to failing phase |

### Phase 5: Generate Master
| Code | Severity | Description | Resolution |
|------|----------|-------------|------------|
| `MCB-P5-001` | 🔴 BLOCKING | Missing required input file (entity-contracts.json) | Re-run Phase 2 |
| `MCB-P5-002` | 🔴 BLOCKING | Missing required input file (api-contracts.json) | Re-run Phase 3 |
| `MCB-P5-003` | 🟠 WARNING | Architecture diagram generation failed | Include text-based architecture instead |
| `MCB-P5-004` | 🟠 WARNING | Feature patterns table incomplete | Add patterns from code analysis |

### Phase 6: Final Verification
| Code | Severity | Description | Resolution |
|------|----------|-------------|------------|
| `MCB-P6-001` | 🔴 BLOCKING | Required artifact missing | List missing files, return to generating phase |
| `MCB-P6-002` | 🔴 BLOCKING | JSON validation failed (malformed output) | Re-generate the failing artifact |
| `MCB-P6-003` | 🟠 WARNING | Stale files detected (not regenerated) | Delete stale files manually |
| `MCB-P6-004` | 🟡 INFO | All verification checks passed | Workflow complete |

---

## Engineering Agent (ENG)

### General / Context
| Code | Severity | Description | Resolution |
|------|----------|-------------|------------|
| `ENG-CTX-001` | 🔴 BLOCKING | `copilot-instructions.md` not found | Run /map-codebase-agent first |
| `ENG-CTX-002` | 🔴 BLOCKING | Project not in registered projects | Add to shared/configuration.md |
| `ENG-CTX-003` | 🟠 WARNING | Context7 unavailable | Request file paths/snippets from user |
| `ENG-CTX-004` | 🟠 WARNING | System architecture docs not found | Run /system-architecture-agent first |
| `ENG-CTX-005` | 🟠 WARNING | MCP tool response truncated | Attempt browser fallback, then ask user |

### ProductSpecReview Mode
| Code | Severity | Description | Resolution |
|------|----------|-------------|------------|
| `ENG-PSR-001` | 🔴 BLOCKING | No Product Spec provided (missing Confluence/Jira link) | Request spec from user |
| `ENG-PSR-002` | 🔴 BLOCKING | Confluence page fetch failed (MCP error) | Request pasted content from user |
| `ENG-PSR-003` | 🟠 WARNING | Critical gaps found (spec not implementation-ready) | Present gap analysis, request clarification |
| `ENG-PSR-004` | 🟡 INFO | No critical gaps found | Proceed to FeaturePlanning |

### DesignAnalysis Mode
| Code | Severity | Description | Resolution |
|------|----------|-------------|------------|
| `ENG-DA-001` | 🟠 WARNING | Figma link provided but MCP extraction failed | Request screenshot from user |
| `ENG-DA-002` | 🟠 WARNING | No Figma link in spec (UI work expected) | Use [TBD – Design] placeholders |
| `ENG-DA-003` | 🟠 WARNING | Figma tokens don't map to project design system | Document mapping gaps |
| `ENG-DA-004` | 🟡 INFO | No Figma link (non-UI work) | Skip phase with approval |

### FeaturePlanning Mode
| Code | Severity | Description | Resolution |
|------|----------|-------------|------------|
| `ENG-FP-001` | 🔴 BLOCKING | Epic creation failed (Jira MCP error) | Request manual Epic creation |
| `ENG-FP-002` | 🔴 BLOCKING | Confluence link-back failed | Request manual update |
| `ENG-FP-003` | 🟠 WARNING | Epic created but Confluence update failed | Doc link manually |
| `ENG-FP-004` | 🟠 WARNING | Assumptions logged (needs validation) | Add `needs-validation` label |
| `ENG-FP-005` | 🟡 INFO | Epic created successfully | Proceed to TechSpec |

### TechSpec Mode
| Code | Severity | Description | Resolution |
|------|----------|-------------|------------|
| `ENG-TS-001` | 🔴 BLOCKING | No Epic found (required input) | Create Epic first via FeaturePlanning |
| `ENG-TS-002` | 🔴 BLOCKING | Architecture validation failed (no pattern match) | Consult copilot-instructions.md |
| `ENG-TS-003` | 🟠 WARNING | Tech Spec has multiple [TBD] placeholders | Surface in Assumptions section |
| `ENG-TS-004` | 🟠 WARNING | Cross-service impact detected but contracts unavailable | Flag for manual review |
| `ENG-TS-005` | 🟠 WARNING | Tech Spec Jira injection failed (MCP error) | Provide content for manual creation |
| `ENG-TS-006` | 🟡 INFO | Tech Spec approved | Proceed to TaskPlanning |

### TaskPlanning Mode
| Code | Severity | Description | Resolution |
|------|----------|-------------|------------|
| `ENG-TP-001` | 🔴 BLOCKING | No Tech Spec found (required input) | Create Tech Spec first |
| `ENG-TP-002` | 🔴 BLOCKING | Figma extraction required but failed | Use [TBD – Design] or request screenshot |
| `ENG-TP-003` | 🔴 BLOCKING | Task creation failed (Jira MCP error) | Request manual task creation |
| `ENG-TP-004` | 🟠 WARNING | Task file paths use placeholders (`...`) | Replace with absolute paths |
| `ENG-TP-005` | 🟠 WARNING | Category field missing (file-categorization.json lookup failed) | Infer from file path |
| `ENG-TP-006` | 🟠 WARNING | Test plan incomplete (<2 test cases) | Add minimum test cases |
| `ENG-TP-007` | 🟡 INFO | All tasks created successfully | Present summary, wait for selection |

### Implementation Mode
| Code | Severity | Description | Resolution |
|------|----------|-------------|------------|
| `ENG-IMPL-001` | 🔴 BLOCKING | Entry conditions not met (no Task/Epic/Spec) | Return to Planning modes |
| `ENG-IMPL-002` | 🔴 BLOCKING | Fast Track rejected (task too complex) | Create proper TechSpec/Tasks |
| `ENG-IMPL-003` | 🟠 WARNING | Test generation failed (missing test framework) | Document test expectations |
| `ENG-IMPL-004` | 🟠 WARNING | Pattern mismatch with copilot-instructions.md | Flag deviation for review |
| `ENG-IMPL-005` | 🟡 INFO | Implementation complete | Proceed to PullRequest |

### BugFix Mode
| Code | Severity | Description | Resolution |
|------|----------|-------------|------------|
| `ENG-BF-001` | 🔴 BLOCKING | Bug report incomplete (no repro steps) | Request structured bug input |
| `ENG-BF-002` | 🔴 BLOCKING | Root cause undetermined | Request additional context |
| `ENG-BF-003` | 🟠 WARNING | Regression test generation failed | Document expected test manually |
| `ENG-BF-004` | 🟡 INFO | Bug fix complete with regression test | Proceed to PullRequest |

### PullRequest Mode
| Code | Severity | Description | Resolution |
|------|----------|-------------|------------|
| `ENG-PR-001` | 🟠 WARNING | No linked Jira issues detected | Add issue keys manually |
| `ENG-PR-002` | 🟠 WARNING | Breaking changes detected | Add breaking change section |
| `ENG-PR-003` | 🟠 WARNING | Self-review checklist incomplete | Complete checklist |
| `ENG-PR-004` | 🟡 INFO | PR description generated | Ready for submission |

### CodeReview Mode
| Code | Severity | Description | Resolution |
|------|----------|-------------|------------|
| `ENG-CR-001` | 🟠 WARNING | Architectural violations detected | Flag for discussion |
| `ENG-CR-002` | 🟠 WARNING | Test coverage insufficient | Request additional tests |
| `ENG-CR-003` | 🟡 INFO | Review complete, no blocking issues | Approve with comments |

### Template Contract Validation (All Planning Modes)

> **Reference**: See `engineering/templates/_template-contracts.md` for the full contract definitions.

| Code | Severity | Description | Resolution |
|------|----------|-------------|------------|
| `ENG-FP-ERR-001` | 🔴 BLOCKING | Epic missing Goal section | Add Goal to template output |
| `ENG-FP-ERR-002` | 🔴 BLOCKING | Epic missing Acceptance Criteria | Add Gherkin scenarios |
| `ENG-FP-ERR-003` | 🔴 BLOCKING | Epic has no Functional Flows | Add at least 2 flows (Happy + Error) |
| `ENG-FP-ERR-004` | 🟠 WARNING | UI work but no Figma link | Add Figma link to Context |
| `ENG-TS-ERR-001` | 🔴 BLOCKING | Tech Spec missing Epic reference | Add valid Jira key |
| `ENG-TS-ERR-002` | 🔴 BLOCKING | Tech Spec has no architecture validation | Reference copilot-instructions.md |
| `ENG-TS-ERR-003` | 🔴 BLOCKING | Tech Spec missing API contracts | Add request/response types |
| `ENG-TS-ERR-004` | 🔴 BLOCKING | Tech Spec has no verification strategy | Add test plan |
| `ENG-TS-ERR-005` | 🟠 WARNING | Core logic has [TBD] placeholders | Resolve or escalate to user |
| `ENG-TP-ERR-001` | 🔴 BLOCKING | Task file paths have placeholders (`...`) | Replace with absolute paths |
| `ENG-TP-ERR-002` | 🟠 WARNING | Task missing Category field | Lookup in file-categorization.json |
| `ENG-TP-ERR-003` | 🔴 BLOCKING | API task missing API Contract | Copy from Tech Spec § 4 |
| `ENG-TP-ERR-004` | 🟠 WARNING | Task has fewer than 2 test cases | Add Happy Path + Edge case |
| `ENG-TP-ERR-005` | 🔴 BLOCKING | Task missing Pattern Context | Add Pattern + Reference File + Rationale |
| `ENG-TP-ERR-006` | 🔴 BLOCKING | Frontend task missing UI Implementation Guide | Add Token Mapping table |
| `ENG-TP-ERR-007` | 🔴 BLOCKING | Frontend task missing Component Instances | Identify reusable components |
| `ENG-TP-ERR-008` | 🟠 WARNING | Frontend task missing Token Mapping | Map Figma values to project tokens |
| `ENG-TP-ERR-009` | 🟠 WARNING | Epic has Figma but task didn't extract | Call MCP Figma tool |

---

## System Architecture Agent (SYS)

### Phase 0: Clean Slate
| Code | Severity | Description | Resolution |
|------|----------|-------------|------------|
| `SYS-P0-001` | 🔴 BLOCKING | Failed to clean output directory | Check file permissions |
| `SYS-P0-002` | 🟡 INFO | Output directory already clean | Continue to Phase 1 |

### Phase 1: Project Inventory
| Code | Severity | Description | Resolution |
|------|----------|-------------|------------|
| `SYS-P1-001` | 🔴 BLOCKING | No registered projects found | Add projects to shared/configuration.md |
| `SYS-P1-002` | 🟠 WARNING | Project missing .ai-instructions | Run /map-codebase-agent on project |
| `SYS-P1-003` | 🟠 WARNING | copilot-instructions.md not readable | Check file format |
| `SYS-P1-004` | 🟡 INFO | All projects scanned successfully | Continue to Phase 2 |

### Phase 2: Service Topology
| Code | Severity | Description | Resolution |
|------|----------|-------------|------------|
| `SYS-P2-001` | 🟠 WARNING | Orphan service detected (no dependencies) | Verify isolation is intentional |
| `SYS-P2-002` | 🟠 WARNING | Circular dependency detected | Flag for architectural review |
| `SYS-P2-003` | 🟠 WARNING | Dependency target not in inventory | Add missing project or mark external |
| `SYS-P2-004` | 🟡 INFO | Service topology generated | Continue to Phase 3 |

### Phase 3: Cross-Service APIs
| Code | Severity | Description | Resolution |
|------|----------|-------------|------------|
| `SYS-P3-001` | 🟠 WARNING | API contract mismatch (caller != provider) | Flag for resolution |
| `SYS-P3-002` | 🟠 WARNING | Missing api-contracts.json for service | Run /map-codebase-agent |
| `SYS-P3-003` | 🟡 INFO | Cross-service APIs documented | Continue to Phase 4 |

### Phase 4: Unified Domain Model
| Code | Severity | Description | Resolution |
|------|----------|-------------|------------|
| `SYS-P4-001` | 🟠 WARNING | Entity conflict detected (same name, different fields) | Flag canonical source |
| `SYS-P4-002` | 🟠 WARNING | Entity duplicate detected (redundant definitions) | Recommend consolidation |
| `SYS-P4-003` | 🟡 INFO | Domain model unified | Continue to Phase 5 |

### Phase 5: Generate System Documentation
| Code | Severity | Description | Resolution |
|------|----------|-------------|------------|
| `SYS-P5-001` | 🟠 WARNING | Mermaid diagram syntax error | Simplify diagram |
| `SYS-P5-002` | 🟠 WARNING | ASCII diagram generation failed | Use simplified layout |
| `SYS-P5-003` | 🟡 INFO | System documentation generated | Continue to Phase 6 |

### Phase 6: Generate Interactive Viewer
| Code | Severity | Description | Resolution |
|------|----------|-------------|------------|
| `SYS-P6-001` | 🟠 WARNING | HTML template not found | Use default template |
| `SYS-P6-002` | 🟠 WARNING | Mermaid rendering error in viewer | Check diagram syntax |
| `SYS-P6-003` | 🟡 INFO | Interactive viewer generated | Continue to Phase 7 |

### Phase 7: Final Verification
| Code | Severity | Description | Resolution |
|------|----------|-------------|------------|
| `SYS-P7-001` | 🔴 BLOCKING | Required artifact missing | Return to generating phase |
| `SYS-P7-002` | 🔴 BLOCKING | JSON validation failed | Re-generate artifact |
| `SYS-P7-003` | 🟠 WARNING | Browser verification failed (viewer not rendering) | Check HTML file |
| `SYS-P7-004` | 🟡 INFO | All verification passed | Workflow complete |

---

## Manager Agent (MGR)

### Beat Mode
| Code | Severity | Description | Resolution |
|------|----------|-------------|------------|
| `MGR-BT-001` | 🔴 BLOCKING | Jira query failed (MCP error) | Check Jira connectivity |
| `MGR-BT-002` | 🔴 BLOCKING | No active sprint found | Verify sprint exists and is active |
| `MGR-BT-003` | 🟠 WARNING | Sprint data incomplete (missing story points) | Calculate by issue count |
| `MGR-BT-004` | 🟡 INFO | Beat report generated | Output ready |

### Risk Mode
| Code | Severity | Description | Resolution |
|------|----------|-------------|------------|
| `MGR-RS-001` | 🔴 BLOCKING | Jira query failed | Check Jira connectivity |
| `MGR-RS-002` | 🟠 WARNING | Blocked issues exceed threshold | Flag for escalation |
| `MGR-RS-003` | 🟠 WARNING | Review stalls detected | Flag for follow-up |
| `MGR-RS-004` | 🟡 INFO | Risk assessment complete | Output ready |

### Status Mode
| Code | Severity | Description | Resolution |
|------|----------|-------------|------------|
| `MGR-ST-001` | 🔴 BLOCKING | Jira query failed | Check Jira connectivity |
| `MGR-ST-002` | 🟠 WARNING | Status data incomplete | Mark as partial report |
| `MGR-ST-003` | 🟡 INFO | Status report generated | Output ready |

### Retro Mode
| Code | Severity | Description | Resolution |
|------|----------|-------------|------------|
| `MGR-RT-001` | 🔴 BLOCKING | No closed sprint found | Verify sprint is closed |
| `MGR-RT-002` | 🟠 WARNING | Velocity data unavailable | Use issue count fallback |
| `MGR-RT-003` | 🟡 INFO | Retro insights generated | Output ready |

---

## MCP Tool Errors (Common)

| Code | Severity | Description | Resolution |
|------|----------|-------------|------------|
| `MCP-ATL-001` | 🔴 BLOCKING | Atlassian authentication failed | Re-authenticate MCP server |
| `MCP-ATL-002` | 🔴 BLOCKING | Jira issue not found | Verify issue key |
| `MCP-ATL-003` | 🟠 WARNING | Confluence page fetch returned truncated content | Use browser fallback |
| `MCP-ATL-004` | 🟠 WARNING | Jira create/update failed (validation error) | Check field requirements |
| `MCP-FIG-001` | 🔴 BLOCKING | Figma authentication failed | Re-authenticate MCP server |
| `MCP-FIG-002` | 🟠 WARNING | Figma node not found | Verify node ID from URL |
| `MCP-FIG-003` | 🟠 WARNING | Figma variable extraction failed | Use design context fallback |
| `MCP-GEN-001` | 🟠 WARNING | MCP tool timeout | Retry with smaller request |
| `MCP-GEN-002` | 🔴 BLOCKING | MCP server unavailable | Check MCP configuration |

---

## Usage Protocol

### 1. Emitting Errors

When a workflow encounters a failure, emit the error in this format:

```markdown
> ⚠️ **Error `MCB-P3-001`**: API endpoint extraction coverage <100%
> **Severity**: 🔴 BLOCKING
> **Details**: 3 of 5 controllers not documented
> **Resolution**: List undocumented controllers: `UserController`, `PaymentController`, `WebhookController`
```

### 2. Error Aggregation

At workflow completion (success or failure), output an error summary:

```markdown
## Workflow Error Summary

| Code | Severity | Count | Status |
|------|----------|-------|--------|
| `MCB-P2-002` | 🟠 WARNING | 3 | Logged |
| `MCB-P3-003` | 🟠 WARNING | 1 | Logged |
| `MCB-P45-001` | 🔴 BLOCKING | 1 | **HALTED** |

**Conclusion**: Workflow halted at Phase 4.5. Return to Phase 3 to document remaining controllers.
```

### 3. Automation Hooks

Error codes enable automation:

```javascript
// Example: Auto-retry logic
if (errorCode === "MCP-GEN-001") {
  await retry({ timeout: timeout * 2 });
}

// Example: Alert on blocking errors
if (severity === "BLOCKING") {
  await notify({ channel: "eng-alerts", message: `Workflow blocked: ${errorCode}` });
}
```

---

## Adding New Error Codes

When adding a new error code:

1. **Follow the naming convention**: `[AGENT]-[PHASE/MODE]-[NUMBER]`
2. **Assign severity**: BLOCKING (🔴), WARNING (🟠), or INFO (🟡)
3. **Write clear resolution**: What should the user/agent do?
4. **Add to the appropriate section**: Keep codes organized by agent and phase

---

## Error Code Index

Quick lookup by agent:

| Agent | Prefix | Code Range |
|-------|--------|------------|
| Map Codebase | `MCB-` | `MCB-P0-xxx` through `MCB-P6-xxx` |
| Engineering | `ENG-` | `ENG-CTX-xxx`, `ENG-PSR-xxx`, `ENG-FP-xxx`, etc. |
| System Architecture | `SYS-` | `SYS-P0-xxx` through `SYS-P7-xxx` |
| Manager | `MGR-` | `MGR-BT-xxx`, `MGR-RS-xxx`, `MGR-ST-xxx`, `MGR-RT-xxx` |
| MCP Tools | `MCP-` | `MCP-ATL-xxx`, `MCP-FIG-xxx`, `MCP-GEN-xxx` |
