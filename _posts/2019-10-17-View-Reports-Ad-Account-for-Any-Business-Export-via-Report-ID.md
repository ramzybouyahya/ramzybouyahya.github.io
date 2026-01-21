---
title: View Reports Ad Account for Any Business (Export via Report ID)
description: Using report ID access, an attacker could export Ads Manager reports for arbitrary businesses.
date: 2025-10-12 11:33:00 +0800
categories: [Meta bug bounty]
tags: [Meta]
pin: false
math: true
mermaid: true
---

### 🧾 Description
Using the Graph API or report download endpoints, an attacker could export Ads Manager reports for any business by enumerating or obtaining a 16‑digit `report_run_id`. Because report IDs are predictable numeric values, an attacker can brute‑force or enumerate IDs and retrieve report data (account name, reach, amount spent, etc.) if they have a valid access token. This exposes potentially sensitive advertising analytics across businesses.

**Impact:** Unauthorized access to advertising performance data, potential leakage of strategic metrics and spend information across organizations.

---

## ⚙️ Steps to Reproduce

1. Visit Ads Manager reporting for a business:  
   `https://business.facebook.com/adsmanager/reporting/business_view?business_id={YOUR_ID_Business}&event_source=BIZ_HOME_TABLE_ITEM`
2. Create a new report and **Save** it with a name.  
3. Open Inspector (DevTools) and observe the network requests/responses; when clicking Export report, the response contains `report_run_id` (a 16‑digit numeric ID).  
4. As a second account (not admin/employee of the victim business), load the direct download endpoint:
   ```http
   GET /ads/report_builder/export/download_report/?report_run_id={ID_report}&scope=business_account HTTP/1.1
   Host: www.facebook.com
   User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:62.0) Gecko/20100101 Firefox/62.0
   Accept: */*
   Content-Type: application/x-www-form-urlencoded
   Connection: close
   ```
   The file will be returned in the attacker's session containing the exported report with account names, reach, amount spent, etc.
5. Alternatively, use Graph API to fetch the report by ID:
   ```http
   GET /{ID_report}&access_token={TOKEN_Attacker} HTTP/1.1
   Host: graph.facebook.com
   User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:62.0) Gecko/20100101 Firefox/62.0
   Accept: */*
   Content-Type: application/x-www-form-urlencoded
   Connection: close
   ```
   - The attacker can obtain a business access token (or other tokens) by inspecting their own Business Manager requests or other means.  
   - With a valid token, the attacker can retrieve the report content programmatically.
6. Because `report_run_id` values are numeric and predictable, an attacker may enumerate IDs to discover valid reports for other businesses.

---

### 🧱 Expected Behavior
Report export/download endpoints and Graph API resources for saved reports should enforce strict access checks tied to the owning Business and should not allow arbitrary token holders to fetch reports belonging to other businesses.

---
## Timeline

- **Reported:** October 17, 2019
- **Triaged:** October 21, 2019
- **Reward:** October 31, 2019
- **Fixed:** Januray 2, 2020 - $7,500

---
