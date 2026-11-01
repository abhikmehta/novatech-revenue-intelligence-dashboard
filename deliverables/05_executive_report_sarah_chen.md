# Executive Report: NovaTech Solutions Revenue Intelligence Dashboard

**Prepared For:** Sarah Chen, VP of Revenue Operations, NovaTech Solutions  
**Prepared By:** Abhik Mehta, Business Intelligence Analyst  
**Date:** November 2026  

---

## 1. Executive Overview & Business Objective

Until now, NovaTech Solutions' revenue operations team has spent every Monday morning manually pulling disconnected reports from three separate operational systems: Customer Relationship Management (CRM) sales pipeline records, marketing campaign execution logs, and customer support ticket queues. Because these systems operated in silos, leadership could not easily trace how marketing investments influenced deal closures or how post-sale support issues impacted customer retention and expansion.

To eliminate manual reporting and provide a single source of truth across the entire customer lifecycle, we designed and deployed the **NovaTech Revenue Intelligence Dashboard** and an accompanying **Amazon QuickSight Q Topic** (an artificial intelligence natural language querying layer that allows business users to ask plain-English questions of underlying datasets). Together, these tools unify marketing funnel performance, sales pipeline velocity, and customer health indicators into one interactive decision-support environment.

---

## 2. Data Strategy, Preparation & Unified Join Architecture

### 2.1 Source Verification & SPICE Ingestion

