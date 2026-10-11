# Requirements Traceability Matrix (RTM)

AugBridge Commerce - Order Fulfillment Automation & Exception Monitoring

The RTM links each pain point to its business requirement, functional requirements, business rules, user stories and UAT test cases.

[← UAT Test Cases](05-uat-test-cases.md) · [README](../README.md)

**Source documents**

| ID | Document |
|---|---|
| AP, BR, BUS | [02_BRD](02-brd.md) |
| FR, NFR | [03_FRD](03-frd.md) |
| US | [04_User_Stories](04-user-stories.md) |
| TC | [05_UAT_Test_Cases](05-uat-test-cases.md) |

**Test status:** *Not run* = test written, not yet executed (the system has not been built). *N/A* = no test for this item.

## Contents

- [Forward traceability](#forward-traceability)
- [Non-functional requirements](#non-functional-requirements)
- [Business rule coverage](#business-rule-coverage)
- [Backward traceability](#backward-traceability)

## Forward traceability

Pain point → BR → FR → BUS → User story → Test cases

| Pain point | BR | BR priority | FR | FR priority | BUS | User story | Test cases | Test status | Release | Why this link |
|---|---|---|---|---|---|---|---|---|---|---|
| AP-01 | (none) | – | (none) | – | – | (none) | (none) | N/A | – | Out of scope (BRD §6.2). |
| AP-02 | (none) | – | (none) | – | – | (none) | (none) | N/A | – | Out of scope: product research (BRD §7.2). |
| AP-03 | BR-01 | Must | FR-01 | Must | – | US-01 | TC-01a | Not run | R1 | One view of the order line replaces checking several tools; CS finds all lines by Etsy Order ID. Issue cases are added in R4 under BR-07. |
| AP-03 | BR-01 | Must | FR-02 | Must | – | US-02 | TC-02a, TC-13a, TC-16a | Not run | R1 (Management: R3, Designer: R4) | A shared workspace needs role-based access, which also protects the mapping. TC-13a checks Management access; TC-16a checks Designer access. |
| AP-04 | BR-02 | Must | FR-03 | Must | – | US-03 | TC-03a, TC-04a | Not run | R1 | One shared Product Type-to-supplier mapping replaces choosing the supplier per order. TC-04a shows the supplier comes from the Product Type, not the SKU. |
| AP-04 | BR-02 | Must | FR-04 | Must | BUS-06 | US-04 | TC-04a, TC-04b, TC-04c | Not run | R1 | The supplier comes from SKU → Product Type → mapping; Supplier 3 is assigned per line only as a fallback. |
| AP-05 | BR-03 | Must | FR-05 | Must | – | US-05 | TC-05a, TC-05b, TC-05c | Not run | R1 | Orders enter Lark automatically, so nothing is retyped; the daily count check covers risk R-02. |
| AP-05 | BR-03 | Must | FR-06 | Must | – | US-06 | TC-06a, TC-06b | Not run | R1 | Lines without the required fulfillment data are stopped before sending, which removes most supplier questions. |
| AP-05 | BR-03 | Must | FR-07 | Must | BUS-06 | US-07 | TC-07a, TC-07b, TC-07c, TC-07d | Not run | R1 | Order data reaches Suppliers 1, 2 and 3 by API, not by typing; a failed send is logged and the line is not marked as sent. |
| AP-05 | BR-03 | Must | FR-08 | Should | – | US-08 | TC-08a | Not run | R3 | Supplier questions get a record and an answer. |
| AP-06, AP-08 | BR-04 | Must | FR-09 | Must | – | US-09 | TC-09a, TC-09b, TC-09c, TC-07c | Not run | R1 | Status and tracking come back by API and feed the queue. |
| AP-06, AP-08 | BR-04 | Must | FR-10, FR-11 | Must | – | US-10 | TC-10a, TC-10b, TC-10c, TC-10d, TC-10e | Not run | R1 | The queue shows only lines that need action, once per active problem (no duplicate X2 or X3); unassigned High items go to the Team Lead. |
| No direct pain point – To-Be control added under BR-04 (source: AP-06, AP-08). | BR-04 | Must | FR-12 | Should | BUS-01 | US-11 | TC-11a, TC-11b | Not run | R3 | Change and cancel requests use the same queue (X5) with rule BUS-01. |
| AP-07 | BR-05 | Must | FR-13 | Must | BUS-07 | US-12 | TC-12a, TC-12b | Not run | R1 | The first tracking number of an order reaches Etsy without copying; later ones stay in Lark. |
| AP-09 | BR-06 | Should | FR-14 | Should | – | US-13 | TC-13a | Not run | R3 (issue rate: R4) | Current figures replace the manual weekly report. |
| AP-10 | BR-07 | Should | FR-15 | Should | BUS-04, BUS-05 | US-14, US-16 | TC-14a, TC-14b, TC-16a | Not run | R4 | Every issue becomes a case with a cause and an owner, shown on its order line (R4). BUS-04 and BUS-05 are policies applied by CS; they are not tested in UAT. |
| AP-10 | BR-07 | Should | FR-16 | Should | BUS-02, BUS-03 | US-15, US-16 | TC-15a, TC-15b, TC-16a | Not run | R4 | Refund or replacement is recorded; a replacement is a new order line that gets its supplier from the mapping and is checked, sent and tracked. |

## Non-functional requirements

| NFR | Checked by | What is checked | Test status |
|---|---|---|---|
| NFR-01 | UAT: TC-05a, TC-13a<br>Technical check in R1: the Developer confirms the hourly schedule. | UAT checks order intake time and dashboard freshness by source. UAT starts the hourly check by hand, so it does not prove the schedule. The Developer confirms the schedule in R1. | Not run |
| NFR-02 | UAT: TC-05c, TC-07d, TC-09c, TC-12b | Missing orders, failed sends, failed supplier updates and failed tracking syncs are logged and shown. | Not run |
| NFR-03 | UAT: TC-10c, TC-14b | Status History for lines, exceptions and issue cases. | Not run |
| NFR-04 | Not in UAT – review before go-live. | Data policy agreed with the vendor before go-live. | N/A (not in UAT) |
| NFR-05 | Not in UAT – check before the pilot and in R5. | Plan limits checked before the pilot; peak volume checked in R5. | N/A (not in UAT) |

## Business rule coverage

| BUS | FR | Test cases | Test status | Coverage note |
|---|---|---|---|---|
| BUS-01 | FR-12 | TC-11a, TC-11b | Not run | Tested: TC-11a (before production), TC-11b (blocked after production starts). |
| BUS-02 | FR-16 | TC-15b | Not run | Tested: TC-15b (replacement blocked until the customer approves the new design). |
| BUS-03 | FR-16 | TC-15a | Not run | Tested: TC-15a (refund or replacement recorded on the case). |
| BUS-04 | FR-15 | – | N/A (policy) | Policy – not tested in UAT. CS applies each supplier's reprint and refund policy on Supplier cases. |
| BUS-05 | FR-15 | – | N/A (policy) | Policy – not tested in UAT. Customer cases stay with CS (TC-14b checks routing only, not the shop policy). |
| BUS-06 | FR-04, FR-07 | TC-04c, TC-07c | Not run | Tested: TC-04c (Supplier 3 assigned to one line with a fallback reason), TC-07c (Supplier 3 sent by API). |
| BUS-07 | FR-13 | TC-12a | Not run | Tested: TC-12a (first tracking to Etsy, later tracking stays in Lark). |

## Backward traceability

Test case → User story → FR → BR → Pain point

| Test case | User story | FR | BR | Pain point | Also verifies | Priority | Release | Test status |
|---|---|---|---|---|---|---|---|---|
| TC-01a | US-01 | FR-01 | BR-01 | AP-03 | – | P1 | R1 | Not run |
| TC-02a | US-02 | FR-02 | BR-01 | AP-03 | – | P1 | R1 | Not run |
| TC-03a | US-03 | FR-03 | BR-02 | AP-04 | – | P1 | R1 | Not run |
| TC-04a | US-03, US-04 | FR-03, FR-04 | BR-02 | AP-04 | – | P1 | R1 | Not run |
| TC-04b | US-04 | FR-04 | BR-02 | AP-04 | – | P1 | R1 | Not run |
| TC-04c | US-04 | FR-04 | BR-02 | AP-04 | BUS-06 | P1 | R1 | Not run |
| TC-05a | US-05 | FR-05 | BR-03 | AP-05 | NFR-01 | P1 | R1 | Not run |
| TC-05b | US-05 | FR-05 | BR-03 | AP-05 | – | P1 | R1 | Not run |
| TC-05c | US-05 | FR-05 | BR-03 | AP-05 | NFR-02 | P1 | R1 | Not run |
| TC-06a | US-06 | FR-06 | BR-03 | AP-05 | – | P1 | R1 | Not run |
| TC-06b | US-06 | FR-06 | BR-03 | AP-05 | – | P1 | R1 | Not run |
| TC-07a | US-07 | FR-07 | BR-03 | AP-05 | – | P1 | R1 | Not run |
| TC-07b | US-07 | FR-07 | BR-03 | AP-05 | – | P1 | R1 | Not run |
| TC-07c | US-07, US-09 | FR-07, FR-09 | BR-03, BR-04 | AP-05, AP-06, AP-08 | BUS-06 | P1 | R1 | Not run |
| TC-07d | US-07 | FR-07 | BR-03 | AP-05 | NFR-02 | P1 | R1 | Not run |
| TC-08a | US-08 | FR-08 | BR-03 | AP-05 | – | P1 | R3 | Not run |
| TC-09a | US-09 | FR-09 | BR-04 | AP-06, AP-08 | – | P1 | R1 | Not run |
| TC-09b | US-09 | FR-09 | BR-04 | AP-06, AP-08 | – | P1 | R1 | Not run |
| TC-09c | US-09 | FR-09 | BR-04 | AP-06, AP-08 | NFR-02 | P1 | R1 | Not run |
| TC-10a | US-10 | FR-10, FR-11 | BR-04 | AP-06, AP-08 | – | P1 | R1 | Not run |
| TC-10b | US-10 | FR-10, FR-11 | BR-04 | AP-06, AP-08 | – | P1 | R1 | Not run |
| TC-10c | US-10 | FR-10, FR-11 | BR-04 | AP-06, AP-08 | NFR-03 | P1 | R1 | Not run |
| TC-10d | US-10 | FR-10, FR-11 | BR-04 | AP-06, AP-08 | – | P1 | R1 | Not run |
| TC-10e | US-10 | FR-10, FR-11 | BR-04 | AP-06, AP-08 | – | P1 | R1 | Not run |
| TC-11a | US-11 | FR-12 | BR-04 | (To-Be control) | BUS-01 | P1 | R3 | Not run |
| TC-11b | US-11 | FR-12 | BR-04 | (To-Be control) | BUS-01 | P1 | R3 | Not run |
| TC-12a | US-12 | FR-13 | BR-05 | AP-07 | BUS-07 | P1 | R1 | Not run |
| TC-12b | US-12 | FR-13 | BR-05 | AP-07 | NFR-02 | P1 | R1 | Not run |
| TC-13a | US-02, US-13 | FR-02, FR-14 | BR-01, BR-06 | AP-03, AP-09 | NFR-01 | P2 | R3 (issue rate: R4) | Not run |
| TC-14a | US-14, US-16 | FR-15 | BR-07 | AP-10 | – | P1 | R4 | Not run |
| TC-14b | US-14, US-16 | FR-15 | BR-07 | AP-10 | NFR-03 | P1 | R4 | Not run |
| TC-15a | US-15, US-16 | FR-16 | BR-07 | AP-10 | BUS-03 | P1 | R4 | Not run |
| TC-15b | US-15, US-16 | FR-16 | BR-07 | AP-10 | BUS-02 | P1 | R4 | Not run |
| TC-16a | US-02, US-14, US-16, US-15 | FR-02, FR-15, FR-16 | BR-01, BR-07 | AP-03, AP-10 | – | P1 | R4 | Not run |
