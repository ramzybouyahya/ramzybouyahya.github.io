---
title: Exposes Private Facebook/Workplace Videos for any user
description: An internal review endpoint allowed access to private videos by ID, exposing CDN URLs for videos marked private
date: 2025-10-7
categories: [Meta bug bounty]
tags: [Meta]
pin: false
math: true
mermaid: true
---

### 🧾 Description
An internal review endpoint (`https://[REDACTED].facebook.com`) allowed fetching private video CDN URLs by supplying arbitrary `video_id` values. This permitted reviewers with access to view private videos (e.g., *Friends*, *Only me*) of any Facebook/Workplace user by calling the endpoint with the target `video_id`.

**Impact:** Unauthorized disclosure of private video content. Sensitive internal tooling exposing private user media constitutes a high‑severity privacy violation. This report was submitted under the **Whitehat Private Bounty**

---

### ⚙️ Steps to Reproduce
1. Navigate to the internal review tool: `https://[REDACTED].facebook.com/` 
2. Open Developer Tools → Console / Network.  
3. Execute the following async request in the console (replace `ID_VID` with the target private video ID):

```js
new AsyncRequest("/[REDACTED]/[REDACTED]/[REDACTED]_video/video_data_async/").setData({"video_id":"ID_VID"}).send()
```

The response returns the CDN URL for the private video (even for videos marked *Friends* or *Only me*), allowing playback/download by the reviewer.

**Result:** Any user who can reach this endpoint and knows (or guesses) a `video_id` can retrieve private video CDN links.

---

## Timeline

- **Reported:** March 11, 2023
- **Triaged:** March 12, 2023
- **Fixed:** March 15, 2023
- **Reward:** June 7, 2023 - $15,000

---
