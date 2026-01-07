# System Architecture – Cross-Project Context Protocol

> **Load When**: Multi-project features, TechSpec, TaskPlanning

**Purpose**: Understand service dependencies and cross-project impacts for multi-project features.

## When to Use

Use System Architecture when:
- In **TechSpec** mode: To identify which services are impacted
- In **TaskPlanning** mode: To get cross-service API contracts
- When feature involves multiple projects

---

## Key Files

From `/system-architecture-agent` output:

| File | Purpose | When to Use |
|------|---------|-------------|
| `${SYSTEM_ARCH_OUTPUT}/analysis/service-topology.json` | Service dependencies | TechSpec Step 2 (identify impacted services) |
| `${SYSTEM_ARCH_OUTPUT}/analysis/cross-service-apis.json` | Inter-service API contracts | TaskPlanning (populate API context) |
| `${SYSTEM_ARCH_OUTPUT}/analysis/unified-domain-model.json` | Canonical entity sources | TechSpec § 5.2 (entity definitions) |
| `${SYSTEM_ARCH_OUTPUT}/system-architecture.md` | Overview with diagrams | Initial context gathering |

---

## Cross-Project Impact Protocol

1. **Read `service-topology.json`** to understand which services call which
2. **For each service in Epic scope**: Check if it calls or is called by other services
3. **If cross-service calls exist**: Read `cross-service-apis.json` for the API contracts
4. **For entity references**: Check `unified-domain-model.json` for canonical source

---

## Tool Failure

- If system architecture files don't exist: Recommend running `/system-architecture-agent` first
- Surface this as a **warning**, not a blocker (per-project work can proceed)

---

## Auto-Injection

For automatic cross-service API context injection, see:
`${AGENT_ROOT}/modes/planning/_cross-service-injection.md`
