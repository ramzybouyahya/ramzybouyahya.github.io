---
title: One-Click CSRF for Unauthorized Session Collaboration Approval on manus
description: One-Click CSRF for Unauthorized Session Collaboration Approval on manus
date: 2026-02-04
categories: [Meta bug bounty]
tags: [Manus.im]
pin: true
math: true
mermaid: true
---

### 🧾 Description
During a security assessment of manus.im, I discovered that the collaboration approval workflow was vulnerable to Cross-Site Request Forgery (CSRF). An attacker could force a session owner to grant them READ_WRITE access to private AI sessions simply by convincing the victim to click a specially crafted link. This bypasses the need for an explicit user confirmation, leading to potential sensitive data leakage.
The application handles session collaboration requests via a predictable GET endpoint. When an attacker initiates a collaboration request, the backend generates a requestId. The subsequent approval mechanism relies solely on this requestId and sessionId passed as URL parameters.
Crucially, the application failed to validate an Anti-CSRF token or perform any secondary authorization check (such as a POST request with a CSRF header) before executing the state-changing operation.

**Impact:** Attackers gain full READ_WRITE permissions to private sessions.

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

---

## Timeline

- **Reported:** May 22, 2018 
- **Triaged:** May 23, 2018 
- **Fixed:** June 16, 2018  
- **Reward:** June 20, 2018 - $1,500

---
