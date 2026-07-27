---
name: Ask GC AI a question grounded in uploaded documents
description: Upload files, ask a chat question that uses them as context, and optionally surface the chat into history.
api: openapi/gc-ai-openapi-original.yml
operations: [uploadFile, getFileStatus, createChatCompletion, getAsyncJob, materializeChat]
---

# Ask GC AI a question grounded in uploaded documents

Use this skill to get a cited, document-grounded answer from GC AI.

## Auth
- Header `Authorization: gcai_...` or `u:gcai_...`. Base URL `https://app.gc.ai/api/external/v1`.

## Steps
1. **Upload context (optional).** `uploadFile` for each document; poll `getFileStatus` until `ready`.
2. **Ask.** `createChatCompletion` (`POST /chat/completions`) with your message and the `file_ids`. Returns `200` inline if it finishes within the wait window, otherwise `202` with a `job_id`.
3. **Poll if async.** On `202`, poll `getAsyncJob` (`GET /jobs/{id}`) until the completion is ready.
4. **Continue the conversation.** Pass the returned `chat_id` back on the next `createChatCompletion` to keep multi-turn context.
5. **Surface the chat (optional).** API chats stay private; call `materializeChat` (`POST /chat/{id}/materialize`) to add it to chat history and get a shareable deep link.

## Rules
- **Inference rate limit:** ~1/min per org, burst 3; honor `Retry-After` on `429`.
- **Interactive clarification is disabled on the API** in beta — the model will not pause to ask follow-up questions.
- Branch on the error `code` (e.g. `INSUFFICIENT_CREDITS`, `RATE_LIMITED`).
