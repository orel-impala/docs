---
title: "Upload and run your first batch"
description: "This guide shows how to submit batch inference to your Impala endpoint using the OpenAI-compatible API. You’ll upload an input file, then create a batch."
---

## Prerequisites

Impala will provide these values to you ahead of time:

- **`BASE_URL`** — the hostname of your Impala endpoint
- **`JOB_ID`** — an identifier reserved for your account (required when creating a batch)
- Create an API key in Settings on app.getimpala.ai, and send it as a bearer token on every request: `-H "Authorization: Bearer $IMPALA_API_KEY"`. (BYOC deployments can disable API keys; if yours is disabled, omit the header.)

Use these values exactly as provided.

---

## Step 1 — Prepare your input file (JSONL)

The input file must be **JSONL** (one JSON object per line). Each line represents one request to the inference endpoint you want to target (e.g. `/v1/chat/completions`).

Example `input.jsonl` for chat completions:

```text
{"custom_id":"req-1","method":"POST","url":"/v1/chat/completions","body":{"model":"your-model","messages":[{"role":"user","content":"Summarize: ..."}]}}
{"custom_id":"req-2","method":"POST","url":"/v1/chat/completions","body":{"model":"your-model","messages":[{"role":"user","content":"Translate: ..."}]}}
```

---

## Step 2 — Upload the input file

Upload your JSONL file using the OpenAI-compatible Files API:

```bash
curl -X POST "$BASE_URL/v1/files" \
  -F "purpose=batch" \
  -F "file=@input.jsonl"
```

Response (truncated):

```json
{
  "id": "file-abc123",
  "object": "file",
  "bytes": 12345,
  "filename": "input.jsonl",
  "purpose": "batch"
}
```

Save the `id` — you’ll use it as `input_file_id` in the next step.

---

## Step 3 — Create the batch

```bash
curl -X POST "$BASE_URL/v1/batches" \
  -H "Content-Type: application/json" \
  -d '{
    "input_file_id": "file-abc123",
    "endpoint": "/v1/chat/completions",
    "completion_window": "unlimited",
    "job_id": "job-xxxxxxxx"
  }'
```

Required request fields:

| Field | Value |
| --- | --- |
| input\_file\_id | The id returned by Step 2 |
| endpoint | One of /v1/chat/completions, /v1/completions, /v1/embeddings, /v1/responses — must match the url field in each JSONL line |
| completion\_window | "unlimited" |
| job\_id | The Job ID supplied to you by Impala (job-xxxxxxxx) |

The response includes an `id` for the batch (for example, `batch-xyz789`).

---

## Step 4 — Poll for status

```bash
curl "$BASE_URL/v1/batches/batch-xyz789"
```

The batch `status` will move through:

`validating` → `in_progress` → `finalizing` → `completed` (or `failed` / `expired`)

When the batch is `completed`, the response includes:

- `output_file_id` — a file containing successful responses
- `error_file_id` — a file containing failed requests (only present if any requests failed)

---

## Step 5 — Download results

```bash
curl "$BASE_URL/v1/files/<output_file_id>/content" > output.jsonl
```

The output file is JSONL: one response per line, matched to your input using `custom_id`.

---

## Using the OpenAI Python SDK

The endpoint is OpenAI-compatible. Point `base_url` to your Impala endpoint:

```python
from openai import OpenAI

client = OpenAI(
  base_url="<BASE_URL>/v1",  # use your Impala base URL
  api_key="reserved",        # placeholder (not used today)
)

# 1. Upload
with open("input.jsonl", "rb") as f:
  file = client.files.create(file=f, purpose="batch")

# 2. Create batch
batch = client.batches.create(
  input_file_id=file.id,
  endpoint="/v1/chat/completions",
  completion_window="unlimited",
  extra_body={"job_id": "job-xxxxxxxx"},
)

# 3. Poll
batch = client.batches.retrieve(batch.id)
print(batch.status)

# 4. Download when completed
if batch.status == "completed":
  content = client.files.content(batch.output_file_id)
  with open("output.jsonl", "wb") as f:
    f.write(content.content)
```

Note: `job_id` is required by Impala. Since the OpenAI SDK doesn’t expose a `job_id` parameter, pass it via `extra_body` as shown.

---

## API reference

- `POST /v1/files` — upload
- `GET  /v1/files/{file_id}` — metadata
- `GET  /v1/files/{file_id}/content` — download
- `POST /v1/batches` — create
- `GET  /v1/batches/{batch_id}` — status
- `GET  /v1/batches` — list

## Need help?

Reach out to your Impala contact directly.