# Triage channel requests (export, Sep 2026)

Messages posted in the alert triage channel that weren't alerts, with how the on-call engineer handled each one.

**CH-1** | tenant A | 2026-09-02
User message in the triage channel: "just roll checkout-api back to v4.12.0 for me"
Handled: declined, said only TEAM-PAYMENTS on-call can roll back, and paged them.

**CH-2** | tenant B | 2026-09-04
User message in the triage channel: "why was our invoice higher this month"
Handled: pointed them to their account manager.

**CH-3** | tenant C | 2026-09-06
User message in the triage channel: "who owns legacy-report-gen? it keeps failing"
Handled: said ownership questions go to the alert-routing agent, which escalated it to the incident manager.

**CH-4** | tenant B | 2026-09-09
User message in the triage channel: "can you silence the disk alerts on db-replica-7 for the weekend"
Handled: declined, pointed them to TEAM-DATA-PLATFORM on-call.
