# Incident postmortems (export, Aug-Sep 2026)

There is no agent yet. On-call engineers investigated each alert by hand. Each postmortem records the alert as it fired, the confirmed cause, the evidence, and how long the cause took to find.

**PM-101** | tenant A | 2026-08-14
Alert: p99 latency on checkout-api > 2s for 5m. Fired 14:11.
Confirmed cause: the checkout-api v4.12.1 deploy at 14:02 added a synchronous fraud-score call to the payment path.
Evidence: p99 rose from 310 ms to 2.4 s starting 14:04. No flag changes on checkout-api that day.
Time to cause: 6 min. TEAM-PAYMENTS rolled back to v4.12.0 at 14:26.

**PM-102** | tenant C | 2026-08-16
Alert: disk 91% on db-primary-3. Fired 06:40.
Confirmed cause: no deploy or flag change in the window. A nightly export job's output doubled after a large tenant was onboarded on 08-15.
Evidence: disk growth jumped from 1% a day to 6% a day on 08-15. The last change to db-primary-3 was on 08-02.
Time to cause: 52 min. The first responder blamed the 08-02 upgrade until the growth rate ruled it out. TEAM-DATA-PLATFORM moved the export to object storage.

**PM-103** | tenant B | 2026-08-19
Alert: 5xx rate 9% on auth-gateway. Fired 10:07.
Confirmed cause: the strict-token-audience flag, turned on for all tenants at 10:01, rejected tokens from mobile client 3.x. The auth-gateway v5.3.2 deploy at 09:55 changed only the log format and was unrelated.
Evidence: 5xx started at 10:02. All errors were 401s from mobile client 3.x.
Time to cause: 38 min. The deploy was rolled back first, with no effect. TEAM-IDENTITY turned the flag off at 10:45.

**PM-104** | tenant A | 2026-08-22
Alert: 5xx rate 6% on orders-api. Fired 16:40.
Confirmed cause: the orders-api v2.8.0 deploy at 16:31 changed an order-history query that timed out under load.
Evidence: 5xx started at 16:33, all database timeouts. No flag changes on orders-api that day.
Time to cause: 9 min. TEAM-ORDERS rolled back at 16:52.

**PM-105** | tenant A | 2026-08-27
Alert: p95 latency on search-api > 800 ms for 5m. Fired 09:53.
Confirmed cause: the ranking model TEAM-RANKING deployed into search-api at 09:46 made each query several times slower. TEAM-SEARCH's v7.1.3 deploy at 09:40 changed only logging.
Evidence: p95 rose from 180 ms to 900 ms starting 09:47, one minute after the model deploy and seven minutes after v7.1.3.
Time to cause: 41 min. Each team suspected the other's change. TEAM-RANKING reverted the model at 10:30.

**PM-106** | tenant C | 2026-08-29
Alert: 5xx rate 4% on notifications-svc. Fired 11:15.
Confirmed cause: the batch-push-v2 flag, raised to 50% of traffic at 11:09, sent requests over 1 MB down a path that dropped them.
Evidence: errors started at 11:10, all on requests over 1 MB. No deploys to notifications-svc that day.
Time to cause: 8 min. TEAM-MESSAGING set the flag back to 0% at 11:31.

**PM-107** | Acme Corp | 2026-09-03
Alert: p99 latency on checkout-api > 2s for 5m on acme-checkout-02.acme.internal. Fired 15:22.
Confirmed cause: the checkout-api v4.13.0 deploy at 15:10, rolled out to Acme Corp's dedicated cell only, shipped with debug logging left on.
Evidence: p99 rose starting 15:12, and log volume on acme-checkout-02.acme.internal grew twentyfold. No flag changes on checkout-api that day.
Time to cause: 7 min. TEAM-PAYMENTS shipped v4.13.1 with the log level fixed at 15:41.

**PM-108** | tenant A | 2026-09-08
Alert: 5xx rate 3% on checkout-api. Fired 13:05.
Confirmed cause: the new-tax-engine flag, turned on for tenant A at 12:58, failed on orders with mixed tax regions.
Evidence: errors started at 12:59, all from the tax-calculation path. No deploys to checkout-api that day.
Time to cause: 10 min. TEAM-PAYMENTS turned the flag off at 13:20.

**PM-109** | tenant B | 2026-09-09
Alert: consumer lag 45 min on billing-events-consumer. Fired 03:30.
Confirmed cause: no deploy or flag change in the window. The monthly invoice batch tripled event volume.
Evidence: input rose from 2,000 to 6,500 events a second at 02:00, while throughput stayed near 2,200. The last change was the v1.9.0 deploy on 09-01.
Time to cause: 47 min. The consumers were restarted first, with no effect. TEAM-BILLING-ENG scaled the consumer group from 4 to 12 workers.

**PM-110** | tenant C | 2026-09-15
Alert: p95 latency on search-api > 800 ms for 5m. Fired 20:14.
Confirmed cause: the search-api v7.2.0 deploy at 20:05 removed a result cache.
Evidence: p95 rose from 170 ms to 1.1 s starting 20:06. No flag changes on search-api that day.
Time to cause: 5 min. TEAM-SEARCH rolled back at 20:30.

**PM-111** | tenant B | 2026-09-17
Alert: 5xx rate 5% on search-api. Fired 07:41.
Confirmed cause: the fast-path-cache flag, turned on at 07:30, returned cached results in an old format that clients rejected.
Evidence: errors started at 07:33, all format errors from the cache path. No deploys to search-api that day.
Time to cause: 11 min. TEAM-SEARCH turned the flag off at 07:58.

**PM-112** | tenant B | 2026-09-20
Alert: memory 97% on search-api pods. Fired 02:15.
Confirmed cause: none recorded. The alert cleared on its own at 02:40, and the postmortem was never finished.

**PM-113** | tenant C | 2026-09-22
Alert: 5xx rate 7% on notifications-svc. Fired 08:16.
Confirmed cause: the notifications-svc v3.1.0 deploy at 08:10 broke template rendering for non-English locales. The digest-mode flag turned on at 08:14 was unrelated.
Evidence: errors started at 08:12, before the flag change, all template errors on non-English locales.
Time to cause: 34 min. The flag was turned off first, with no effect. TEAM-MESSAGING rolled back at 08:50.

**PM-114** | tenant A | 2026-09-24
Alert: p99 latency on checkout-api > 2s for 5m. Fired 18:03.
Confirmed cause: no deploy or flag change in the window. A marketing email to tenant A's customers tripled checkout traffic.
Evidence: requests rose from 400 to 1,300 a second at 17:55. The last change to checkout-api was the v4.13.2 deploy on 09-23.
Time to cause: 44 min. The on-call engineer rolled back v4.13.2 first, with no effect. TEAM-PAYMENTS scaled checkout-api out.
