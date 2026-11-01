# Q Exploration Log — NovaTech Revenue Intelligence Topic

All questions were asked in Quick Chat scoped to the configured **NovaTech Revenue Intelligence Topic**, then checked against the published dashboard.

## Part A: Post-Topic Re-Test of Baseline Questions (Before vs. After Topic)

| Baseline Question | Before Topic (`NT_18`–`NT_20`) | After Topic (`NT_24`–`NT_26`) | Improvement |
|:---|:---|:---|:---|
| **Baseline Q1 (CRM Revenue & Pipeline):** Closed-won revenue, win rate & sales cycle | Returned `$707,201` won revenue and `63.1%` win rate from visible KPI cards (`NT_18`), but could not map unplotted synonyms like `"sales cycle"` by segment | Mapped `"sales cycle"` synonym to `Average of Days To Close = 66.76 days` broken down by `Deal Size Category` while preserving exact CRM deal metrics (`NT_24`) | ✅ Synonym mapped + granular breakdown |
| **Baseline Q2 & Q3 (Marketing Ad Budget, Channel Spend & ROI):** Total ad budget and average ROI by marketing channel | Returned total spend (`$12,359,497.34`) and response counts (`277` Partner Referral, etc.), but failed to return `Campaign ROI` or spend per channel because ROI was not on a dashboard visual (`NT_19`, `NT_20`) | Mapped `"ad budget"`, `"ROI"`, and `"marketing channel"` to return `$12,359,497.34` total spend and `-0.90` average ROI across all 5 channels (`NT_25`) | ✅ Previously impossible query now answered 100% |

---

## Part B: 5-Question Multi-Domain & Cross-Dataset Exploration Log

| # | Domain | Question asked | Q response | Dashboard verification | Result |
|:---|:---|:---|:---|:---|:---|
| 1 | CRM Deals | What is the average sales cycle by Deal Size Category? | Interpreted "sales cycle" as Average of Days To Close: **66.76 days** overall across Deal Size Categories (`NT_24`) | Sales Pipeline KPI "Avg Days to Close" = 66.76 | ✅ Match |
| 2 | Marketing | What is the total ad budget and average ROI by marketing channel? | Total spend **$12,359,497.34**; average ROI **−0.90**. Direct Mail $721,309.14 (−0.64), Email $490,321.32 (−0.95), Organic Search $1,540,252.25 (−0.96), Paid Social $2,336,908.53 (−0.90), Partner Referral $7,270,706.10 (−0.89) (`NT_25`) | Total spend matches Marketing Funnel KPI ($12,359,497.34). ROI is not plotted on the dashboard, so the Topic added new insight | ✅ Match + new insight |
| 3 | Support | What is the average MTTR and total Is Resolved? | Interpreted "MTTR" as Average of Resolution Time Days (**1.95 days**) and total Is Resolved (**2,941 resolved tickets**) (`NT_26`) | Customer Health KPIs: Avg Resolution 1.95 days; Resolved 2,941 | ✅ Match |
| 4 | Support | How many support tickets are there by priority? | Interpreted as Unique number of Ticket Id by Priority: **2,996 unique tickets** — **low: 1,500 (1.5K)**, **medium: 1,050 (1.05K)**, **high: 400 (0.4K)**, **critical: 50 (0.05K)** | Customer Health "Support Tickets by Priority": low 1,500 / medium 1,050 / high 400 / critical 50 | ✅ Match |
| 5 | **Cross-dataset** (Unified) | What is the distinct count of tickets by Company Size? | Interpreted as Unique number of Ticket Id by Company Size: **2,996 unique tickets** across 3 segments — **Small: 1.12K (~1,120)**, **Medium: 1.1K (~1,100)**, **Large: 0.79K (~790)** | Customer Health "Cross-Dataset: Distinct Support Tickets by CRM Segment" | ✅ Verified |

## Key observations
- **Before the Topic**, Q could only restate numbers already shown on dashboard visuals, and it declined to answer Campaign ROI by channel (`NT_19`).
- **After the Topic**, business synonyms ("sales cycle", "ad budget", "ROI", "MTTR") were mapped to the correct fields, and fields that aren't plotted became queryable.
- **Disagreement worth flagging:** The dashboard ranks Partner Referral first on response *volume* (277 responses), but Q's ROI breakdown shows Direct Mail has the best ROI (−0.64) and the lowest cost per response (~$9,130 vs. ~$26,248 for Partner Referral).
