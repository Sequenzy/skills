# Command Reference

## Source Of Truth

- Command registration: `packages/cli/src/index.tsx`
- Auth storage and config: `packages/cli/src/config.ts`
- HTTP requests: `packages/cli/src/api.ts`
- Implemented handlers: `packages/cli/src/commands/`

If docs and code disagree, trust the code.

## Authentication

### Interactive login

```bash
sequenzy login
```

- Starts device auth against `POST /api/device-auth/initiate`
- Polls `POST /api/device-auth/poll`
- Opens `${SEQUENZY_APP_URL}/setup/auth?code=...` in the browser
- Stores the API key in `Bun.secrets` when available, otherwise in local config

### Non-interactive auth

Set `SEQUENZY_API_KEY` in the environment. `packages/cli/src/config.ts` checks this before local storage, so it is the safest path for automation.

### Identity and logout

```bash
sequenzy whoami
sequenzy logout
```

Caveat: `whoami` prints cached local config, not a fresh API lookup. It is good for "am I authenticated here?" but not for authoritative account discovery.

## Environment Variables

```bash
SEQUENZY_API_KEY=...
SEQUENZY_API_URL=https://api.sequenzy.com
SEQUENZY_APP_URL=https://sequenzy.com
```

Notes:

- `SEQUENZY_API_KEY` overrides local keychain/config state
- the current CLI code defaults `SEQUENZY_APP_URL` to `https://sequenzy.com`
- older docs may mention `https://app.sequenzy.com`; do not assume that is current

## Stats

```bash
sequenzy stats
sequenzy stats --period 30d
sequenzy stats --campaign camp_123
sequenzy stats --sequence seq_123
```

Behavior:

- no ID: `GET /api/v1/metrics?period=7d|30d|90d`
- `--campaign`: `GET /api/v1/metrics/campaigns/:id`
- `--sequence`: `GET /api/v1/metrics/sequences/:id`

Output includes:

- `sent`
- `delivered`
- `opened`
- `clicked`
- `unsubscribed`
- `openRate`
- `clickRate`

## Subscribers

### List

```bash
sequenzy subscribers list
sequenzy subscribers list --tag vip
sequenzy subscribers list --segment seg_123
sequenzy subscribers list --limit 100
```

Behavior:

- sends `GET /api/v1/subscribers`
- maps `--segment` to `segmentId`
- maps `--tag` to `tags`
- maps `--limit` to `limit`

### Add

```bash
sequenzy subscribers add user@example.com
sequenzy subscribers add user@example.com --tag premium --attr name=John --attr plan=pro
```

Behavior:

- sends `POST /api/v1/subscribers`
- body shape is `{ email, tags, attributes }`

Caveat:

- The current handler treats `--tag` as a single value in practice. Do not rely on multiple tags in one call unless the CLI implementation changes.

### Get

```bash
sequenzy subscribers get user@example.com
```

Behavior:

- sends `GET /api/v1/subscribers/:email`
- prints the raw JSON payload

### Remove

```bash
sequenzy subscribers remove user@example.com
sequenzy subscribers remove user@example.com --hard
```

Behavior:

- sends `DELETE /api/v1/subscribers/:email`
- body is `{ hardDelete: boolean }`
- without `--hard`, the CLI frames this as unsubscribe

## Transactional Send

### Template-based

```bash
sequenzy send user@example.com --template welcome --var name=John
```

### Raw HTML

```bash
sequenzy send user@example.com --subject "Hello" --html "<h1>Hi</h1>"
sequenzy send user@example.com --subject "Hello" --html-file ./email.html
```

Behavior:

- sends `POST /api/v1/transactional/send`
- body shape is `{ to, templateId, subject, html, variables }`

Validation enforced by the CLI:

- require either `--template` or `--html`/`--html-file`
- require `--subject` when sending raw HTML

## Commands To Treat As Unsupported

The following command groups appear in registration or docs but do not currently attach handlers in `packages/cli/src/index.tsx`:

- `campaigns`
- `sequences`
- `templates`
- `tags`
- `lists`
- `segments`
- `account`
- `websites`
- `generate`

Do not tell an agent to execute these as if they are working end-to-end.

## Operational Caveats

- The CLI has no implemented company-selection flow yet.
- `whoami` mentions `sequenzy account`, but `account` is currently just a placeholder command.
- Bulk subscriber import is not exposed in the current CLI. Use the dashboard or API instead.
- Marketing campaign creation and sequence management are not currently usable through the implemented CLI handlers.
