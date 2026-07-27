---
name: Review a contract with a GC AI Playbook
description: Upload a contract, run a review Playbook against it, and read back structured check results.
api: openapi/gc-ai-openapi-original.yml
operations: [uploadFile, getFileStatus, listPlaybooks, runPlaybook, getAsyncJob]
---

# Review a contract with a GC AI Playbook

Use this skill to run GC AI's contract-review Playbooks against an uploaded document.

## Auth
- Send `Authorization: gcai_...` (org-scoped) or `Authorization: u:gcai_...` (user-scoped) on every request.
- Base URL: `https://app.gc.ai/api/external/v1`.

## Steps
1. **Upload the contract.** `uploadFile` (multipart form data). Capture the returned file `id`.
2. **Wait for processing.** Poll `getFileStatus` (`GET /files/{id}`) until `status` is `ready` — extraction/embedding runs asynchronously.
3. **Pick a Playbook.** `listPlaybooks` to find the review Playbook you want (or create one with the playbook/check operations first).
4. **Run the review.** `runPlaybook` (`POST /playbooks/{id}/run`) with the file id(s). If it returns `202` with a `job_id`, poll `getAsyncJob` (`GET /jobs/{id}`) until complete.
5. **Read results.** The result carries per-check findings (severity, clause references, suggested redlines).

## Rules
- **Rate limits:** playbook runs are inference-tier — ~1/min per org, burst 3 over 180s. On `429` (code `RATE_LIMITED`) honor `Retry-After`; do not tight-loop.
- **Errors:** branch on the machine-readable `code` field, not the human `error` string (see `errors/gc-ai-problem-types.yml`).
- **Async:** treat `202` as normal; poll the job rather than retrying the request.
