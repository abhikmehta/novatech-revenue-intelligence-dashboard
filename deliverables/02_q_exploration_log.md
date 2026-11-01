# Q Exploration Log — NovaTech Revenue Intelligence Topic

All questions were asked in Quick Chat scoped to the configured **NovaTech Revenue Intelligence Topic**, then checked against the published dashboard.

| # | Domain | Question asked | Q response | Dashboard verification | Result |
|---|---|---|---|---|---|
| 1 | CRM Deals | What is the average sales cycle by Deal Size Category? | Interpreted "sales cycle" as Average of Days To Close: **66.76 days** (NT_24) | Sales Pipeline KPI "Avg Days to Close" = 66.76 | ✅ Match |
| 2 | Marketing | What is the total ad budget and average ROI by marketing channel? | Total spend **$12,359,497.34**; average ROI **−0.90**. Direct Mail $721,309.14 (−0.64), Email $490,321.32 (−0.95), Organic Search $1,540,252.25 (−0.96), Paid Social $2,336,908.53 (−0.90), Partner Referral $7,270,706.10 (−0.89) (NT_25) | Total spend matches Marketing Funnel KPI ($12,359,497.34). ROI is not plotted on the dashboard, so the Topic added new insight | ✅ Match + new insight |
| 3 | Support | What is the average MTTR and total Is Resolved? | ⟨paste Q answer — expected 1.95 days / 2,941⟩ (NT_26) | Customer Health KPIs: Avg Resolution 1.95 days; Resolved 2,941 | ⟨✅ / ❌⟩ |
| 4 | Support | How many support tickets are there by priority? | ⟨paste Q answer⟩ | Customer Health "Support Tickets by Priority": low 1,500 / medium 1,050 / high 400 / critical 50 | ⟨✅ / ❌⟩ |
| 5 | **Cross-dataset** (Unified) | What is the distinct count of tickets by Company Size? | ⟨paste Q answer⟩ | Customer Health "Cross-Dataset: Distinct Support Tickets by CRM Segment" | ⟨✅ / ❌⟩ |

## Key observations
- **Before the Topic**, Q could only restate numbers already shown on dashboard visuals, and it declined to answer Campaign ROI by channel (NT_19).
- **After the Topic**, business synonyms ("sales cycle", "ad budget", "ROI", "MTTR") were mapped to the correct fields, and fields that aren't plotted became queryable.
- **Disagreement worth flagging:** the dashboard ranks Partner Referral first on response *volume*, but Q's ROI breakdown shows Direct Mail has the best ROI (−0.64) and the lowest cost per response (~$9,130 vs ~$26,248 for Partner Referral).
