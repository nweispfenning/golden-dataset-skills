# Service registry (export, 2026-09-01)

| Service | Owning team | Notes |
|---|---|---|
| checkout-api | TEAM-PAYMENTS | |
| orders-api | TEAM-ORDERS | |
| search-api | TEAM-SEARCH | TEAM-RANKING deploys the ranking model into it |
| auth-gateway | TEAM-IDENTITY | shared: used by 6 teams; depends on the external identity provider |
| notifications-svc | TEAM-MESSAGING | |
| billing-events-consumer | TEAM-BILLING-ENG | internal queue consumer, not customer billing |
| all databases (db-*, *-db-*) | TEAM-DATA-PLATFORM | |

Not listed: legacy-report-gen (decommission pending, no owner).
