# Executive Report: NovaTech Solutions Revenue Intelligence Dashboard

**Prepared For:** Sarah Chen, VP of Revenue Operations, NovaTech Solutions  
**Prepared By:** Abhik Mehta, Business Intelligence Analyst  
**Date:** November 2026  

---

## 1. Executive Overview & Business Objective

Until now, NovaTech Solutions' revenue operations team has spent every Monday morning manually pulling disconnected reports from three separate operational systems: Customer Relationship Management (CRM) sales pipeline records, marketing campaign execution logs, and customer support ticket queues. Because these systems operated in silos, leadership could not easily trace how marketing investments influenced deal closures or how post-sale support issues affected customer retention.

To eliminate manual reporting and establish a single source of truth across the customer lifecycle, we designed and deployed the **NovaTech Revenue Intelligence Dashboard** and a dedicated **Amazon QuickSight Q Topic** (an artificial intelligence natural language querying layer that enables business users to ask plain-English questions of underlying datasets). Together, these tools unify marketing funnel performance, sales pipeline velocity, and customer health indicators into one interactive decision-support environment.

---

## 2. Data Strategy, Preparation & Unified Join Architecture

### 2.1 Source Verification & SPICE Ingestion

Before building visualizations, we audited all three pre-indexed organizational knowledge bases against the official NovaTech Data Dictionary and imported the source files into **SPICE** (Super-fast, Parallel, In-memory Calculation Engine, Amazon QuickSight's in-memory analytics engine designed for rapid query performance). We verified exact record counts and date ranges across all four SPICE datasets:

* **CRM Deals Dataset:** 499 closed sales deals across 22 SPICE columns (June 17, 2023 to January 25, 2025), capturing deal outcomes (315 Won and 184 Lost), realized revenue, sales cycle length, and loss reasons.
* **Marketing Campaigns Dataset:** 2,240 campaign touchpoints across 23 SPICE columns (January 1, 2023 to January 31, 2025), tracking multi-channel ad spend, funnel stages, lead conversions, and return on investment.
* **Support Tickets Dataset:** 3,000 customer support inquiries across 22 SPICE columns (June 1, 2023 to February 28, 2025), recording ticket priority, resolution time, product area friction, and open backlog.
* **Unified Account-Level Dataset:** 65,448 combined rows across 67 columns, linking CRM accounts to their associated marketing touches and support tickets.

### 2.2 Data Quality Remediation & Business Metric Engineering

During data preparation, we standardized field types and excluded non-analytical system metadata. Specifically, we converted the CRM List Price field from a whole-number integer to a decimal currency format to ensure accurate pricing ratios. We also validated key calculated business metrics:

* **Days to Close:** The elapsed calendar days between a deal's creation date and its closing date.
* **Won Deal Indicator & Price Realization Ratio:** A binary indicator distinguishing won deals from lost deals, paired with the ratio of final realized deal value relative to initial list price.
* **Resolution Time in Days:** The elapsed time required to resolve customer support tickets.

We also documented three structural dataset characteristics so stakeholders interpret totals accurately:
1. **Closed-Only Pipeline Scope:** All 499 CRM records represent completed opportunities (315 Won and 184 Lost). Lost opportunities record $0 in realized deal value, while won opportunities naturally have blank values in the Loss Reason column.
2. **Expected Missing Values:** In the marketing dataset, 24 records (1.1%) lack self-reported annual income values. In the support dataset, 59 tickets (2.0%) have no resolution date because those tickets remain actively open.
3. **Unmatched Prospect & Legacy Accounts (Orphan Records):** While all 85 active CRM customer accounts (ACCT-001 through ACCT-085) appear in the marketing and support tables, 15 additional account identifiers (ACCT-101 through ACCT-115) appear in marketing (150 rows, or 6.7%) and support (204 rows, or 6.8%) without a matching CRM deal. These represent top-of-funnel prospects not yet converted into sales opportunities or legacy support-only accounts.

### 2.3 Anchor Table Selection & Controlling Row Multiplication (Fan-Out)

To construct the unified dataset, we selected **CRM Deals** as our **anchor table** (the primary base table in a data join that determines which core business entities are retained) and performed sequential **Left Joins** (a join operation that retains all records from the primary anchor table and attaches matching records from secondary tables) on the shared Account ID key to Marketing Campaigns and Support Tickets.

Because a single customer account participates in multiple deals, multiple marketing campaigns, and dozens of support tickets, joining three one-to-many tables naturally causes **row multiplication (fan-out)**—expanding 499 CRM deals into 65,448 joined rows. To prevent row multiplication from inflating financial totals or ticket counts, we applied a hybrid dataset governance strategy:
* **Single-Domain Accuracy (Sheets 1 and 2):** The Marketing Funnel and Sales Pipeline sheets query their unjoined source datasets directly, guaranteeing exact additive totals ($12,359,497.34 in marketing spend and $707,201 in closed-won revenue).
* **Cross-Domain De-Duplication (Sheet 3):** The Customer Health sheet queries the unified dataset using **distinct count aggregations** (counting each unique Ticket ID and Account ID only once regardless of how many joined rows it appears in).

---

## 3. Dashboard Design Rationale & Interactive Architecture

The published dashboard is organized into three sheets aligned with the NovaTech customer journey:

* **Sheet 1 — Marketing Funnel:** Built for demand-generation analysis. Top-level **Key Performance Indicator (KPI)** summary cards highlight total marketing spend ($12,359,497.34), total campaign touches (2,240), and total responses (609, a 27.2% conversion rate). A funnel chart tracks stage progression (848 Qualified Leads, 465 Leads, 412 Closed Won, 404 Prospects, and 111 Opportunities), a combo chart compares campaign spend against responses, and a bar chart ranks channel volume. An interactive Marketing Channel dropdown filter dynamically updates all visuals on the sheet.
* **Sheet 2 — Sales Pipeline:** Built for deal velocity and win/loss inspection. KPI cards display closed-won revenue ($707,201), total closed deals (499 at a 63.1% win rate), and average sales cycle duration (66.76 days). Charts break down deal outcomes, won revenue by customer segment (Enterprise, Large, Medium, Small), and lost deals by objection category. Alongside a pinned Company Size filter control, we configured **one-click interactive filtering** across visuals: clicking the Enterprise segment bar immediately filters the sheet to show $319,423 in Enterprise won revenue across 201 deals with a 64.45-day average closing cycle.
* **Sheet 3 — Customer Health:** Built for post-sale retention and escalation management. KPI cards track total support volume (3,000 tickets across 2,996 unique Ticket IDs), average resolution speed (1.95 days), and resolved versus open backlog (2,941 resolved; 59 open). Visuals display ticket counts by priority (1,500 low, 1,050 medium, 400 high, 50 critical), resolution time by product area, and a cross-dataset chart comparing distinct support tickets and distinct accounts by CRM segment. A **cross-sheet navigation action** allows executives to click any customer segment and jump directly to the Sales Pipeline sheet to inspect revenue exposure.

---

## 4. Impact of Topic Configuration on AI Natural Language Querying (Quick Chat)

We tested Quick Chat both before and after configuring the **NovaTech Revenue Intelligence Topic**:

* **Baseline Performance (Before Topic Configuration):** When scoped only to the published dashboard without a configured Topic, Quick Chat accurately answered questions whose exact numbers appeared on KPI cards (such as confirming $707,201 in closed-won revenue and a 63.1% win rate). However, when asked about unplotted dataset columns or business synonyms—such as total ad budget and average Campaign ROI by marketing channel—Quick Chat returned overall spend ($12,359,497.34) and channel response counts (Partner Referral 277, Paid Social 131, Organic Search 97, Direct Mail 79, Email 25), but stated that Campaign ROI by channel was unavailable because it was not displayed on a dashboard visual.
* **Post-Topic Improvements (After Semantic Enrichment):** We configured the Topic by enabling underlying analytical fields, excluding noisy system identifiers, and mapping business synonyms (such as "ad budget" for Campaign Spend, "ROI" for Campaign ROI, "sales cycle" for Days to Close, and "MTTR" for Resolution Time Days). After configuration, Q answered the same ROI question it had previously declined. It returned total spend of $12,359,497.34 and an average Campaign ROI of −0.90 across five channels, with Direct Mail performing best (−0.64) and Organic Search worst (−0.96). It also correctly read 'sales cycle' as average Days to Close (66.76 days, matching the dashboard) and 'MTTR' as average Resolution Time Days (1.95 days across 2,941 resolved tickets).

---

## 5. Key Quantified Business Insights & Recommended Actions

1. **Marketing Channel Efficiency & Budget Reallocation:**
   * **Quantified Finding:** NovaTech spent **$12,359,497.34** across **2,240** campaign touches, generating **609** responses (**27.2%** conversion rate) at a blended cost of **$20,294.74 per response**. While **Partner Referral** generated the highest volume (**277 responses**, or 45.5%), it consumed **$7,270,706.10 (58.8% of total spend)** at **$26,248 per response** (−0.89 ROI). Conversely, **Direct Mail** spent **$721,309.14** to generate **79 responses** at just **$9,130 per response** and achieved the strongest channel ROI (**−0.64**).
   * **Recommended Action:** Partner Referral produces the most responses but absorbs 58.8% of spend at roughly $26,248 per response, while Direct Mail costs about $9,130 per response. Cap Partner Referral spend and shift 15–20% of it into Direct Mail and Organic Search, targeting a blended cost-per-response below $15,000.

2. **Sales Pipeline Velocity & Mid-Cycle Loss Prevention:**
   * **Quantified Finding:** Across **499** closed deals, NovaTech achieved a **63.1% win rate** (**315 Won** vs. **184 Lost**) and **$707,201** in closed-won revenue with an average sales cycle of **66.76 days**. **Enterprise** ($319,423 across 201 deals, 64.45 days to close) and **Large** segments drive the majority of revenue, but **184 deals (36.9%)** were lost evenly across five preventable objections: No Decision Made, Poor Product Fit, Competitor Won, Timing Not Right, and Budget Constraints.
   * **Recommended Action:** Institute a mandatory **Day-35 Executive ROI & Solution Alignment Review** for all open Enterprise and Large deals to address No Decision and Budget Constraints stalls before day 60, targeting a **20% reduction in lost deals** next quarter.

3. **Customer Support Backlog & High-Severity Escalation Routing:**
   * **Quantified Finding:** Out of **3,000** support tickets (**2,996** unique Ticket IDs), **450 tickets (15.0%)** are high-severity (**400 high**, **50 critical**), and **59 tickets (2.0%)** remain unresolved against an overall average resolution speed of **1.95 days** (**2,941** resolved). Cross-dataset distinct counts show heavy support activity across Small (~1,120 tickets), Medium (~1,100 tickets), and Large (~790 tickets) segments.
   * **Recommended Action:** Route all **50 critical** and **59 unresolved** tickets into a dedicated Tier-3 rapid-response engineering queue and trigger automated Customer Success alerts whenever an active account logs more than **3 high-priority tickets in a quarter**.

---

## 6. AI Analysis vs. Dashboard Comparison

* **Where AI and the Dashboard Agreed:** Both the structured dashboard visuals and QuickSight's AI features aligned 100% on core metrics: $12,359,497.34 in marketing spend, 2,240 leads, 609 campaign responses (27.2% conversion), 277 Partner Referral responses, $707,201 in closed-won revenue, 63.1% win rate (315 won / 499 total), 66.76 average days to close, and 1.95-day average support resolution time across 2,941 resolved tickets.
* **Where Differences Occurred & Why:** Before Topic configuration, Quick Chat could only summarize metrics already plotted on visible dashboard charts. Once the Topic was configured, Quick Chat could immediately query unplotted fields and business synonyms across datasets, while the dashboard remained essential for visual trend inspection, one-click filtering, and governed cross-sheet navigation.
* **Where AI and the dashboard disagreed:** The dashboard's response-volume chart makes Partner Referral look like the strongest channel, but Q's ROI breakdown showed every channel returning negative ROI, with Direct Mail the most efficient. Volume and efficiency tell different stories, so budget decisions should use both views.
