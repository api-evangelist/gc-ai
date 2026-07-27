---
name: Organize a matter as a GC AI Project
description: Create a Project, attach files, and run project-scoped chats around a single deal or vendor.
api: openapi/gc-ai-openapi-original.yml
operations: [createProject, uploadFile, attachFileToProject, listProjectFiles, createChatCompletion]
---

# Organize a matter as a GC AI Project

Group all files, folders, and chats for one deal/vendor/program under a Project.

## Auth
- Header `Authorization: gcai_...` or `u:gcai_...`. Base URL `https://app.gc.ai/api/external/v1`.

## Steps
1. **Create the project.** `createProject` (`POST /projects`) with a name and optional custom instructions. Capture the project `id`.
2. **Upload documents.** `uploadFile`; poll `getFileStatus` until `ready`.
3. **Attach files.** `attachFileToProject` (`POST /projects/{id}/files`) — this records the link; the file is not moved.
4. **Confirm contents.** `listProjectFiles` (`GET /projects/{id}/files`) to verify what is attached.
5. **Work the matter.** `createChatCompletion` referencing the project context for grounded answers.

## Rules
- Detaching a file/folder from a project does **not** delete it — it stays in the org and can be reattached (idempotent).
- Standard CRUD endpoints allow 120 req/min per org; chat completions are inference-tier (~1/min, burst 3).
- Org-scoped keys cannot reach privately shared or access-controlled resources — use a user-scoped `u:gcai_...` key for those.
