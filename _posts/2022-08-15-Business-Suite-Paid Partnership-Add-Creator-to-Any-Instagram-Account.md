---
title: Business Suite (Paid Partnership) - Add Creator to Any Instagram Account
description: Authorization flaw allowed adding arbitrary creators to a brand's Paid Partnership on Instagram via GraphQL mutation.
date: 2022-08-15 
categories: [Meta bug bounty]
tags: [Meta]
pin: false
math: true
mermaid: true
---

### 🧾 Description 
An attacker could manage Paid Partnerships for any business‑linked Instagram account by adding arbitrary creators through a GraphQL mutation.  
The attacker only needed:
- The **brand Instagram ID** (`brand_ig_fbid`)
- The **creator Instagram ID** (`creator_ig_fbid`)
- Their **own** business access token (not belonging to the target Business)

**Impact:** Unauthorized brand–creator linking invites could be sent on behalf of the target Business, enabling abuse of branded content ads, fraud, or reputational harm.

---

## ⚙️ Steps to Reproduce

**Users:**  
- **UserA** — Admin of **BusinessA** (`business_id = AAA`), connected to Instagram (`brand_ig_fbid = XXX`)  
- **UserB** — Attacker (not a member of BusinessA)

**Preconditions:** BusinessA is linked to a **professional Instagram account**.

1. **Recon:** UserB learns the target **Instagram Brand ID** (`{XXX}`) and the desired **Creator IG ID** (`{ID_creator}`).  
2. **Access Token:** UserB goes to `https://business.facebook.com/settings/?business_id={ID_business}` and captures **their own** business/adnroid access token from network traffic.  
3. **Graph API Explorer:** Visit `https://developers.facebook.com/tools/explorer/` and configure a **POST** GraphQL request.  
4. **Submit Mutation:**

### 📤 Request
```http
POST /graphql HTTP/1.1
Host: graph.facebook.com
User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:62.0) Gecko/20100101 Firefox/62.0
Accept: */*
Content-Type: application/x-www-form-urlencoded
Connection: close

&doc_id=5822628081097871&variables={"input": {"client_mutation_id": "9","actor_id": "0","brand_ig_fbid": "{XXX}","business_id": "{AAA}","creator_ig_fbid": "{ID_creator}"}}
```

### ✅ Response (example)
```json
{
  "data": {
    "xfb_create_creator_ad_permissions": {
      "proxy": { "id": "ID" },
      "branded_content_ads_permission": { "id": "ID" }
    }
  },
  "extensions": { "is_final": true }
}
```

5. **Verification:** UserA opens  
   `https://business.facebook.com/latest/settings/business_paid_partnerships?business_id={AAA}`  
   → Sees an **invitation** has been created to the specified **creator** by UserB.

---

## 🧱 Expected Behavior
Only **authorized admins** of the target **BusinessA** should be able to add **creators** to its **Paid Partnership** settings for the linked Instagram brand.

---

## Timeline

- **Reported:** August 15, 2022
- **Triaged:** August 17, 2022
- **Fixed:** August 20, 2022 
- **Reward:** August 22, 2022

---
