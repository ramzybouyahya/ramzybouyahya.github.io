---
title: Join Workplace Without Approval of Workplace Admin
description: Users from allowed/verified domains could join a Workplace without admin approval using invite link or activation flow.
author: ramzybouyahya
date: 2021-08-9 11:33:00 +0800
categories: [Meta bug bounty]
tags: [Meta]
pin: false
math: true
mermaid: true
---

### 🧾 Description
Workplace admins can configure joining rules to require admin approval for all join requests (e.g., *Anyone from allowed or verified domains* + *Admins must approve all requests to join this Workplace*).  
However, an attacker who obtains an invite link and controls an email in the allowed/verified domain can complete the activation flow and become a member **without admin approval**. This bypass breaks the intended approval workflow and allows unauthorized users to join the workplace.

**Impact:** Unauthorized access to internal workplace resources, potential data access, and membership-based privileges. This undermines administrative control over membership and increases risk of insider threats or information leakage.

---

## ⚙️ Steps to Reproduce

**Actors & environment**
- **User_A** — Admin of `WorkplaceA` (domain: `workdomain.tn`)
- **User_B** — Legitimate invite target (`user_B@workdomain.tn`)
- **User_C** — Attacker (`user_C@workdomain.tn`) — has an email address in the same allowed/verified domain
- Browser: Chrome / Firefox; any OS

**Preconditions**
- In **WorkplaceA** settings:  
  - **Joining this Workplace** = **Anyone from allowed or verified domains**  
  - **Access requests** = **Admins must approve all requests to join this Workplace**

**Steps**
1. **User_A** (admin) generates an invite link via the admin settings. The invite link format resembles:  
   `https://workdomain.workplace.com/work/landing/input/?signup_id={ID}&signup_nonce={nonce}`
2. **User_B** uses the invite link and fills registration details. **UserA** receives a join request notification and **approves** UserB — normal flow.
3. **User_C** (attacker) visits a generic invite/create page: `https://work.workplace.com/company_creation/invite` and enters their email address (same verified domain).
4. Workplace sends **User_C** an activation email with a code/link. **User_C** uses this activation link/code to complete the account activation flow.
5. **Observed:** Instead of being placed into an **"awaiting approval"** state, **User_C** is activated and added to the Workplace **without requiring an admin to approve** the join request.

**Result:** The admin approval control is bypassed; any email address at the verified domain can be used to join immediately when the activation flow is followed.

---

### 🧱 Expected Behavior
When **Access requests** are set to **Admins must approve all requests**, any new account created for the allowed/verified domain should be placed into a pending state until an admin explicitly approves the request. Activation links or invite flows must not elevate a user to full membership without admin verification.

---

## Timeline

- **Reported:** August 9, 2021
- **Triaged:** September 1, 2021
- **Reward:** November 10, 2021 - $2,500
- **Fixed:** November 25, 2021

---
