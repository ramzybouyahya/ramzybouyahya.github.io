---
title: Unlocking Private Conversations - Exploiting Systemic IDOR Vulnerabilities on monica.im
description: Insecure Direct Object Reference (IDOR) / Broken Access Control
date: 2026-07-06
categories: [Meta bug bounty]
tags: [Monica.im]
pin: false
math: true
mermaid: true
---

### 🧾 Description

During a recent security research session on monica.im, I discovered a systemic Broken Access Control/IDOR vulnerability affecting multiple API endpoints. The application failed to validate ownership when handling requests related to sharing and metadata modification. This allowed an attacker with knowledge of a target's unique identifier (`share_id` or `uid`) to bypass privacy controls, expose private conversations, manipulate chat titles, and alter the visibility of AI-generated audio transcripts.

Here is a breakdown of the three main attack vectors discovered during this investigation.

---

#### Vulnerability 1: Unauthorized Visibility Manipulation of Shared Chats

**Description**
A security vulnerability exists in the Chat Sharing feature of monica.im. When a user shares a chat conversation, the system generates a unique `shareID`. The owner of the chat has the right to switch the visibility of this conversation between "Public" and "Private".
The issue is that the application does not properly verify if the person requesting a visibility change is the actual owner of the chat. An attacker with knowledge of the `share_id` can send a request to the server to force a Private conversation back to Public status. This allows the attacker to bypass the owner's privacy settings and regain access to the chat content at any time.

**Impact**

1. **Unauthorized Access to Private Chats:** Attackers can force-expose conversations that users explicitly marked as private.
2. **Data Leakage:** Sensitive information, prompts, or AI responses contained within a "Private" chat can be viewed by unauthorized parties.
3. **Broken Privacy Controls:** The "Private/Only Me" setting becomes ineffective against an attacker who has the conversation's `share_id`.

**Repro Steps**
1. **Victim Side:** Create a new chat on monica.im and click the "Share" button to generate a public link.
   *Example Link:* `https://monica.im/share/chat?shareId=0XObvzcBOwiMNQCi`
2. **Victim Side:** The victim then decides to make the chat Private (Only Me) to stop others from seeing it. At this point, the link should return a 404 or access denied.
3. **Attacker Side:** The attacker, using their own account/session, sends the following POST request to the API, using the victim's `shareId`:

```http
POST /api/chat_share/update_share HTTP/2
Host: api.monica.im
Content-Type: application/json
Authorization: [Attacker_Session_Token]

{
  "permission": "public",
  "share_id": "0XObvzESWSOwiMNQCi"
}
```

4. **Result:** The server accepts the request and changes the conversation status back to Public. The attacker can now view the victim's private conversation again using the original share link.

---

#### Vulnerability 2: Unauthorized Chat Title Modification

**Description**
During further testing on the same endpoint, I discovered that `/api/chat_share/update_share` also allows for unauthorized modification of the chat Title.

**Bypass Details**
By replacing the `permission` parameter with `title` in the JSON payload, an attacker can rename any conversation using its `shareId`. This works regardless of whether the conversation is currently set to Public or Private, confirming a total lack of ownership validation on this metadata.

**PoC Request:**

```http
POST /api/chat_share/update_share HTTP/2
Host: api.monica.im
Content-Type: application/json
Authorization: [Attacker_Session_Token]

{
  "share_id": "0XObvzAQOwiMNQCi",
  "title": "Modified by Attacker (IDOR)"
}
```

#### Vulnerability 3: AI Audio-to-Text Privacy Manipulation

**Description**
This IDOR is not limited to chat sharing but is a systemic issue affecting other tools as well, such as the Audio-to-Text service (`https://monica.im/tools/ai-audio-to-text`).

The same Broken Access Control pattern was identified in the `/api/audio_tool/edit_transcript_task` endpoint. The system allows an unauthorized user to change the privacy settings of any transcription task by knowing its `uid`.

By manipulating the `all_can_visits` boolean parameter in the request body, an attacker can:
* Set it to `true` -> Make a private transcript Public.
* Set it to `false` -> Make a public transcript Private (Only Me).

**PoC Request:**

```http
POST /api/audio_tool/edit_transcript_task HTTP/2
Host: api.monica.im
Content-Type: application/json
Authorization: [Attacker_Session_Token]

{
  "all_can_visits": true,
  "uid": "c7948003-e05a-4300-b54b-3a32616f5"
}

```

### 📅 Timeline

* **Reported:** February 4, 2026
* **Triaged:** February 16, 2026
* **Fixed:** April 18, 2026
* **Reward:** April 24, 2026
