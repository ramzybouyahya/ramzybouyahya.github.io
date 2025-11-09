---
title: Delete Any Ads Reporting Preview Shared with Others
description: Anyone with the preview link ID could delete/expire shared Ads Reporting previews using Graph API, impacting externally shared reports.
author: ramzybouayhya
date: 2021-06-30 11:33:00 +0800
categories: [Meta bug bounty]
tags: [Meta]
pin: false
math: true
mermaid: true
---

### 🧾 Description 
Admins of a Business can create an Ads Reporting view and share a public preview link with people outside the business.  
The generated link is associated with a unique ID. Due to missing authorization checks, any user with their own business access token and the link ID could call the **Graph API** to **DELETE** that preview, causing the link to expire immediately.

**Impact:** An attacker who learns a preview link ID can invalidate externally shared reports, disrupting collaboration, data reviews, and workflows.

---

## ⚙️ Steps to Reproduce

**Users:**  
- **UserA** — Admin of **BusinessA** (creates and shares the report preview)  
- **UserB** — Attacker (not a member of BusinessA; has their **own** business access token)

1. **UserA** visits  
   `https://business.facebook.com/adsmanager/reporting/business_view?business_id={ID_BusinessA}`
2. In the top bar, **UserA** clicks **Save** to save the Ads Reporting view.
3. **UserA** clicks **Share** → a dialog appears.
4. **UserA** toggles **Share with others** → backend generates a **preview link ID**
5. **UserA** clicks **Preview** and confirms the link is **publicly accessible**.
6. **UserB** obtains the **preview link ID** and, using **Graph API Explorer** (or any HTTP client) with **UserB’s own business or android access token**, submits:

### 📤 Request
```
DELETE v10.0/{ID_LINK}
```

### ✅ Response
```json
{
  "success": true,
  "__fb_trace_id__": "",
  "__www_request_id__": ""
}
```

7. **UserA** (or any recipient) refreshes the preview link → sees the error:  **"This report link isn't working"**

---

## 🧱 Expected Behavior
Only the **owner/admin** of the Business (or explicitly authorized users) should be able to delete/expire a shared report link.

---

## Timeline

- **Reported:** August 15, 2022
- **Triaged:** August 17, 2022
- **Reward:** August 20, 2022
- **Fixed:** September 22, 2022

---
