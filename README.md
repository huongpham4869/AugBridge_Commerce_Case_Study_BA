# AugBridge Commerce — Order Fulfillment & Exception Monitoring

**Business Analyst · Independent Portfolio Project** · Aug – Oct 2026

A Business Analysis case study on manual order fulfillment in a small cross-border e-commerce business that sells personalized print-on-demand products on Etsy. The process and pain points are based on my own work in this kind of business.

> [!NOTE]
> AugBridge Commerce is a fictional company and the order data is simulated. The proposed solution has not been built, the UAT test cases have not been run, and no savings or results are claimed.

**Short on time?** Read the [Project Summary](docs/01-project-summary.md) (11 slides), then the [RTM](docs/06-rtm.md) to see how each pain point traces to requirements and test cases.

## Overview

- **Business:** 6 Etsy shops, 3 external suppliers and a 9-person team
- **Volume:** about 400 orders a week, rising to about 2,800 in peak season
- **Question:** How can the team stop checking every order by hand and find the orders that need action earlier?
- **Output:** BRD, FRD, 16 user stories, 34 UAT test cases, RTM, and a SQL analysis of 10,685 simulated orders

## Business problem

Each order needs four manual steps: choosing a supplier, entering the order in the supplier's system, checking its status again and again, and copying the tracking number back to Etsy. The same four people handle both product development and customer support (PD&CS), so this work grows with every order.

Order, supplier and issue information sits in Etsy, three supplier systems, spreadsheets, notebooks and chat apps. There is no shared view of which orders are stuck, so in peak season late orders are often found too late. The Team Lead also prepares the weekly report by hand.

