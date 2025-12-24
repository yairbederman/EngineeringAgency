# Shared Configuration

> **Single Source of Truth**: All agents reference this file for global settings and the project list.
> When adding/removing projects or changing global constants, update **only this file**.

---

## Global Constants

| Variable | Value | Description |
|----------|-------|-------------|
| `SYSTEM_NAME` | `WG3` | Global system identifier |
| `SYSTEM_ARCH_OUTPUT_ROOT` | `${WORKSPACE_ROOT}/${SYSTEM_NAME}-system-architecture` | Contract path for system architecture artifacts |
| `GLOBAL_WORKFLOWS_ROOT` | `.` | Root path for global_workflows directory |

---

## Atlassian Configuration

> **Cloud ID**: `lognetsystems.atlassian.net` (Use `mcp0_getAccessibleAtlassianResources` to resolve UUID if needed)

| Variable | Value | Description |
|----------|-------|-------------|
| `${ATLASSIAN_CLOUD_ID}` | `lognetsystems.atlassian.net` | Base Cloud ID |
| `${JIRA_PROJECT_KEY}` | `W0` | Default Project Key |
| `${CONFLUENCE_SPACE_KEY}` | `WGPro30` | Default Space Key |

---

## Configuration

> [!IMPORTANT]
> **Team Setup Required**: Each team member must set `WORKSPACE_ROOT` to their local projects directory.

| Variable | Description | Value |
|----------|-------------|---------|
| `${WORKSPACE_ROOT}` | Root directory containing all projects | `C:/My Projects/WG3` |

---

## Registered Projects

| Variable | Name | Type | Role | Path |
|----------|------|------|------|------|
| `${PROJECT_FRONTEND}` | wg-client | Frontend | Next.js web application | `${WORKSPACE_ROOT}/wg-client` |
| `${PROJECT_CMS_API}` | wg-cms-api | Backend | CMS content management | `${WORKSPACE_ROOT}/wg-cms-api` |
| `${PROJECT_DATA_API}` | wg-data-api | Backend | Site data and configuration | `${WORKSPACE_ROOT}/wg-data-api` |
| `${PROJECT_ANCILLARY_API}` | wg-ancillary-api | Backend | Ancillary products & services | `${WORKSPACE_ROOT}/wg-ancillary-api` |
| `${PROJECT_ORDERMANAGER_API}` | wg-ordermanager-api | Backend | Order processing & management | `${WORKSPACE_ROOT}/wg-ordermanager-api` |
| `${PROJECT_PAYMENT_API}` | wg-payment-api | Backend | Payment transactions | `${WORKSPACE_ROOT}/wg-payment-api` |
| `${PROJECT_SEARCH_API}` | wg-search-api | Backend | Search functionality | `${WORKSPACE_ROOT}/wg-search-api` |
| `${PROJECT_TRIPDETAILS_API}` | wg-tripdetails-api | Backend | Trip details & itinerary | `${WORKSPACE_ROOT}/wg-tripdetails-api` |
| `${PROJECT_BOOKING_API}` | wg-booking-api | Backend | Booking processing & management | `${WORKSPACE_ROOT}/wg-booking-api` |
| `${PROJECT_EMAIL_API}` | wg-email-api | Backend | Email notifications & templates | `${WORKSPACE_ROOT}/wg-email-api` |
| `${PROJECT_INVOICE_API}` | wg-invoice-api | Backend | Invoice generation & management | `${WORKSPACE_ROOT}/wg-invoice-api` |
| `${PROJECT_PREORDER_API}` | wg-preorder-api | Backend | Pre-order processing & management | `${WORKSPACE_ROOT}/wg-preorder-api` |
| `${PROJECT_LTS_CORE}` | lts-core | Library | Core shared utilities & integrations | `${WORKSPACE_ROOT}/lts-core` |

---

## Adding New Projects

1. Add a row to the table above
2. Run `/map-codebase-agent` on the new project
3. Run `/system-architecture-agent` to update cross-project docs

---

## Project Selection Criteria

| Work Type | Use Project Variable |
|-----------|---------------------|
| Frontend work (UI, forms, displays) | `${PROJECT_FRONTEND}` |
| CMS/Admin operations | `${PROJECT_CMS_API}` |
| Data processing, external APIs | `${PROJECT_DATA_API}` |
| Ancillary products & services | `${PROJECT_ANCILLARY_API}` |
| Order processing & management | `${PROJECT_ORDERMANAGER_API}` |
| Payment transactions & integration | `${PROJECT_PAYMENT_API}` |
| Search functionality & indexing | `${PROJECT_SEARCH_API}` |
| Trip details & itinerary service | `${PROJECT_TRIPDETAILS_API}` |
| Booking processing & management | `${PROJECT_BOOKING_API}` |
| Email notifications & templates | `${PROJECT_EMAIL_API}` |
| Invoice generation & management | `${PROJECT_INVOICE_API}` |
| Full-stack features | Multiple projects |

---

## Used By

- `/engineering-agent` – for project selection during TechSpec/TaskPlanning
- `/system-architecture-agent` – for scanning all projects
- `/map-codebase-agent` – target project validation
