# Shared Project Registry

> **Single Source of Truth**: All agents reference this file for the project list.
> When adding/removing projects, update **only this file**.

---

## Registered Projects

| Variable | Name | Type | Role | Path |
|----------|------|------|------|------|
| `${PROJECT_FRONTEND}` | wg-client | Frontend | Next.js web application | `C:\My Projects\WG3\wg-client` |
| `${PROJECT_CMS_API}` | wg-cms-api | Backend | CMS content management | `C:\My Projects\WG3\wg-cms-api` |
| `${PROJECT_DATA_API}` | wg-data-api | Backend | Site data and configuration | `C:\My Projects\WG3\wg-data-api` |
| `${PROJECT_ANCILLARY_API}` | wg-ancillary-api | Backend | Ancillary products & services | `C:\My Projects\WG3\wg-ancillary-api` |
| `${PROJECT_ORDERMANAGER_API}` | wg-ordermanager-api | Backend | Order processing & management | `C:\My Projects\WG3\wg-ordermanager-api` |
| `${PROJECT_PAYMENT_API}` | wg-payment-api | Backend | Payment transactions | `C:\My Projects\WG3\wg-payment-api` |
| `${PROJECT_SEARCH_API}` | wg-search-api | Backend | Search functionality | `C:\My Projects\WG3\wg-search-api` |
| `${PROJECT_TRIPDETAILS_API}` | wg-tripdetails-api | Backend | Trip details & itinerary | `C:\My Projects\WG3\wg-tripdetails-api` |

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
| Full-stack features | Multiple projects |

---

## Used By

- `/engineering-agent` – for project selection during TechSpec/TaskPlanning
- `/system-architecture-agent` – for scanning all projects
- `/map-codebase-agent` – target project validation
