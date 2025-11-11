---
title: View Draft, Archived and Inactive Effects for Any Facebook or Instagram User
description: Preview parameter allowed unauthorized viewing of draft, archived, and inactive Spark AR effects for any user.
author: ramzybouayhya
date: 2019-09-10 11:33:00 +0800
categories: [Meta bug bounty]
tags: [Meta]
pin: false
math: true
mermaid: true
---

### 🧾 Description 
Facebook and Instagram users can create AR effects via the Spark AR Hub (https://www.facebook.com/sparkarhub/).
Normally, draft, archived, and inactive effects are private, only visible to their creators until published.
However, due to improper validation of the effect_id parameter in the Instagram preview endpoint, anyone could access and view these effects (including private or archived ones).

**Impact:** This vulnerability allowed unauthorized access to private, unpublished, or archived AR effects, exposing users unpublished designs, assets, and creator information.

---

## ⚙️ Steps to Reproduce

1. Visit **Spark AR Hub:**  
   `https://www.facebook.com/sparkarhub/`
2. Create two effects — one as **draft**, another **archived**, for both Facebook and Instagram targets.  
3. Access an effect preview URL:  
   `https://www.instagram.com/a/r/?effect_id={ID_effect_instagram_or_facebook}`  
   → Response: Returns **thumbnail**, **effect metadata**, and **creator user ID** even if the effect is **draft**, **archived**, or **inactive**.

**Result:** Anyone who knows or guesses an `effect_id` can view otherwise non‑public effects.

---

### 🧱 Expected Behavior
Draft, archived, and inactive effects must remain private, only accessible to the effect’s owner or authorized collaborators.

---

## Timeline

- **Reported:** September 10, 2019
- **Triaged:** September 11, 2019
- **Reward:** September 22, 2019
- **Fixed:** November 7, 2019

---