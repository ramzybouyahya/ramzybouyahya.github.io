---
title: Internal Paths/Files Leakage via Malformed Access Token on graph.meta.ai
description: The graph.meta.ai API leaks detailed internal path and file information when a malformed or invalid access token is supplied in a GET request.
date: 2025-05-20 11:33:00 +0800
categories: [Meta bug bounty]
tags: [Meta]
author: ramzybouyahya
pin: false
math: true
mermaid: true
---

## 🧾Description

The `graph.meta.ai` API leaks detailed internal path and file information when a malformed or invalid access token is supplied in a GET request. Instead of returning a standard OAuth error response, the API exposes internal stack traces and file system paths.

This behavior is triggered by providing a short or malformed token, which leads to a decryption failure within Meta’s internal cryptographic handling logic.

---

## ⚙️ Steps to Reproduce

**Request:**

```http
GET /?access_token={malformed_token} HTTP/1.1
Host: graph.meta.ai
User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:62.0) Gecko/20100101 Firefox/62.0
Accept: */*
Content-Type: application/x-www-form-urlencoded
Connection: close
```

**Response:**

![image](https://raw.githubusercontent.com/ramzybouyahya/cdnramzy/main/1_1Ki8luY57i8HAe5AyTVfjw.webp){: width="600"}

---

## Impact

Attackers could exploit this behavior to gather internal information, such as:
- Internal paths
- Files names and handler locations

---

## Timeline
- **Reported:** May 20, 2025  
- **Triaged:** May 20, 2025  
- **Fixed:** May 23, 2025  
- **Reward:** June 25, 2025

---
