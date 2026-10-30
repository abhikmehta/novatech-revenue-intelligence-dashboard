# Deliverable 1 — Knowledge Base Verification Log

**Analyst:** Abhik Mehta
**Method:** Asked Quick Chat (scope: All data) questions against each pre-indexed knowledge base, then compared the answers with the NovaTech Data Dictionary. Entries 1–6 were answered from the indexed data before the dictionary was shared with Quick Chat. Entries 7–8 were asked in a new chat with no dictionary attached.

| # | Knowledge Base | Question asked in Quick Chat | Q response | Data Dictionary expected | Match? / Notes |
|:---|:---|:---|:---|:---|:---|
| 1 | CRM Deals | "How many records are in the CRM deals data?" | 499 records | 499 rows, 20 columns | ✅ Match. CRM KB is complete. |
| 2 | CRM Deals | "What are the distinct deal stages and how many deals are in each?" | Won 315 (63.1%), Lost 184 (36.9%) | `deal_stage` = Won / Lost only | ✅ Match. Closed deals only, so the Sales Pipeline sheet uses win rate (63.1%) and won/lost analysis. |
| 3 | Marketing Campaigns | "How many campaigns are there and what channels are used?" | 6 campaigns, 5 channels (Direct Mail, Email, Organic Search, Paid Social, Partner Referral) | Same 6 `campaign_name` and 5 `campaign_channel` values | ✅ Match. Every campaign uses all 5 channels. |
| 4 | Marketing Campaigns | "What is the total spend and total conversions?" | $12,359,497.34 spend; 609 conversions out of 2,240 leads (27.2%) | `campaign_response` 0/1 flag; spend $501.71–$14,995.14 per row | ✅ Consistent. Spend per response is about $20,295, a business finding to test with Campaign ROI. |
| 5 | Support Tickets | "How many support tickets are there by priority?" | 3,000: Low 1,500 / Medium 1,050 / High 400 / Critical 50 | 3,000 rows; low/medium/high/critical = 1500/1050/400/50 | ✅ Match. Values are stored lowercase, so use lowercase in filters and calculated fields. |
| 6 | Support Tickets | "What is the average resolution time, and are any values missing?" | ~1.95 days; 2,941 resolved, 59 missing (2.0%) | `ticket_resolved_date` blank for 59 unresolved tickets | ✅ Match. Exclude nulls from averages and report them as "Unresolved". |
| 7 | CRM Deals | "How many unique account IDs are in the CRM deals data?" | 85 (ACCT-001 to ACCT-085) | 85 unique `account_id` values | ✅ Match. This confirms the join key range for the unified dataset. |
| 8 | Marketing + Support | "How many marketing and support rows have account IDs from ACCT-101 to ACCT-115?" | Marketing 150 (6.7%), Support 204 (6.8%) | 150 and 204 orphan rows | ✅ Match. These rows have no CRM match and drop out of a CRM-anchored left join, which is expected. |
## Evidence Screenshots
- [NT_00_01_prompt_q1_crm_record_count.png](../screenshots/00_kb_verification/NT_00_01_prompt_q1_crm_record_count.png)
- [NT_00_02_q2_crm_deal_stages.png](../screenshots/00_kb_verification/NT_00_02_q2_crm_deal_stages.png)
- [NT_00_03_q3_q4_marketing.png](../screenshots/00_kb_verification/NT_00_03_q3_q4_marketing.png)
- [NT_00_04_q5_support_priority.png](../screenshots/00_kb_verification/NT_00_04_q5_support_priority.png)
- [NT_00_05_q6_support_resolution.png](../screenshots/00_kb_verification/NT_00_05_q6_support_resolution.png)

## Summary of findings
- **All 8 checks match the Data Dictionary**, so all three knowledge bases are complete and accurate.
- **Scope:** CRM holds closed deals only (Won/Lost). Funnel progression comes from marketing `funnel_stage`.
- **Business flag:** about $20,295 spend per positive campaign response.
- **To handle in data prep:** 59 unresolved tickets, lowercase priority values, 24 null `annual_income` values (1.1%), and 354 orphan rows (150 marketing + 204 support) outside the CRM account range.
