---
title: Bypass Pixel Role (Partner Business)
description: Partner businesses with analyst role could escalate to pixel editor via the sharing_agreement endpoint.
author: ramzybouayhya
date: 2019-1-14 11:33:00 +0800
categories: [Meta bug bounty]
tags: [Meta]
pin: false
math: true
mermaid: true
---

### 🧾 Description
A partner business granted the analyst role on a Pixel could abuse the `business/objects/sharing_agreement/` endpoint to create sharing agreements and escalate permissions — effectively becoming a pixel editor. This allows unauthorized modification of Pixel settings and potential misuse of tracking/ads data on behalf of the victim business.

**Impact:** Partner businesses can escalate privileges from low‑privileged roles (analyst) to higher privileges (editor), enabling manipulation of advertising pixels and data collection configurations across businesses.

---

## ⚙️ Steps to Reproduce

**Actors**  
- **Victim:** Admin of `Business_A` (owns `Pixel_A`)  
- **Attacker:** Admin of `Business_B` & `Business_C` (partner businesses)

**Steps**

1. **Business_A** owner adds **Business_B** as a partner on `Pixel_A` with role **analyst**.  
2. **Attacker** uses the endpoint `business/objects/sharing_agreement/` to create a sharing agreement referencing a third party business (**Business_C**), supplying permitted roles that include editor privileges.

### 📤 Request
```http
POST /business/objects/sharing_agreement/ HTTP/1.1
Host: business.facebook.com
Content-Type: application/x-www-form-urlencoded

&from_business_id={ID_Business_B}&asset_id={ID_Pixel_A}&asset_type=pixel&to_business_id={ID_Business_C}&top_permitted_roles[0]=187605565181235&top_permitted_roles[1]=213091122906638&relationship_type[0]=Agency&relationship_type[1]=Ad%20Manager&other_relationship=
```

### ✅ Response
```
for (;;);{"__ar":1,"payload":{"success":true,"sharing_status":"In Progress"},"bootloadable":{},"ixData":{},"bxData":{},"gkxData":{},"qexData":{},"lid":"6646428536265726141"}
```

4. The attacker then switches to **Business_C** and completes the flow to add themselves as a **pixel editor**, effectively bypassing the intended analyst-only limitation.

**Result:** A partner business with only analyst access is able to orchestrate role escalation and gain editor privileges on the pixel asset.

---

### 🧱 Expected Behavior
Sharing agreement creation must strictly validate that the initiating business and the requested roles are within the permissions granted by the target business. An analyst role must not be able to elevate privileges to editor without explicit approval by the pixel owner.

---

## Timeline

- **Reported:** Januray 14, 2019 
- **Triaged:** Januray 15, 2019 
- **Fixed:** Januray 18, 2019  
- **Reward:** Februray 12, 2019 - $3,000

---
