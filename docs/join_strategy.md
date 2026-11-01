# Join Strategy

- **Anchor table:** CRM Deals (keeps the 85 revenue accounts ACCT-001–085)
- **Join 1:** CRM → Marketing Campaigns, **Left Join** on `account_id`
- **Join 2:** → Support Tickets, **Left Join** on `account_id.1` (marketing key; safe because all 85 CRM accounts exist in marketing)
- **Result:** 65,448 rows × 67 columns (fan-out from one-to-many relationships)
- **Excluded orphans:** ACCT-101–115 → 150 marketing rows (6.7%) and 204 support rows (6.8%) with no CRM deal
- **Fan-out control:** Sheets 1–2 use single-source datasets for exact totals; Sheet 3 cross-domain visuals use the unified dataset with **distinct count**
