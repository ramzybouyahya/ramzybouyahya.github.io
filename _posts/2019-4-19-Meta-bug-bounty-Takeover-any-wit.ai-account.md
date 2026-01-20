---
title: Takeover any wit.ai account
description: The vulnerability permits an unauthenticated actor to takeover any wit.ai account. The only prerequisite observed is knowledge of the target wit.ai identifier; no additional credentials are required.
author: ramzybouyahya
date: 2019-04-8 11:33:00 +0800
categories: [Meta bug bounty]
tags: [Meta]
pin: false
math: true
mermaid: true
---

## 🧾Description

During routine testing of Wit.ai’s Facebook login integration I discovered a critical vulnerability: the backend accepted JWT `signed_request` tokens without verifying the HMAC-SHA256 signature. This allowed forging a valid `signed_request` by modifying the payload (e.g., `user_id`) and re-signing with any secret. The server relied only on the header & payload and ignored signature verification.

An attacker could impersonate other Facebook users and obtain `access_token` for their sessions.

---

## Vulnerability details

- **Type:** JWT signature verification bypass (HMAC-SHA256)
- **Root cause:** The server accepted tokens when header and payload looked correct but failed to validate the signature. The `algorithm` field in the JWT payload/header indicated `HMAC-SHA256`, but the implementation did not verify the HMAC signature against the server-side secret.
- **Impact:** An attacker can craft a JWT with an arbitrary `user_id` and a forged signature (signed with any key) and successfully authenticate as the targeted Facebook user in Wit.ai.

---

## ⚙️ Steps to Reproduce

1) Generate a new token and replace the user_id with the victim's ID. The secret value doesn't matter, put anything, because the signature is not being verified.

![image](https://raw.githubusercontent.com/ramzybouyahya/cdnramzy/main/12546.png)

2) Using the token that is created, send the request, the response will contain the victim's access token, which would allow you full control of the victim's account.

##### Request

```http
POST /me/facebook_signin HTTP/1.1
Host: api.wit.ai
User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:62.0) Gecko/20100101 Firefox/62.0
Accept: */*
Content-Type: application/x-www-form-urlencoded
Connection: close

&signed_request={JWT_TOKEN}
```


##### Response

```
TOKEN:"VICTIM_TOKEN"
```

Expected result (before the patch): the server returns a DATA containing an access_token for the targeted user_id.

---

## Timeline

- **Reported:** April 8, 2019
- **Triaged:** April 15, 2019
- **Fixed:** April 16, 2019
- **Reward:** April 30, 2019 - $5,000

---
