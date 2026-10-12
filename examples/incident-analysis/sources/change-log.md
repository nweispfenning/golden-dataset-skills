# Change log: deploys and flag changes (export, 2026-09-30)

| Date | Time | Service | Change | Team | Notes |
|---|---|---|---|---|---|
| 2026-08-02 | 10:15 | db-primary-3 | maintenance: minor version upgrade | TEAM-DATA-PLATFORM | |
| 2026-08-14 | 11:30 | orders-api | deploy v2.7.4 | TEAM-ORDERS | |
| 2026-08-14 | 14:02 | checkout-api | deploy v4.12.1 | TEAM-PAYMENTS | |
| 2026-08-19 | 09:55 | auth-gateway | deploy v5.3.2 | TEAM-IDENTITY | config only: log format |
| 2026-08-19 | 10:01 | auth-gateway | flag strict-token-audience: off to on, all tenants | TEAM-IDENTITY | |
| 2026-08-22 | 16:31 | orders-api | deploy v2.8.0 | TEAM-ORDERS | |
| 2026-08-27 | 09:40 | search-api | deploy v7.1.3 | TEAM-SEARCH | logging changes only |
| 2026-08-27 | 09:46 | search-api | ranking model rank-2026-08-27 | TEAM-RANKING | |
| 2026-08-29 | 11:09 | notifications-svc | flag batch-push-v2: 0% to 50% | TEAM-MESSAGING | |
| 2026-09-01 | 15:20 | billing-events-consumer | deploy v1.9.0 | TEAM-BILLING-ENG | |
| 2026-09-03 | 15:10 | checkout-api | deploy v4.13.0, Acme Corp cell only | TEAM-PAYMENTS | |
| 2026-09-08 | 12:58 | checkout-api | flag new-tax-engine: off to on, tenant A | TEAM-PAYMENTS | |
| 2026-09-15 | 20:05 | search-api | deploy v7.2.0 | TEAM-SEARCH | |
| 2026-09-17 | 07:30 | search-api | flag fast-path-cache: off to on | TEAM-SEARCH | |
| 2026-09-22 | 08:10 | notifications-svc | deploy v3.1.0 | TEAM-MESSAGING | |
| 2026-09-22 | 08:14 | notifications-svc | flag digest-mode: off to on | TEAM-MESSAGING | |
| 2026-09-23 | 16:45 | checkout-api | deploy v4.13.2 | TEAM-PAYMENTS | |
| 2026-09-24 | 09:00 | orders-api | deploy v2.8.3 | TEAM-ORDERS | |
