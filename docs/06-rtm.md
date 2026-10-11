# Requirements Traceability Matrix (RTM)

AugBridge Commerce - Order Fulfillment Automation & Exception Monitoring

Sources: [02_BRD](02-brd.md) (AP, BR, BUS), [03_FRD](03-frd.md) (FR, NFR), [04_User_Stories](04-user-stories.md) (US), [05_UAT_Test_Cases](05-uat-test-cases.md) (TC).

[← UAT Test Cases](05-uat-test-cases.md) · [README](../README.md)

> UAT has not been run; the system has not been built.

## Contents

- [Traceability](#traceability)
- [Non-functional requirements](#non-functional-requirements)
- [Business rule coverage](#business-rule-coverage)

## Traceability

Pain point → BR → FR → User story → Test cases

| Pain point | BR | BR priority | FR | FR priority | User story | Test cases | Release | Why this link |
|---|---|---|---|---|---|---|---|---|
| AP-01 | (none) | – | (none) | – | (none) | (none) | – | Out of scope (BRD §6.2). |
| AP-02 | (none) | – | (none) | – | (none) | (none) | – | Out of scope: product research (BRD §7.2). |
| AP-03 | BR-01 | Must | FR-01 | Must | US-01 | TC-01a | R1 | One view of the order line replaces checking several tools; CS finds all lines by Etsy Order ID. Issue cases are added in R4 under BR-07. |
| AP-03 | BR-01 | Must | FR-02 | Must | US-02 | TC-02a, TC-13a, TC-16a | R1 (Management: R3, Designer: R4) | A shared workspace needs role-based access, which also protects the mapping. TC-13a checks Management access; TC-16a checks Designer access. |
| AP-04 | BR-02 | Must | FR-03 | Must | US-03 | TC-03a, TC-04a | R1 | One shared Product Type-to-supplier mapping replaces choosing the supplier per order. TC-04a shows the supplier comes from the Product Type, not the SKU. |
| AP-04 | BR-02 | Must | FR-04 | Must | US-04 | TC-04a, TC-04b, TC-04c | R1 | The supplier comes from SKU → Product Type → mapping; Supplier 3 is assigned per line only as a fallback. |
| AP-05 | BR-03 | Must | FR-05 | Must | US-05 | TC-05a, TC-05b, TC-05c | R1 | Orders enter Lark automatically, so nothing is retyped; the daily count check covers risk R-02. |
| AP-05 | BR-03 | Must | FR-06 | Must | US-06 | TC-06a, TC-06b | R1 | Lines without the required fulfillment data are stopped before sending, which removes most supplier questions. |
| AP-05 | BR-03 | Must | FR-07 | Must | US-07 | TC-07a, TC-07b, TC-07c, TC-07d | R1 | Order data reaches Suppliers 1, 2 and 3 by API, not by typing; a failed send is logged and the line is not marked as sent. |
| AP-05 | BR-03 | Must | FR-08 | Should | US-08 | TC-08a | R3 | Supplier questions get a record and an answer. |
| AP-06, AP-08 | BR-04 | Must | FR-09 | Must | US-09 | TC-09a, TC-09b, TC-09c, TC-07c | R1 | Status and tracking come back by API and feed the queue. |
| AP-06, AP-08 | BR-04 | Must | FR-10, FR-11 | Must | US-10 | TC-10a, TC-10b, TC-10c, TC-10d, TC-10e | R1 | The queue shows only lines that need action, once per active problem (no duplicate X2 or X3); unassigned High items go to the Team Lead. |
| No direct pain point – To-Be control added under BR-04 (source: AP-06, AP-08). | BR-04 | Must | FR-12 | Should | US-11 | TC-11a, TC-11b | R3 | Change and cancel requests use the same queue (X5) with rule BUS-01. |
| AP-07 | BR-05 | Must | FR-13 | Must | US-12 | TC-12a, TC-12b | R1 | The first tracking number of an order reaches Etsy without copying; later ones stay in Lark. |
| AP-09 | BR-06 | Should | FR-14 | Should | US-13 | TC-13a | R3 (issue rate: R4) | Current figures replace the manual weekly report. |
| AP-10 | BR-07 | Should | FR-15 | Should | US-14, US-16 | TC-14a, TC-14b, TC-16a | R4 | Every issue becomes a case with a cause and an owner, shown on its order line (R4). BUS-04 and BUS-05 are policies applied by CS; they are not tested in UAT. |
| AP-10 | BR-07 | Should | FR-16 | Should | US-15, US-16 | TC-15a, TC-15b, TC-16a | R4 | Refund or replacement is recorded; a replacement is a new order line that gets its supplier from the mapping and is checked, sent and tracked. |

## Non-functional requirements

| NFR | Checked by | What is checked |
|---|---|---|
| NFR-01 | UAT: TC-05a, TC-13a<br>Technical check in R1: the Developer confirms the hourly schedule. | UAT checks order intake time and dashboard freshness by source. UAT starts the hourly check by hand, so it does not prove the schedule. The Developer confirms the schedule in R1. |
| NFR-02 | UAT: TC-05c, TC-07d, TC-09c, TC-12b | Missing orders, failed sends, failed supplier updates and failed tracking syncs are logged and shown. |
| NFR-03 | UAT: TC-10c, TC-14b | Status History for lines, exceptions and issue cases. |
| NFR-04 | Not in UAT – review before go-live. | Data policy agreed with the vendor before go-live. |
| NFR-05 | Not in UAT – check before the pilot and in R5. | Plan limits checked before the pilot; peak volume checked in R5. |

## Business rule coverage

| BUS | FR | Test cases | Coverage note |
|---|---|---|---|
| BUS-01 | FR-12 | TC-11a, TC-11b | Tested: TC-11a (before production), TC-11b (blocked after production starts). |
| BUS-02 | FR-16 | TC-15b | Tested: TC-15b (replacement blocked until the customer approves the new design). |
| BUS-03 | FR-16 | TC-15a | Tested: TC-15a (refund or replacement recorded on the case). |
| BUS-04 | FR-15 | – | Policy – not tested in UAT. CS applies each supplier's reprint and refund policy on Supplier cases. |
| BUS-05 | FR-15 | – | Policy – not tested in UAT. Customer cases stay with CS (TC-14b checks routing only, not the shop policy). |
| BUS-06 | FR-04, FR-07 | TC-04c, TC-07c | Tested: TC-04c (Supplier 3 assigned to one line with a fallback reason), TC-07c (Supplier 3 sent by API). |
| BUS-07 | FR-13 | TC-12a | Tested: TC-12a (first tracking to Etsy, later tracking stays in Lark). |
