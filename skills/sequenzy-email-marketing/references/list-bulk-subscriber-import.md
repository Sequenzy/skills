# Bulk add subscribers to Sequenzy lists

Use this when a user needs to populate subscriber lists for campaign targeting or asks about `lists import`, list membership imports, or `/subscribers/bulk` failures.

## Current CLI/API shape

Published `@sequenzy/cli` 0.0.33+ exposes:

```bash
sequenzy lists add-subscribers <listId> \
  --emails-file ./batch.csv \
  --duplicate-strategy skip \
  --opt-in-mode default \
  --company <companyId> \
  --json
```

Older global installs can be stale. If `sequenzy lists --help` only shows `list` and `create`, use:

```bash
npx -y @sequenzy/cli@latest lists add-subscribers <listId> --emails-file ./batch.csv --company <companyId> --json
```

or upgrade:

```bash
npm install -g @sequenzy/cli@latest
sequenzy --version
```

## API endpoint

Correct endpoint:

```http
POST https://api.sequenzy.com/api/v1/lists/{listId}/subscribers
```

There is no wired `/api/v1/lists/{listId}/subscribers/bulk` endpoint in the CLI/API surface inspected in this session; that path returns `404 NOT_FOUND`.

Headers:

```http
Authorization: Bearer $SEQUENZY_API_KEY
Content-Type: application/json
x-company-id: <companyId>   # needed for personal API keys / company override
```

Payload:

```json
{
  "emails": ["one@example.com", "two@example.com"],
  "duplicateStrategy": "skip",
  "enrollInSequences": false,
  "optInMode": "default"
}
```

Valid values:

- `duplicateStrategy`: `skip`, `merge`, `overwrite`
- `optInMode`: `default`, `confirmed`, `double_opt_in`
- `enrollInSequences`: boolean

Do not send `subscribers` as the direct API payload key for this endpoint; the CLI/file parser accepts JSON objects with `emails` or `subscribers`, but the outgoing API request uses `emails`.

## Chunking and timeout avoidance

The CLI chunks `emails` into batches of 100 and posts each chunk sequentially. For ~325 subscribers in a list, expect four requests: `100 + 100 + 100 + 25`.

If single-add commands time out around dozens of requests, prefer `lists add-subscribers` or direct API chunks of <=100 emails rather than one call per subscriber.

## File formats accepted by `--emails-file`

- Newline-separated emails.
- CSV/TSV/semicolon-delimited file with a recognized email column header.
- CSV without a recognized header: first column is used.
- JSON array of strings or objects with `email`.
- JSON object with `emails` or `subscribers` array.

Recognized email headers are case-insensitive:

```txt
email
e-mail
email address
mail
```

Example CSV:

```csv
email
one@example.com
two@example.com
```

Example direct curl:

```bash
curl -X POST "https://api.sequenzy.com/api/v1/lists/$LIST_ID/subscribers" \
  -H "Authorization: Bearer $SEQUENZY_API_KEY" \
  -H "Content-Type: application/json" \
  -H "x-company-id: $COMPANY_ID" \
  -d '{
    "emails": ["one@example.com", "two@example.com"],
    "duplicateStrategy": "skip",
    "enrollInSequences": false,
    "optInMode": "default"
  }'
```
