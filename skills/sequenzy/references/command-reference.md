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

- starts device auth against `POST /api/device-auth/initiate`
- polls `POST /api/device-auth/poll`
- opens `${SEQUENZY_APP_URL}/setup/auth?code=...` in the browser
- stores the API key in `Bun.secrets` when available, otherwise in local config

### Non-interactive auth

Set `SEQUENZY_API_KEY` in the environment. `packages/cli/src/config.ts` checks this before local storage, so it is the safest path for automation.

### Identity and logout

```bash
sequenzy whoami
sequenzy account
sequenzy logout
```

Behavior:

- `whoami` prints cached local config only
- `account`: `GET /api/v1/account`
- `logout` removes locally stored auth

Caveat:

- treat `whoami` as "is this machine authenticated?" rather than authoritative server-side account discovery

## Environment Variables

```bash
SEQUENZY_API_KEY=...
SEQUENZY_API_URL=https://api.sequenzy.com
SEQUENZY_APP_URL=https://sequenzy.com
```

Notes:

- `SEQUENZY_API_KEY` overrides local keychain/config state
- the current CLI code defaults `SEQUENZY_APP_URL` to `https://sequenzy.com`
- many company-scoped commands accept `--company`, which sends `x-company-id` for personal API keys

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
sequenzy subscribers list --tag vip --company comp_123 --json
```

Behavior:

- sends `GET /api/v1/subscribers`
- maps `--segment` to `segmentId`
- maps `--tag` to `tags`
- maps `--limit` to `limit`
- fetches every result page by default when `--limit` is omitted
- supports `--company` and `--json`

### Add

```bash
sequenzy subscribers add user@example.com
sequenzy subscribers add user@example.com --tag premium --attr name=John --attr plan=pro
sequenzy subscribers add user@example.com --tag premium --tag beta --company comp_123 --json
```

Behavior:

- sends `POST /api/v1/subscribers`
- body shape is `{ email, tags, customAttributes }`
- supports repeated `--tag` values
- supports `--company` and `--json`

### Get

```bash
sequenzy subscribers get user@example.com
sequenzy subscribers get user@example.com --company comp_123 --json
```

Behavior:

- sends `GET /api/v1/subscribers/:email`
- returns the full subscriber profile, including list memberships, sequence enrollments, email stats, and recent activity
- supports `--company` and `--json`

### Remove

```bash
sequenzy subscribers remove user@example.com
sequenzy subscribers remove user@example.com --hard
sequenzy subscribers remove user@example.com --company comp_123 --json
```

Behavior:

- without `--hard`, sends `PATCH /api/v1/subscribers/:email` with `{ status: "unsubscribed" }`
- with `--hard`, sends `DELETE /api/v1/subscribers/:email`
- supports `--company` and `--json`

## Transactional Send

### Template-based

```bash
sequenzy send user@example.com --template tmpl_123 --var name=John
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

## Companies, Lists, Tags, And Segments

### Companies

```bash
sequenzy companies list
sequenzy companies get comp_123
sequenzy companies create example.com --name Example
```

Behavior:

- `companies list`: `GET /api/v1/companies`
- `companies get`: `GET /api/v1/companies/:id`
- `companies create`: `POST /api/v1/companies`

### Lists

```bash
sequenzy lists list
sequenzy lists create Newsletter --description "Public newsletter list"
sequenzy lists create VIP --private --company comp_123
```

Behavior:

- `lists list`: `GET /api/v1/lists`
- `lists create`: `POST /api/v1/lists`
- create body shape is `{ name, description, isPrivate }`

### Tags

```bash
sequenzy tags
sequenzy tags --company comp_123 --json
```

Behavior:

- sends `GET /api/v1/tags`
- this is list-only; there are no tag mutation commands in the current CLI

### Segments

```bash
sequenzy segments list
sequenzy segments count seg_123
sequenzy segments create --name "Bought Pro" --stripe-product prod_pro
sequenzy segments create --name "3+ Pro Payments" --stripe-product prod_pro --purchase-operator at-least --payments 3
```

Behavior:

- `segments list`: `GET /api/v1/segments`
- `segments count`: `GET /api/v1/segments/:id/count`
- `segments create`: `POST /api/v1/segments`
- `--filter-json` accepts the raw segment filter array used by the API/MCP
- Stripe product filters use `field: "stripeProduct"` and product IDs, not product names
- threshold operators encode the count as `productId:count`, for example `prod_pro:3`

## Templates

```bash
sequenzy templates list
sequenzy templates get tmpl_123
sequenzy templates create welcome --subject "Welcome" --html-file ./welcome.html
sequenzy templates update tmpl_123 --subject "Updated" --html-file ./welcome-v2.html
sequenzy templates delete tmpl_123
```