The BRD lists 10 pain points, 8 of them in scope ([BRD §3](docs/02-brd.md#3-problem-statement-and-pain-points)).

## Objectives and scope

The project has three objectives ([BRD §7](docs/02-brd.md#7-business-objectives-and-project-scope)):

1. Reduce repetitive order work.
2. Detect and handle order exceptions earlier.
3. Give the team one shared view of orders, suppliers and customer issues.

Fewer late orders is the expected outcome, but it also depends on supplier production and carrier time, so it is not used alone to judge the solution.

| In scope | Out of scope |
|---|---|
| Order intake from Etsy, supplier mapping, sending orders and receiving status and tracking by API, tracking sync to Etsy, the exception queue, issue cases and a dashboard | Product research and listing, splitting the PD&CS role, changes to supplier systems, automatic supplier re-routing and carrier delays |

## My role

This is an independent portfolio project. It is not client work and not part of my employment. I:

- mapped the As-Is process from my experience in print-on-demand operations
- defined the dataset scenario and the analysis questions, drafted the SQL queries, and ran and checked them
- carried out the root cause analysis and wrote the business requirements, business rules and KPIs
- compared five solution options and designed the To-Be process, exception rules and data model
- wrote the functional requirements, user stories, UAT test cases and the RTM

## Evidence and key findings

There is no central order database in the current process, so I analyzed a simulated 12-week dataset in SQLite: 10,685 orders, 6 shops, 3 suppliers and 15 SKUs, with weeks 8–11 as peak season.

| Weeks | Orders per week | Overdue rate |
|---|---|---|
| Normal (1–7, 12) | 380 – 520 | 5.2% – 7.1% |
| Peak (8–11) | 900 – 2,750 | 81.0% – 83.7% |

*Overdue = more than 10 business days from order to delivery. Query and full output: [BRD §5.1](docs/02-brd.md#51-sql-analysis-of-simulated-order-data) and [`sql/analysis_queries.sql`](sql/analysis_queries.sql).*

What the data shows, and what it does not:

- Peak weeks carry up to about 7 times the normal volume, and most peak orders are late in the simulation.
- The overdue rate follows the peak setting, not the volume: week 8 (900 orders) is 81.0% and week 10 (2,750 orders) is 81.7%.
- The data cannot separate internal handling, supplier production and carrier time, so it does not prove what causes the delays. The overdue rate is used as context, not as a KPI or a target.

For this reason, the project measures manual handling time per order and the exception workload instead. Each figure in the project has a clear type:

| Figure | Value | Type |
|---|---|---|
| Overdue rate, normal / peak weeks | 5.2–7.1% / 81.0–83.7% | Simulated (SQL output) |
| Weekly reporting effort today | 6–8 hours | Estimate |
| Reporting time the dashboard may save | About 5 hours a week | Estimate, not measured |
| Planning benchmark | 6.0 minutes per order (40 Developer hours × 60 ÷ 400 orders) | Decision aid, not a saving |
| Manual handling time per order | Not measured yet | Baseline in R0, measured again in the R2 pilot |

The 6.0-minute benchmark compares the planned Developer effort with the weekly order volume. If the pilot shows a saving below 6.0 minutes per order, time saving alone may not justify the effort, and management reviews the other benefits and the actual cost ([BRD §13](docs/02-brd.md#13-business-case)).

## Root cause and process analysis

[![As-Is order fulfillment process](assets/diagrams/as-is-2-order-fulfillment.png)](assets/diagrams/as-is-2-order-fulfillment.png)

*As-Is order fulfillment process. Select the image to open it at full size. All four As-Is processes are in [BRD §4](docs/02-brd.md#4-current-process-as-is).*

Why are late or stuck order lines found too late (AP-08)? The cause chain in [BRD §6.1](docs/02-brd.md#61-ap-08-late-detection-of-stuck-or-late-order-lines):

1. PD&CS checks order status by hand in each supplier system.
2. Status information is spread across Etsy, supplier systems and spreadsheets.
3. There is no shared status view and no rule that flags the order lines that need action.

The main gap is the missing shared view and exception rule, which BR-01 and BR-04 address. Manual work per order may also make peak workload hard to manage, but the data cannot confirm this, so it stays a working hypothesis for the pilot to test.

The combined PD&CS role (AP-01) also causes context switching. Splitting the role would not remove the repetitive order work, so it is out of scope.

## Proposed solution and decision rationale

The proposal uses Lark as one shared workspace for order fulfillment:

- Etsy orders arrive in Lark automatically, with one order line per item.
- A Product Type → preferred supplier mapping assigns Supplier 1 or 2. If the preferred supplier cannot fulfill a line, CS can move that line only to the backup, Supplier 3.
- Each line is checked for missing data, then sent to the supplier by API. Status and tracking come back by API, and the first tracking number is synced to Etsy.
- CS works from an exception queue (X1–X6) instead of checking every order.

[![To-Be order fulfillment process](assets/diagrams/to-be-1-order-fulfillment.png)](assets/diagrams/to-be-1-order-fulfillment.png)

*To-Be order fulfillment process. Green boxes are done by the system ([FRD §4](docs/03-frd.md#4-to-be-process)).*

**Why Lark.** I scored five options against eight weighted criteria taken from the requirements and constraints ([FRD §2](docs/03-frd.md#2-solution-evaluation-and-selection)).

| Option | Score (max 54) |
|---|---|
| A. Lark | **50** |
| B. Airtable + Slack + Zapier | 43 |
| D. Custom web app | 40 |
| C. Notion + Slack + Make | 35 |
| E. Better spreadsheets | 28 |

The main reason is fit with the problem: information is already spread across too many tools, and a multi-tool stack could repeat that. Lark also fits the cost for about 10 users and a one-Developer setup. The accepted trade-off is that Lark's API support is only adequate, so a small integration layer and failure alerts are needed. Automatic supplier re-routing was rejected because there is no supplier stock data.

**Key business rules** ([BRD §11](docs/02-brd.md#11-business-rules)):

- Customer changes or cancellations are accepted only before the supplier starts production (BUS-01).
- Supplier 3 is used for a single order line only, and the Product Type mapping does not change (BUS-06).
- The first tracking number received for an Etsy order is synced to Etsy; later ones stay in Lark (BUS-07).

**How success would be checked** ([FRD §11](docs/03-frd.md#11-release-plan)): R0 confirms API access and the supplier mapping, and measures the baseline. R1 builds the core flow. The R2 pilot runs on 1–2 shops and compares manual time per order with the R0 baseline and the 6.0-minute benchmark; management then decides to continue, adjust or stop. R3–R5 add the rest of the queue, issue cases and a peak-season review. No targets are set before the pilot.

## Key BA artifacts

| # | Document | What to look at |
|---|---|---|
| 01 | [Project Summary](docs/01-project-summary.md) | The whole case in 11 slides |
| 02 | [BRD](docs/02-brd.md) | Pain points, As-Is processes, SQL evidence, root cause, scope, business requirements, business rules, KPIs, benchmark and risks |
| 03 | [FRD](docs/03-frd.md) | Solution scoring, To-Be processes, roles, exception rules, integration, data model, functional requirements, wireframes and release plan |
| 04 | [User Stories](docs/04-user-stories.md) | 16 stories with Given / When / Then acceptance criteria |
| 05 | [UAT Test Cases](docs/05-uat-test-cases.md) | 34 test cases (33 P1, 1 P2), including 13 negative cases. Not run. |
| 06 | [RTM](docs/06-rtm.md) | Pain point → business requirement → functional requirement → user story → test case, with the reason for each link |
| – | [`data/`](data) and [`sql/`](sql) | Simulated dataset, data dictionary, schema and queries with their output |

One trace from the RTM:

| Pain point | Business requirement | Functional requirements | User stories | Test cases |
|---|---|---|---|---|
| AP-04: supplier selection is done by hand for each order | BR-02: Product Type → preferred supplier mapping | FR-03, FR-04 | US-03, US-04 | TC-03a, TC-04a, TC-04b, TC-04c |

[![Wireframe: Needs action view and order search in Lark Base](assets/wireframes/needs-action-and-order-search.png)](assets/wireframes/needs-action-and-order-search.png)

*Wireframe of the exception queue and order search, with illustrative data. It shows how FR-01, FR-10 and FR-11 could look in Lark; it is not a built screen ([FRD §10](docs/03-frd.md#10-wireframes)).*

## Tools and techniques

- **Tools:** SQL (SQLite), Excel, Word and PowerPoint for the original documents, swimlane process diagrams and wireframes
- **Techniques:** As-Is / To-Be process mapping, root cause analysis, weighted solution scoring, MoSCoW prioritization, business rules, exception rules and statuses, logical data model, integration flows and field mapping, user stories with acceptance criteria, UAT design with negative tests, requirements traceability

## Assumptions and limitations

- The company, team, volumes and dataset were created for this case study. The process steps come from my own operational experience and have not been validated with real users. In a real project they would be checked through interviews, time sampling, supplier discussions and To-Be walkthroughs.
- Etsy and supplier API access, API limits and Developer hours are assumptions to confirm in R0.
- The dataset cannot separate internal, supplier and carrier time, and supplier assignment was randomized, so it cannot be used to compare suppliers ([data notes](data/README.md#what-the-data-cannot-show)).
- Manual work per order is a working hypothesis. It is measured in R0 and R2, and no targets are set before the pilot.
- The solution has not been built and UAT has not been run.

## Repository structure

```
.
├── README.md                  Project overview (this page)
├── docs/                      Documents in Markdown, readable on GitHub
│   ├── 01-project-summary.md
│   ├── 02-brd.md
│   ├── 03-frd.md
│   ├── 04-user-stories.md
│   ├── 05-uat-test-cases.md
│   └── 06-rtm.md
├── assets/
│   ├── diagrams/              As-Is and To-Be process diagrams, system context, data model
│   ├── wireframes/            Needs-action view, operations dashboard
│   └── slides/                Slide images for the Project Summary
├── data/                      Simulated dataset (SQLite and CSV) and data dictionary
└── sql/                       Schema and analysis queries with their output
```

## How to review or reproduce the analysis

1. **5 minutes:** this page, the [Project Summary](docs/01-project-summary.md) and the [RTM](docs/06-rtm.md).
2. **20 minutes:** [BRD §3–6](docs/02-brd.md#3-problem-statement-and-pain-points) for the problem, evidence and root cause, then [FRD §2](docs/03-frd.md#2-solution-evaluation-and-selection) and [§4–5](docs/03-frd.md#4-to-be-process) for the decision, To-Be process and exception rules, then a few [user stories](docs/04-user-stories.md) and their [UAT cases](docs/05-uat-test-cases.md).
3. **Re-run the SQL:** with the SQLite command line, from the repository root:

   ```bash
   sqlite3 data/augbridge_simulated.db < sql/analysis_queries.sql
   ```

   You can also open `data/augbridge_simulated.db` in a tool such as DB Browser for SQLite and paste one query at a time. The expected output is written under each query. The CSV files open directly on GitHub.

## Skills demonstrated

- Turning operational experience into clear pain points and testable requirements
- Using data carefully: showing what the evidence supports and what it does not
- Process mapping and root cause analysis
- Comparing options with explicit criteria and trade-offs
- Writing user stories, UAT test cases and an RTM that link end to end
- Basic SQL to support the business evidence