Before building visualizations, we audited all three pre-indexed organizational knowledge bases against the official NovaTech Data Dictionary and imported the source datasets into **SPICE** (Super-fast, Parallel, In-memory Calculation Engine, Amazon QuickSight's in-memory analytics engine designed for rapid query performance):

| Dataset Name | Record Count | Column Count (Raw / SPICE) | Date Coverage | Primary Analytical Role |
|:---|:---|:---|:---|:---|
| **CRM Deals (`novatech_crm_deals.csv`)** | 499 deals | 20 / 22 columns | June 17, 2023 – January 25, 2025 | Closed sales pipeline outcomes, deal values, cycle length, and loss reasons |
| **Marketing Campaigns (`novatech_marketing_campaigns.csv`)** | 2,240 touches | 20 / 23 columns | January 1, 2023 – January 31, 2025 | Multi-channel campaign spend, funnel stage progression, and lead responses |
| **Support Tickets (`novatech_support_tickets.csv`)** | 3,000 tickets | 20 / 22 columns | June 1, 2023 – February 28, 2025 | Ticket severity distribution, resolution speed, product area friction, and open backlog |
| **Unified Account-Level Dataset** | 65,448 rows | 67 columns (93.2 MB) | January 1, 2023 – February 28, 2025 | Cross-domain analysis linking CRM accounts to marketing touches and support tickets |

### 2.2 Data Quality Remediation & Business Metric Engineering

During data preparation, we standardized field types and removed non-analytical metadata (`data_source`). Specifically, we corrected `list_price` in the CRM dataset from a whole-number integer to a decimal currency format to ensure accurate pricing ratios. We also validated pre-existing and newly engineered business metrics, including:

* **Days to Close:** The elapsed calendar days between a deal's creation date and its closed date.
* **Win Indicator & Price Realization Ratio:** A binary indicator distinguishing closed-won deals from closed-lost deals, paired with the ratio of final realized deal value relative to initial list price.
* **Resolution Time (Days):** The elapsed time required to resolve customer support inquiries.

Several important data characteristics were documented to prevent misinterpretation:

1. **Closed-Only Pipeline Scope:** All 499 CRM records represent completed opportunities (`315 Won` and `184 Lost`). Lost opportunities record a realized revenue value of `$0`, while won opportunities appropriately contain blank (`null`) values in the `loss_reason` column.
2. **Expected Missing Values:** In the marketing dataset, 24 records (`1.1%`) lack self-reported annual income values. In the support dataset, 59 tickets (`2.0%`) have no resolution date because those tickets remain actively open.
3. **Unmatched Prospect & Legacy Accounts (Orphan Records):** While all 85 active CRM customer accounts (`ACCT-001` through `ACCT-085`) appear across the marketing and support tables, 15 additional account identifiers (`ACCT-101` through `ACCT-115`) appear in marketing (`150 rows`, or `6.7%`) and support (`204 rows`, or `6.8%`) without a matching CRM deal record. These represent top-of-funnel prospects not yet converted into formal sales opportunities or legacy support-only accounts.

### 2.3 Anchor Table Selection & Handling Row Multiplication (Fan-Out)

To build the fourth, unified dataset, we selected **CRM Deals (`novatech_crm_deals.csv`)** as our **anchor table** (the primary base table in a data join that defines which core business entities are retained) and executed sequential **Left Joins** (a join operation that keeps all records from the primary anchor table and attaches matching records from secondary tables) on the shared `account_id` key to Marketing Campaigns and Support Tickets.

Anchoring on CRM Deals ensures that cross-functional account health analysis remains focused on the 85 revenue-generating pipeline accounts (`ACCT-001` to `ACCT-085`). Because a single customer account has multiple deals, multiple marketing campaign touches, and dozens of support tickets, joining these three one-to-many tables naturally produces **row multiplication (fan-out)**—expanding the 499 CRM deals into **65,448 combined rows**. To prevent this row multiplication from artificially inflating financial totals or ticket counts, we enforced a strict **hybrid dataset governance strategy**:

* **Single-Domain Accuracy (Sheets 1 and 2):** The *Marketing Funnel* and *Sales Pipeline* sheets query their respective unjoined source datasets directly, guaranteeing 100% exact additive totals (`$12,359,497.34` marketing spend and `$707,201` closed-won revenue).
* **Cross-Domain De-Duplication (Sheet 3):** The *Customer Health* sheet uses the unified dataset for cross-domain visuals using **distinct count aggregations** (counting each unique `ticket_id` and `account_id` only once regardless of how many joined rows it appears in).

---

## 3. Dashboard Design Rationale & Interactive Architecture

The dashboard is organized into three progressive sheets mirroring the NovaTech customer journey:

1. **Sheet 1 — Marketing Funnel (`Marketing_Funnel`):** Designed for demand-generation review. Top-level **Key Performance Indicator (KPI)** summary cards highlight total marketing spend (`$12,359,497.34`), total campaign leads (`2,240`), and total responses (`609`, a `27.2%` conversion rate). A stage funnel visual tracks lead progression (`848 Qualified Leads`, `465 Leads`, `412 Closed Won`, `404 Prospects`, `111 Opportunities`), a combo chart compares spend against responses across all six campaigns, and a horizontal bar chart ranks channel effectiveness. A pinned `campaign_channel` dropdown filter enables dynamic slice-and-dice analysis across all visuals.
2. **Sheet 2 — Sales Pipeline (`Sales_Pipeline`):** Designed for pipeline velocity and win/loss inspection. KPI cards display total closed-won revenue (`$707,201`), total pipeline deals (`499` at a `63.1%` win rate), and average sales cycle duration (`66.76 days`). Visuals break down outcomes (`315 Won` vs. `184 Lost`), closed-won revenue by customer segment (`Enterprise`, `Large`, `Medium`, `Small`), and lost deals by root cause. In addition to a pinned `Company Size` filter control, we configured **one-click interactive filtering actions** on the visuals: clicking the `Enterprise` segment bar immediately filters the entire sheet to reveal `$319,423` in Enterprise won revenue across `201` deals with an accelerated `64.45-day` average closing cycle.
3. **Sheet 3 — Customer Health (`Customer_Health`):** Designed for post-sale retention and escalation management. KPI cards track total support volume (`3,000` tickets), average resolution speed (`1.95 days`), and resolved vs. open backlog (`2,941` resolved; `59` open). Visuals display ticket distribution by priority (`1,500 low`, `1,050 medium`, `400 high`, `50 critical`), average resolution time by product area, and a **cross-dataset visual** built from the unified dataset comparing distinct support tickets and distinct accounts across CRM customer segments. Finally, a **cross-sheet navigation action** (`Navigate to Sales Pipeline`) enables executives to click any customer segment on the Customer Health sheet and jump directly to the Sales Pipeline sheet to inspect revenue exposure.

---

## 4. Impact of Topic Configuration on AI Natural Language Querying (Quick Chat)

We evaluated Amazon QuickSight's AI-powered Quick Chat both **before** and **after** configuring the dedicated **NovaTech Revenue Intelligence Topic**:

* **Baseline Performance (Before Topic Configuration):** When Quick Chat was scoped only to the published dashboard without a configured Topic, it accurately answered questions whose exact numbers were printed on visible KPI cards (such as confirming `$707,201` in closed-won revenue and a `63.1%` win rate). However, when asked business-phrased questions involving unplotted dataset columns or domain synonyms—such as *"What is our total ad budget and average Campaign ROI across each marketing channel?"*—Quick Chat returned the overall spend (`$12,359,497.34`) and channel response counts (`Partner Referral: 277`, `Paid Social: 131`, `Organic Search: 97`, `Direct Mail: 79`, `Email: 25`), but explicitly stated that it could not retrieve per-channel `Campaign ROI` because that field was not surfaced in the dashboard visuals.
* **Post-Topic Improvements (After Semantic Enrichment):** By creating the **NovaTech Revenue Intelligence Topic**, excluding noisy technical identifiers (`data_source`, `lead_id`, `ticket_id`), and enriching fields with business synonyms (`ad budget` for `campaign_spend`, `ROI` for `Campaign ROI`, `closed-won revenue` for `deal_value`, `MTTR` for `Resolution Time (days)`, and `severity` for `priority`), Quick Chat gained direct semantic access to all underlying columns across datasets. After configuration, Q answered the same ROI question it had previously declined. It returned total spend of $12,359,497.34 and an average Campaign ROI of −0.90 across five channels, with Direct Mail performing best (−0.64) and Organic Search worst (−0.96). It also correctly read 'sales cycle' as average Days to Close (66.76 days, matching the dashboard).

---

## 5. Key Quantified Business Insights & Recommended Actions

1. **Marketing Channel Efficiency & Budget Reallocation:**
   * **Quantified Finding:** NovaTech invested **$12,359,497.34** across **2,240** campaign touches (`Digital Retarget` at `$2,423,781.95` and `NovaPulse Launch` at `$2,423,202.47` were the largest campaigns), generating **609** total responses (**27.2%** response rate) at an average cost of **$20,294.74 per response**. **Partner Referral** generated **277 responses (45.5% of all responses)** and **Paid Social** generated **131 responses (21.5%)**, whereas **Organic Search (97)**, **Direct Mail (79)**, and **Email (25)** trailed significantly.
   * **Recommended Action:** Partner Referral produces the most responses but absorbs 58.8% of spend at roughly $26,248 per response, while Direct Mail costs about $9,130 per response. Cap Partner Referral spend and shift 15–20% of it into Direct Mail and Organic Search, targeting a blended cost-per-response below $15,000.

2. **Sales Pipeline Velocity & Mid-Cycle Loss Prevention:**
   * **Quantified Finding:** Across **499** closed deals, NovaTech achieved a **63.1% win rate** (**315 Won** vs. **184 Lost**) and **$707,201** in closed-won revenue with an average sales cycle of **66.76 days**. **Enterprise (`$319,423` across 201 deals, 64.45 days to close)** and **Large** segments drive the vast majority of revenue. However, **184 deals (36.9%)** were lost evenly across five preventable objections: *No Decision Made*, *Poor Product Fit*, *Competitor Won*, *Timing Not Right*, and *Budget Constraints*.
   * **Recommended Action:** Institute a mandatory **Day-35 Executive ROI & Solution Alignment Review** for all open Enterprise and Large opportunities to eliminate *No Decision* and *Budget Constraints* stalls, targeting a **20% reduction in lost deals** next quarter.

3. **Customer Support Backlog & High-Severity Escalation Routing:**
   * **Quantified Finding:** Out of **3,000** support tickets, **450 tickets (15.0%)** are high-severity (**400 high**, **50 critical**), and **59 tickets (2.0%)** remain unresolved against an overall average resolution speed of **1.95 days** (**2,941** resolved). Cross-dataset analysis reveals substantial support ticket volume across active Small, Medium, and Large/Enterprise accounts.
   * **Recommended Action:** Route all **50 critical** and **59 unresolved** tickets into a dedicated Tier-3 rapid-response engineering queue and configure automated Customer Success alerts whenever any active account logs more than **3 high-priority tickets in a single quarter**.

---

## 6. AI Analysis vs. Dashboard Comparison

Comparing QuickSight's AI-powered capabilities (Executive Summary and Quick Chat) against our structured dashboard sheets revealed strong complementarity:

* **Where AI and Dashboard Agreed:** Both the published dashboard visuals and the AI-generated Executive Summary / Quick Chat aligned 100% on core additive metrics: `$12,359,497.34` in marketing spend, `2,240` leads, `609` campaign responses (`27.2%` conversion), `277` Partner Referral responses, `$707,201` in closed-won revenue, `63.1%` win rate (`315` won / `499` total), `66.76` average days to close, and `3,000` support tickets with a `1.95-day` average resolution time.
* **Where Differences Occurred & Why:** Before Topic configuration, Quick Chat could only summarize data already aggregated inside visible dashboard charts—meaning it could not answer ad-hoc questions about unplotted columns like `Campaign ROI` or `downtime_minutes`. Conversely, once the Topic was linked to the datasets, Quick Chat could rapidly answer ad-hoc multi-variable questions that would otherwise require building a new chart, while the structured dashboard remained essential for visual pattern recognition, one-click segment filtering, and governed cross-dataset distinct counting.
* **Where AI and the dashboard disagreed:** The dashboard's response-volume chart makes Partner Referral look like the strongest channel, but Q's ROI breakdown showed every channel returning negative ROI, with Direct Mail the most efficient. Volume and efficiency tell different stories, so budget decisions should use both views.
