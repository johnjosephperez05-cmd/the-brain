# Window Valet — Engagement Status (as of 2026-09-02)

## Roadmap context
8-week GHL stabilization engagement for The Window Valet (Josh LeClair). Weeks 1-2 closed (audit, Drip/Nurture fixes, Not Booked workflow, webhook validation, round-robin replaced with 3-way SMS broadcast, duplicate-opportunity setting off, Lead Response & Admin Accountability system built). Currently deep in Week 3-4.

## Week 3-4 status
7 of 10 original priorities done: Phone recording/voicemail (native GHL, 5 accounts, transcription on), Reply Needed/communications system, Lead-response accountability view, Cancellation/No-Show recovery, Owner management dashboard, Marketing attribution testing (clean, no fix needed), and Lead-response accountability/escalation.

Not started yet: Workflow cleanup & documentation (full pass), Week 7 Partner nurture campaign, Week 8 AI & Advanced Automation (confirmed the GHL plan already includes most AI features, so no plan-tier blocker there).

## Major incident this week: "1. New Lead" false-lead bug
Existing customers who touched a lead-gen form again (e.g. calling to confirm an already-booked appointment) were getting the full new-lead treatment: new Opportunity created, Renee alerted, welcome messages sent. Root cause: no exclusion check for existing customers at the top of "1. New Lead." Fixed with a Condition branch checking existing-customer tags (`qbo customers`, `myblindco import`) OR whether the contact has ever been through "1. Appointment Booked" workflow. Also hardened Drip/Nurture with a "Booked?" check before every email step. Cleaned up 79 contacts found with duplicate opportunities in the same pipeline (Lead Pipeline, Sales Pipeline, and the legacy Lead Flow Pipeline) — user reviewed manually.

## Pipeline audit findings
Ran a full opportunity audit via the GHL API. Key findings:
- Proposal Sent stage: 29 opportunities, 28 stuck untouched >30 days (oldest 97 days). Built a fix: "3. Proposal Sent" workflow now has a 3-touch SMS follow-up (day 3/7/14) + escalation task to the deal owner, with a sold/not-sold check before every step, plus a "Proposal Decided, Exit" companion workflow.
- Two legacy pipelines still holding live data: "Customer Order Flow Pipeline" (40 opps) and "Lead Flow Pipeline" (~180 opps), both predate the newer "(NEW)" pipelines built in May. ~148 records across both are still marked "open" despite 150+ days of no activity. Recommendation sent to Josh: preserve/archive the closed ones now, spot-check the open ones for any genuinely-forgotten customer before archiving those too. Still waiting on that spot-check + Josh's final call.

## New top priority: Customer Journey & Cross-System Integration
Josh elevated this to one of the most important projects (raised 2026-08-29). Goal: follow a customer from lead through to financial result across GHL (front end/CRM), MyBlindCo (post-sale: quotes, orders, installs), QuickBooks (financials), and Mapsly (geographic intelligence). Explicit ask: map first, build later, staged approach.

Research/findings so far:
- **QuickBooks ↔ GHL**: native integration exists in GHL (Settings → Integrations → QB Connect). Confirmed **not yet connected** in this account. Just needs setup, no custom work.
- **Mapsly ↔ GHL**: native, bi-directional, real-time HighLevel connector exists. Confirmed account is an **active Enterprise plan, 3 seats, $210/mo**. Usage is very low (96 of 2,400 routing credits, 0 AI credits, 0 transcription minutes), so there's real paid capacity sitting unused.
- **MyBlindCo ↔ GHL**: this was the genuinely custom unknown. Emailed MyBlindCo support directly. Audra McGrath (COO) confirmed **a direct API integration between MBC and GHL is available, independent of their "my5starz" white-labeled environment** (which Josh explicitly declined — Window Valet keeps its own independently-built GHL account). The direct API syncs contact data, appointment data, contact status, and order status into GHL so drip campaigns/automations can trigger correctly. There's a monthly fee, amount still TBD. Reply drafted asking for pricing + a Zoom demo, not yet confirmed sent.
- Existing one-way custom webhook ("My Blind Co, GHL Erica/Justin" workflow) already pushes some data GHL→MyBlindCo; confirmed it does not send SMS. Exact fields it sends still need to be pulled from the workflow builder.

**Important design constraint from Josh (2026-08-29):** a previous team built a two-way GHL↔MyBlindCo *scheduling* sync that was shut down due to double-booking — a cancellation on one side wouldn't reliably cancel on the other. That's why sales appointments (GHL) and installations (MyBlindCo) are deliberately kept on separate calendars today. Design principle going forward: **no calendar writes crossing between GHL and MyBlindCo.** Any scheduling intelligence (drive time, workload, better appointment suggestions) should be read-only, using Mapsly to pull data from both systems and suggest, never auto-write into either calendar.

Deliverables Josh explicitly wants for this project (documented, not all built yet): full customer-journey map, cross-system customer-matching approach (email/phone/job number/etc.), a data ownership map, a recommended integration architecture, a recommended Phase 1 (his own assumption: GHL opportunity → MyBlindCo quote/order first, QuickBooks after), and the beginning of a reporting-structure design (lead source, salesperson, closing %, gross profit, ZIP code, etc. — define where each number comes from, not the dashboard itself yet).

A PDF roadmap document covering all of this (plus Workflow Cleanup/Documentation) lives at `the window valet/Week3-4_Not_Started_Roadmap.pdf` in the local project folder, kept updated in place as new findings land. A second PDF, `Week7-8_Not_Started_Roadmap.pdf`, covers the Partner System (Week 7, blocked on tag standardization) and AI & Advanced Automation (Week 8, not started).

## Other confirmed-done items this week
- LeadConnector mobile app confirmed installed for Renee, Erica, and Justin.
- Call recording live natively on all 5 GHL accounts (admin + 2 sales), transcription confirmed enabled. Who-can-listen permissions never got checked — user said leave it for now.
- Unread-conversation-badge behavior confirmed as intentional GHL design (opening a conversation doesn't mark it read; Mark as Read or replying does), not a bug — GHL support confirmed this directly.
- Investigated a duplicate reschedule text sent to a customer (Richard Hitchcock); ruled out all automations and the calendar's native notification, concluded it was sent manually twice by staff (Erica) — confirmed with her.

## Open items as of 2026-09-02
- Waiting on Audra/MyBlindCo for API integration pricing + Zoom demo scheduling
- Pull exact fields sent by the existing MyBlindCo webhook (My Blind Co, GHL Erica/Justin workflow)
- Connect QuickBooks natively in GHL settings
- Spot-check the ~148 "open" records in the two legacy pipelines before archiving them
- Draft the tag standardization proposal for Josh's sign-off (this blocks Week 7's partner nurture campaign)