Behavior:

- `templates list`: `GET /api/v1/templates`
- `templates get`: `GET /api/v1/templates/:id`
- `templates create`: `POST /api/v1/templates`
- `templates update`: `PUT /api/v1/templates/:id`
- `templates delete`: `DELETE /api/v1/templates/:id`

Caveats:

- create requires `name`, `subject`, and `html`
- update accepts `name`, `subject`, and `html` only
- HTML content is stored as a single text block by the current API path
- deletion can fail if the template is still referenced by a campaign or sequence

## Campaigns

```bash
sequenzy campaigns list
sequenzy campaigns list --status draft --company comp_123
sequenzy campaigns get camp_123
sequenzy campaigns create "April Launch" --subject "We shipped" --html-file ./campaign.html
sequenzy campaigns update camp_123 --subject "Updated subject"
sequenzy campaigns update camp_123 --reply-to support@example.com
sequenzy campaigns update camp_123 --reply-profile reply_123
sequenzy campaigns test camp_123 --to you@example.com
```

Behavior:

- `campaigns list`: `GET /api/v1/campaigns`
- `campaigns get`: `GET /api/v1/campaigns/:id`
- `campaigns create`: `POST /api/v1/campaigns`
- `campaigns update`: `PUT /api/v1/campaigns/:id`
- `campaigns test`: `POST /api/v1/campaigns/:id/test`

Caveats:

- create supports `name`, `subject`, and `html`
- update supports `name`, `subject`, `html`, `--reply-to`, and `--reply-profile`
- `--reply-to` resolves an existing reply profile by email and `--reply-profile` sets it directly by ID
- `--reply-to` and `--reply-profile` are mutually exclusive
- `campaigns get` now includes saved reply-to details when the campaign has a reply profile
- only draft campaigns can be updated through this API path
- there is no CLI command for sending, scheduling, pausing, or cancelling campaigns
- in the current backend checkout, `campaigns test` returns a success message path rather than a confirmed email send

## Sequences

```bash
sequenzy sequences list
sequenzy sequences get seq_123
sequenzy sequences create onboarding --trigger event_received --event-name signup.completed --goal "Guide new users to activation" --email-count 4
sequenzy sequences create onboarding --trigger contact_added --list-id list_123 --steps-file ./steps.json
sequenzy sequences update seq_123 --steps-file ./sequence-updates.json
sequenzy sequences enable seq_123
sequenzy sequences disable seq_123
sequenzy sequences delete seq_123
```

Behavior:

- `sequences list`: `GET /api/v1/sequences`
- `sequences get`: `GET /api/v1/sequences/:id`
- `sequences create`: `POST /api/v1/sequences`
- `sequences update`: `PUT /api/v1/sequences/:id`
- `sequences enable`: `POST /api/v1/sequences/:id/enable`
- `sequences disable`: `POST /api/v1/sequences/:id/disable`
- `sequences delete`: `DELETE /api/v1/sequences/:id`

Caveats:

- CLI sequence creation supports either AI `--goal` mode or explicit `--steps-json` / `--steps-file` mode
- `--email-count` is only meaningful with `--goal`
- trigger-specific options depend on `--trigger`
- updates accept either step payloads or email payloads via `--steps-*` or `--emails-*`

## API Keys

```bash
sequenzy api-keys create
sequenzy api-keys create --name "CI deploy key" --company comp_123
```

Behavior:

- sends `POST /api/v1/api-keys`
- body shape is `{ name }`

Caveat:

- the plain API key is returned only at creation time; save it immediately

## Websites

```bash
sequenzy websites list --company comp_123
sequenzy websites add example.com --company comp_123
sequenzy websites check example.com --company comp_123
sequenzy websites guide --framework nextjs --use-case transactional
```

Behavior:

- `websites list`: `GET /api/v1/websites`
- `websites add`: `POST /api/v1/websites`
- `websites check`: `GET /api/v1/websites/:domain`
- `websites guide`: `POST /api/v1/integration-guide`

## Commands To Treat As Unsupported

The following command group is intentionally placeholder-only in the current CLI:

- `generate`

Also treat these requested workflows as unsupported in the CLI even though related nouns exist:

- campaign send, schedule, pause, cancel, or resume flows
- bulk subscriber import
- tag creation, update, or deletion
- list update or deletion

## Operational Caveats

- prefer `SEQUENZY_API_KEY` for automation instead of interactive login
- use `--json` when another tool or agent needs the raw response
- when the user asks for a workflow outside the current CLI surface, say so directly and choose between dashboard or direct API use instead of inventing commands
