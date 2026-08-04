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

## Dashboard URLs

```bash
sequenzy urls --company comp_123
sequenzy urls --company comp_123 --sequence seq_123
sequenzy urls --company comp_123 --campaign camp_123
sequenzy urls --company comp_123 --template tmpl_123
sequenzy urls --company comp_123 --landing-page lp_123
sequenzy urls --company comp_123 --transactional tx_123
sequenzy urls --company comp_123 --email-send send_123
sequenzy urls --company comp_123 --settings-tab integrations
sequenzy urls --company comp_123 --json
```

Behavior:

- uses `SEQUENZY_APP_URL` as the base URL, defaulting to `https://sequenzy.com`
- if `--company` is omitted, tries the current company from `GET /api/v1/account`
- returns route templates, settings tab values, and concrete URLs when a company ID is known
- also accepts `--email <id>` (email editor), `--domain <id>` (sending domain), and `--status` for campaign/sequence list URLs
- campaign, sequence, template, company, and account outputs include `url` or `appUrls` fields when the company can be resolved

Common route patterns:

- sequence editor: `/dashboard/company/{companyId}/sequences/{sequenceId}`
- campaign editor: `/dashboard/company/{companyId}/campaign/{campaignId}`
- template/email editor: `/dashboard/company/{companyId}/emails/{emailId}`
- settings: `/dashboard/company/{companyId}/settings`
- settings tab: `/dashboard/company/{companyId}/settings?tab={tab}`

## Stats

```bash
sequenzy stats
sequenzy stats --period 30d
sequenzy stats --start "2026-05-01T00:00:00Z" --end "2026-05-31T00:00:00Z"
sequenzy stats --campaign camp_123
sequenzy stats --sequence seq_123
sequenzy stats --transactional password-reset
sequenzy stats --email-type transactional
sequenzy stats --campaign camp_123 --mailbox-provider gmail
sequenzy stats --campaign camp_123 --include-bots
```

Behavior:

- no ID: `GET /api/v1/metrics?period=7d|30d|90d`, or a custom range with both `--start` and `--end`
- `--campaign`: `GET /api/v1/metrics/campaigns/:id`
- `--sequence`: `GET /api/v1/metrics/sequences/:id`
- `--transactional <id-or-slug>` reports one saved transactional email; its stats are all-time by default, pass `--period` for a window
- use exactly one of account-level stats, `--campaign`, `--sequence`, or `--transactional`; `--email-type` (e.g. `transactional`) applies only to account-level stats
- `--mailbox-provider` (e.g. `gmail`, `microsoft`, `yahoo`) scopes account or campaign engagement metrics to one recipient mailbox provider; replies, conversions, and revenue cannot be segmented per provider and report 0 under this filter
- bot/scanner opens and clicks are excluded by default; pass `--include-bots` to inspect raw engagement
- sequence stats include a live active/waiting enrollment breakdown by current node plus per-step failures

Output includes:

- `sent`
- `delivered`
- `opened`
- `clicked`
- `unsubscribed`
- `openRate`
- `clickRate`

Campaign stats also include a `clickedLinks` array when the campaign has tracked link clicks: the top 20 destination URLs, most clicked first, each with `url`, `clicks`, and `percentage` (that link's share of every recorded link click). The CLI prints these as a Clicked Links section; use `--json` for the raw array. The MCP `get_campaign_stats` tool returns the same top-level array.

Campaign stats also include a `polls` array when the campaign collected poll / NPS answers: one entry per poll block with `question`, `totalResponses`, per-answer counts and percentages, and an `nps` breakdown (`score`, `average`, promoters / passives / detractors) for NPS surveys.

## Events

```bash
sequenzy events --campaign camp_123
sequenzy events --sequence seq_123 --event-types delivery,open,click
sequenzy events --campaign camp_123 --type open --period 24h --page 2 --limit 100
```

Behavior:

- `--campaign`: `GET /api/v1/metrics/campaigns/:id/events`; `--sequence`: `GET /api/v1/metrics/sequences/:id/events`; use exactly one
- defaults to delivery events; use `--type` for a single event type or `--event-types` for a comma-separated set (delivery, open, click, ...)
- accepts `--period` (`1h`, `24h`, `7d`, `30d`, `90d`) or `--start`/`--end`, plus `--page` and `--limit` pagination
- bot/scanner opens and clicks are excluded by default; pass `--include-bots` to inspect raw engagement
- MCP parity: `list_campaign_events` and `list_sequence_events`

## Subscribers

### List

```bash
sequenzy subscribers list
sequenzy subscribers list --tag vip
sequenzy subscribers list --list "Master List" --json
sequenzy subscribers list --segment seg_123
sequenzy subscribers list --limit 100
sequenzy subscribers list --tag vip --company comp_123 --json
```

Behavior:

- sends `GET /api/v1/subscribers`
- maps `--segment` to `segmentId`
- maps `--tag` to `tags`
- maps `--list` to `list`; the API resolves list ID first, then exact list name
- maps `--limit` to `limit`
- fetches every result page by default when `--limit` is omitted
- supports `--company` and `--json`

### Add

```bash
sequenzy subscribers add user@example.com
sequenzy subscribers add user@example.com --first-name John --last-name Doe --tag premium --attr plan=pro
sequenzy subscribers add user@example.com --phone "+15550100" --sms-consent
sequenzy subscribers add user@example.com --list list_123 --skip-sequences
sequenzy subscribers add user@example.com --created-at "2024-03-01T00:00:00Z"
sequenzy subscribers add user@example.com --tag premium --tag beta --company comp_123 --json
```

Behavior:

- sends `POST /api/v1/subscribers`
- body shape is `{ email, firstName, lastName, tags, customAttributes }`
- `--first-name` / `--last-name` set the native profile name fields; do not store names in `--attr`
- `--phone` (with optional `--phone-country` for national numbers) writes the native phone field, not a custom attribute; pair it with `--sms-consent` to assert express written SMS consent
- `--external-id <id>` stores a customer-owned subscriber ID that other subscriber commands can address with `--external-id`
- `--list <ids...>` adds the subscriber to specific lists, `--no-lists` to none; the default is all lists
- `--skip-sequences` suppresses sequence enrollment for this add
- `--created-at` preserves a contact's real signup date (for migrations) so date-relative segments stay correct; it never moves an existing contact's date later and suppresses sequence enrollment because it describes the past
- supports repeated `--tag` values
- supports `--company` and `--json`

### Update

```bash
sequenzy subscribers update user@example.com --status unsubscribed
sequenzy subscribers update user@example.com --first-name Ana --last-name Silva
sequenzy subscribers update user@example.com --phone "+15550100" --sms-consent
sequenzy subscribers update user@example.com --tag churned --attr cancelReason=too_expensive
sequenzy subscribers update user@example.com --attr plan=pro --merge
```

Behavior:

- sends `PATCH /api/v1/subscribers/:email`
- requires at least one of `--status`, `--first-name`, `--last-name`, `--phone`, `--sms-consent`/`--no-sms-consent`, `--tag`, or `--attr`
- `--first-name` / `--last-name` set the native profile name fields; do not store names in `--attr`
- changing `--phone` resets SMS consent unless `--sms-consent` is passed in the same command; `--no-sms-consent` revokes it
- `--tag` replaces the subscriber's tags; `--attr` replaces custom attributes unless `--merge` is passed
- `--external-id <id>` addresses the subscriber by customer-owned ID instead of email
- supports `--company` and `--json`

### Get

```bash
sequenzy subscribers get user@example.com
sequenzy subscribers get user@example.com --company comp_123 --json
```

Behavior:

- sends `GET /api/v1/subscribers/:email`
- returns the full subscriber profile, including list memberships, sequence enrollments, email stats, and recent activity
- `--external-id <id>` looks the subscriber up by customer-owned ID; `--include-bots` includes bot/scanner engagement in the stats
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
- `--external-id <id>` addresses the subscriber by customer-owned ID
- supports `--company` and `--json`

### Import

```bash
sequenzy subscribers import ./contacts.csv --list list_123
sequenzy subscribers import --records-json '[{"email":"a@example.com","firstName":"Ana","plan":"pro"}]'
sequenzy subscribers import ./contacts.csv --duplicate-strategy merge --opt-in-mode confirmed
sequenzy subscribers import-status imp_123
```

Behavior:

- `import` sends `POST /api/v1/subscribers/imports` and queues a full-record import from CSV, JSON file, or `--records-json`
- CSV columns for email, names, external ID, phone, status, and tags are detected automatically; other columns become custom attributes
- `--duplicate-strategy skip|merge|overwrite` (default `skip`) controls existing-contact behavior; `--list <ids...>` adds every imported subscriber to lists
- `--enroll-sequences` enrolls matching subscribers (requires the `automations:trigger` scope); `--default-phone-country` and `--sms-consent` handle imported phone numbers
- `--opt-in-mode default|confirmed|double_opt_in` sets email consent; use `confirmed` only when the contacts already gave verified consent
- imports run asynchronously; poll the returned ID with `subscribers import-status` (`GET /api/v1/subscribers/imports/:importId`). A completed import can still contain failed rows - inspect failed counts and the `--json` reason maps
- MCP parity: `create_subscriber_import` and `get_subscriber_import`

### Notes

```bash
sequenzy subscribers notes list user@example.com
sequenzy subscribers notes add user@example.com "Asked for annual invoicing"
sequenzy subscribers notes delete note_123
```

Behavior:

- `notes list`: `GET /api/v1/subscribers/:email/notes`; `notes add`: `POST /api/v1/subscribers/:email/notes`; `notes delete`: `DELETE /api/v1/subscribers/notes/:noteId`
- `--external-id` addresses the subscriber by customer-owned ID
- MCP parity: `list_subscriber_notes`, `add_subscriber_note`, and `delete_subscriber_note`

### Trigger Event

```bash
sequenzy subscribers event user@example.com saas.purchase -p amount=99 -p plan=pro
sequenzy subscribers event user@example.com saas.purchase --occurred-at "2025-11-02T10:00:00Z"
```

Behavior:

- sends the named custom event for one subscriber; `-p/--property` sets event properties and `-a/--attr` updates custom attributes in the same call
- without `--occurred-at` the event is live and runs the full chain: sync rules, sequence enrollment, waiting steps, and goal conversions
- with an `--occurred-at` more than an hour in the past the event is recorded as history: stored with its real timestamp and counted by segments, but no sequences enroll and no webhooks fire; the contact's signup date also moves back to when the event occurred (never later). Use it for backfills, never to fake a live event
- `--event-id` provides an idempotency key
- MCP parity: `trigger_subscriber_event` and `trigger_subscriber_events` (bulk)

### Bulk Tags

```bash
sequenzy subscribers tags add vip --email one@example.com two@example.com
sequenzy subscribers tags add imported-2026 --emails-file ./batch.csv
sequenzy subscribers tags remove trial --emails-json '["one@example.com"]'
```

Behavior:

- sends `POST /api/v1/subscribers/bulk/tags/add|remove`; targets come from repeated `--email`, `--emails-json`, `--emails-file`, `--external-id`, or `--subscriber-id`
- only touches contacts that already exist; unknown identifiers are reported, never created
- tag automations are skipped by default (what historical backfills want); pass `--trigger-automations` to fire `tag_added` sequences
- requests are chunked at 500 targets each; totals in the output are combined across chunks
- use `subscribers add`/`subscribers update` for single-contact tag changes
- MCP parity: `bulk_add_subscriber_tags` and `bulk_remove_subscriber_tags`

## Transactional Emails

### Send

```bash
sequenzy send user@example.com --template welcome --var firstName=Ana
sequenzy send user@example.com --subject "Hello" --html "<h1>Hi</h1>"
sequenzy send user@example.com --subject "Hello" --html-file ./email.html
sequenzy send user@example.com --subject "Hello" --html-file ./email.html --email-type marketing
```

Behavior:

- sends `POST /api/v1/transactional/send`
- body shape is `{ to, templateId, subject, html, variables }`
- use either `--template` (a transactional API slug, cannot be combined with `--subject`) or `--html`/`--html-file` with a required `--subject`
- `--email-type marketing` adds the standard unsubscribe footer and RFC 8058 one-click-unsubscribe headers; it supports one recipient only
- `--reply-to` overrides the saved template or company Reply-To default
- `--no-track-clicks` / `--no-track-opens` opt out of tracking for this send only; they cannot enable tracking the account has disabled
- `--vars-json` accepts nested objects and arrays (including arrays for repeat blocks); `--var` overrides individual scalar keys
- if the recipient matches a subscriber, saved first and last names fill omitted name variables; `--var` values override them
- MCP parity: `send_email`

### Saved Transactional Templates

```bash
sequenzy transactional list --status enabled
sequenzy transactional get welcome
sequenzy transactional create password-reset --prompt "Write a password reset email"
sequenzy transactional create welcome --subject "Welcome" --html-file ./welcome.html
sequenzy transactional update welcome --no-enabled
sequenzy transactional delete welcome --yes
```

Behavior:

- `list`: `GET /api/v1/transactional` with `--search`, `--status`, `--sort`, `--order`, and `--include-bots` filters
- `get <slug>`: `GET /api/v1/transactional/:slug`
- `create <name>`: `POST /api/v1/transactional`; provide exactly one of `--prompt`, HTML, or blocks; `--slug` sets the API slug used by `sequenzy send --template`
- `update <slug>` is a partial update: only the fields you pass change; replacing the body takes HTML or blocks, not both; `--no-enabled` stops sends while keeping the template and its stats
- `delete <slug>` frees the slug for reuse and stops further sends; past deliveries and stats are kept; requires the `transactional:delete` scope
- `sequenzy transactional widgets preferences-token <email>` generates a subscription preferences iframe token
- per-template stats: `sequenzy stats --transactional <id-or-slug>`
- MCP parity: `list_transactional_emails`, `get_transactional_email`, `create_transactional_email`, `update_transactional_email`, `delete_transactional_email`, and `get_transactional_stats`

## Email Sends

```bash
sequenzy email-sends list --recipient user@example.com --days 7
sequenzy email-sends list --status bounced --email-type transactional
sequenzy email-sends list --all --csv email-sends.csv
sequenzy email-sends get send_123
sequenzy email-sends get send_123 --html-only
```

Behavior:

- `list`: `GET /api/v1/email-sends`; matches the dashboard's 14-day delivery history with `--search`, `--subject`, `--recipient`, `--status`, `--email-type`, `--bounce-type`, `--campaign`, `--transactional`, `--sequence`, `--days`, paging, and sort filters
- the `opened` status includes clicked deliveries, matching the dashboard
- `--all --csv <path>` exports every matching delivery
- `get <emailSendId>`: `GET /api/v1/email-sends/:id`; returns timestamps and the complete event timeline; `--html` prints the stored HTML body when still available, `--html-only` prints just the HTML
- email send rows are short-lived; if a row was cleaned up, `get` falls back to retained ClickHouse events
- MCP parity: `list_email_sends` and `get_email_send`

## Suppressions

```bash
sequenzy suppressions get user@example.com
sequenzy suppressions remove user@example.com --yes
```

Behavior:

- `get`: `GET /api/v1/suppressions/:email`, optionally scoped with `--region`; `remove`: clears the suppression
- lookup is exact-address only because the SES account-level list is shared across companies
- removal clears stale bounce entries only; complaint and unsubscribe protections are preserved
- MCP parity: `get_recipient_suppression` and `remove_recipient_suppression`

## Companies, Lists, Tags, And Segments

### Companies

```bash
sequenzy companies list
sequenzy companies get comp_123
sequenzy companies create example.com --name Example
sequenzy companies update comp_123 --primary-color '#0ea5e9' --tone-voice "clear, direct, warm"
sequenzy companies update comp_123 --company-context-file ./product.md
sequenzy companies update comp_123 --sender-profile-id sender_abc --from-name "Example News"
sequenzy companies update comp_123 --enable-reply-tracking --reply-tracking-domain-mode sequenzy
```

Behavior:

- `companies list`: `GET /api/v1/companies`
- `companies get`: `GET /api/v1/companies/:id`
- `companies create`: `POST /api/v1/companies`; takes a domain plus optional `--name` and queues website processing automatically
- `companies update`: `PATCH /api/v1/companies/:id` covering brand fields (`--description`, `--logo-url`, `--primary-color`, `--brand-colors-json`, `--font-family`, `--email-theme-json`, `--email-direction`), AI context (`--tone-voice`, `--company-context`, `--value-props-json`, `--testimonials-json`, `--pricing-json`, `--email-length-preference`), compliance (`--privacy-policy-url`, `--terms-url`, `--address`, `--language`, `--social-links-json`), and sender identity (`--sender-profile-id`, `--from-email`, `--from-name`, `--reply-profile-id`, `--reply-to`, `--reply-to-name`, `--enable/disable-reply-tracking`, `--reply-tracking-domain-mode`, `--enable/disable-reply-forwarding`)
- most JSON-valued flags have `-file` variants to avoid shell escaping
- MCP parity: `get_company`, `create_company`, and `update_company`

### Lists

```bash
sequenzy lists list
sequenzy lists create Newsletter --description "Public newsletter list"
sequenzy lists create VIP --private --company comp_123
sequenzy lists update list_123 --name "Weekly Newsletter" --private
sequenzy lists update list_123 --no-private
sequenzy lists add-subscribers list_123 --email one@example.com two@example.com
sequenzy lists add-subscribers list_123 --emails-json '["one@example.com","two@example.com"]'
sequenzy lists add-subscribers list_123 --emails-file ./batch-001.csv
sequenzy lists import list_123 --emails-file ./batch-001.csv
sequenzy lists remove-subscribers list_123 --email one@example.com two@example.com
sequenzy lists remove-subscribers list_123 --emails-file ./churned.csv
sequenzy lists delete list_123 --yes
```

Behavior:

- `lists list`: `GET /api/v1/lists`
- `lists create`: `POST /api/v1/lists`
- create body shape is `{ name, description, isPrivate }`
- `lists update`: `PATCH /api/v1/lists/:listId` with at least one of `--name`, `--description`, `--private`, or `--no-private`
- `lists delete`: `DELETE /api/v1/lists/:listId`; removes the list and all of its memberships, reports `removedMemberships`, and keeps the subscribers themselves
- `lists add-subscribers` and `lists import`: `POST /api/v1/lists/:listId/subscribers`
- `lists remove-subscribers`: `POST /api/v1/lists/:listId/subscribers/remove`
- add-subscribers body shape is `{ emails, duplicateStrategy, enrollInSequences, optInMode }`
- remove-subscribers takes the same email input formats as add-subscribers, only removes list memberships, and reports `removed` plus `notFound` emails
- the CLI splits large files into API-safe batches of up to 500 emails for both add and remove
- files may be newline-separated, CSV with an email column, a JSON email array, or a JSON object with `emails` or `subscribers`
- CSV headers named `email`, `e-mail`, `email address`, or `mail` are detected; otherwise the first column is used
- `lists delete` prompts for confirmation; pass `--yes` to skip
- MCP parity: `update_list`, `delete_list`, and `remove_subscribers_from_list` (max 500 emails per call)

### Tags

```bash
sequenzy tags
sequenzy tags list --company comp_123 --json
sequenzy tags create vip --color purple
sequenzy tags update tag_123 --color red
sequenzy tags delete tag_123 --yes
```

Behavior:

- `tags list`: `GET /api/v1/tags`; bare `sequenzy tags` without a subcommand still lists tag definitions for backwards compatibility
- `tags create`: `POST /api/v1/tags` with `{ name, color? }`
- `tags update`: `PATCH /api/v1/tags/:tagId` with `{ color }` (`--color` is required)
- `tags delete`: `DELETE /api/v1/tags/:tagId`
- tag names are normalized to lowercase with dashes, so `VIP Customer` becomes `vip-customer`
- the color defaults to `gray`; valid colors are `gray`, `red`, `orange`, `amber`, `yellow`, `lime`, `green`, `emerald`, `teal`, `cyan`, `sky`, `blue`, `indigo`, `violet`, `purple`, `fuchsia`, `pink`, and `rose`
- system tags cannot be updated or deleted
- tags still referenced by sequences cannot be deleted until those sequences stop using them
- deleting a tag removes it from every subscriber; the delete prompt warns about this, and `--yes` skips it
- MCP parity: `list_tags`, `create_tag`, `update_tag`, and `delete_tag`

### Segments

```bash
sequenzy segments list
sequenzy segments count seg_123
sequenzy segments create --name "Bought Pro" --stripe-product prod_pro
sequenzy segments create --name "3+ Pro Payments" --stripe-product prod_pro --purchase-operator at-least --payments 3
sequenzy segments create --name "VIP or Churn Risk" --match any --filter-json '[{"field":"tag","operator":"contains","value":"vip"},{"field":"emailOpened","operator":"is_not","value":"30d"}]'
sequenzy segments create --name "Active non-paying" --filter-json '{"kind":"group","id":"root","joinOperator":"and","children":[{"kind":"filter","id":"f1","field":"attribute","operator":"gte","value":"last_login_days_ago:0"},{"kind":"group","id":"g1","joinOperator":"or","children":[{"kind":"filter","id":"f2","field":"attribute","operator":"is_empty","value":"plan_end"},{"kind":"filter","id":"f3","field":"attribute","operator":"lt","value":"plan_end:2026-04-21"}]}]}'
sequenzy segments update seg_123 --name "Churn Risk"
sequenzy segments update seg_123 --filters-json '[{"field":"tag","operator":"contains","value":"vip"}]'
sequenzy segments update seg_123 --join-operator or
sequenzy segments delete seg_123 --yes
```

Behavior:

- `segments list`: `GET /api/v1/segments`
- `segments count`: `GET /api/v1/segments/:id/count`
- `segments create`: `POST /api/v1/segments`
- `segments update`: `PATCH /api/v1/segments/:segmentId` with at least one of `--name`, `--filters-json`, `--filters-file`, or `--join-operator and|or`
- `segments delete`: `DELETE /api/v1/segments/:segmentId`; prompts for confirmation, `--yes` skips
- update filters replace the existing filter set; `--filters-json`/`--filters-file` accept the same array or `root` object shapes as create, and missing filter IDs are filled in by the CLI
- `--filter-json` accepts either the legacy raw segment filter array or a nested filter `root` object
- `--match all|any` controls whether top-level filters are combined with `and` or `or`
- MCP/API use `filterJoinOperator: "and" | "or"` for the same behavior
- nested segment logic uses `{ "kind": "group", "joinOperator": "and" | "or", "children": [...] }`
- custom event filters use `field: "event"` with values like `saas.purchase:30d`, `saas.purchase:all`, or `saas.purchase:5:30d`
- saved segment composition uses `field: "segment"` with `operator: "is" | "is_not"` and the referenced segment id as `value`
- Stripe product filters use `field: "stripeProduct"` and product IDs, not product names
- commerce purchase filters use `--commerce-product <id>` or `--commerce-collection <key>` with `--purchase-operator` and `--orders <number>` for order-count thresholds; `--payments` is the Stripe equivalent
- threshold operators encode the count as `productId:count`, for example `prod_pro:3`
- MCP parity: `update_segment` (reuses the create filter schemas) and `delete_segment`

## Products And Digital Delivery

```bash
sequenzy products list
sequenzy products list --provider stripe --search guide
sequenzy products list --all
sequenzy products sync
sequenzy products upsert my-course --title "Video Course" --price-cents 4900 --currency usd --file ./course.zip
sequenzy products upsert --products-file ./catalog.json
sequenzy products delete my-course
sequenzy products attach-file <product-id> --file ./guide.pdf
sequenzy products attach-file <product-id> --url https://example.com/template.zip --name template.zip
sequenzy products detach-file <product-id>
```

Behavior:

- `products list`: `GET /api/v1/products`, optionally with `?provider=stripe|shopify|woocommerce|manual&search=...`; returns one page by default (100-per-request cap), pass `--all` to page through larger catalogs
- `products sync`: `POST /api/v1/products/sync`; queues a catalog sync (`--integration` selects one when several are connected) and returns 404 without an active integration
- `products upsert [productId]`: creates or updates API-provider products keyed by your own `productId`; supports `--title`, `--description`, `--price-cents`, `--currency`, `--image-url`, `--product-url`, `--in-stock`/`--out-of-stock`, `--provider-created-at`, and bulk `--products-json`/`--products-file` for up to 100 products; combine `--file` or `--url` with a single-product upsert to attach the deliverable in the same command
- `products delete <productId>`: `DELETE /api/v1/products/:id`
- `products attach-file --file`: `POST /api/v1/products/delivery/upload-url` for a presigned URL, PUTs the file bytes there, then `PUT /api/v1/products/:id/delivery` with `source: "upload"`
- `products attach-file --url`: `PUT /api/v1/products/:id/delivery` with `source: "url"`
- `products detach-file`: `DELETE /api/v1/products/:id/delivery`
- MCP equivalents: `list_products`, `upsert_products`, `delete_product`, `attach_product_file` (URL attach only), `remove_product_file`, `sync_products`

Caveats:

- the `<product-id>` argument is the internal Sequenzy product ID from `products list`, not the Stripe `prod_...` ID; the Stripe ID is shown as the provider product ID in list output
- uploads accept PDF, ePub, ZIP, images, audio, video, and text files up to 100MB; HTML, SVG, and executables are rejected
- after attaching, purchases of the product enrich the `saas.purchase` event with `download.url` and `download.name`, so purchase sequences can deliver the file with `{{event.download.url}}` and `{{event.download.name}}`
- to start a purchase sequence only for one product, the trigger needs a `productIds equals <stripe product id>` property filter on the `saas.purchase` event; this is configured in the dashboard sequence editor ("Only for product" picker), not through current CLI/MCP flags
- products archived in Stripe stay listed with an archived flag, and attached files survive catalog re-syncs
- the upload endpoint returns 503 when file storage is not configured on the server; fall back to `--url` in that case

## Audience Syncs (Meta Ads)

```bash
sequenzy audience-syncs list
sequenzy audience-syncs ad-accounts
sequenzy audience-syncs create --template recent-buyers --ad-account act_123 --name "Sequenzy - Recent buyers"
sequenzy audience-syncs create --segment seg_123 --ad-account act_123 --name "Sequenzy - VIPs" --frequency hourly
sequenzy audience-syncs update sync_123 --frequency weekly --active false
sequenzy audience-syncs sync sync_123
sequenzy audience-syncs delete sync_123
```

Behavior:

- syncs a segment to a Meta custom audience; requires the Meta Ads integration connected in the dashboard (Settings -> Integrations)
- `create` requires `--ad-account` (from `audience-syncs ad-accounts`) and `--name`, plus either `--segment <id>` or `--template` (`zero-ltv`, `no-purchase-1y`, `recent-buyers`, `high-spenders-ecom`, `non-buyers`, `engaged` - the segment is created automatically on first use)
- `--frequency hourly|daily|weekly` defaults to daily; `sync` triggers an immediate run
- audiences are add-only: subscribers who leave the segment stay in the Meta audience
- MCP parity: `list_audience_syncs`, `list_ad_accounts`, `create_audience_sync`, `update_audience_sync`, `delete_audience_sync`, and `sync_audience_now`

## Templates

```bash
sequenzy templates list
sequenzy templates list --label edm
sequenzy templates get tmpl_123
sequenzy templates create welcome --subject "Welcome" --label edm --html-file ./welcome.html
sequenzy templates create welcome --subject "Welcome" --blocks-file ./welcome-blocks.json
sequenzy templates update tmpl_123 --subject "Updated" --label edm --html-file ./welcome-v2.html
sequenzy templates update tmpl_123 --blocks-file ./welcome-v2-blocks.json
sequenzy templates render tmpl_123 --out preview.html
sequenzy templates localizations set tmpl_123 es --blocks-file ./welcome-es.json
sequenzy templates localizations sync tmpl_123 --locale es fr
sequenzy templates delete tmpl_123
```

Behavior:

- `templates list`: `GET /api/v1/templates`, optionally with `?label=...`
- `templates get`: `GET /api/v1/templates/:id`
- `templates create`: `POST /api/v1/templates`
- `templates update`: `PUT /api/v1/templates/:id`
- `templates render`: `GET /api/v1/templates/:id/render`; same personalization and output flags as `campaigns render` (see Campaigns)
- `templates localizations set <id> <locale>` stores caller-supplied localized content immediately; the locale must be enabled in the company's localization settings and cannot be the primary locale; provide exactly one HTML or blocks content source
- `templates localizations sync <id>` queues AI translation for selected `--locale` values, or every enabled non-primary locale when omitted
- `templates delete`: `DELETE /api/v1/templates/:id`
- MCP parity: `set_template_localization` and `sync_template_localizations`

Caveats:

- list accepts `--label <labels...>` to filter by template label name
- create requires `name`, `subject`, and either `html` or `blocks`; it can also assign labels with `--label <labels...>`
- update accepts `name`, `subject`, `html`, `blocks`, and replacement labels with `--label <labels...>`
- `--blocks-json` and `--blocks-file` pass Sequenzy block arrays through directly
- conditional email content is only available through block JSON, using a block-level `condition` object
- raw HTML is still stored as a single text block by the current API path
- deletion can fail if the template is still referenced by a campaign or sequence

## Campaigns

```bash
sequenzy campaigns list
sequenzy campaigns list --status draft --label edm --company comp_123
sequenzy campaigns get camp_123
sequenzy campaigns create "April Launch" --prompt "Announce our new dashboard"
sequenzy campaigns create "April Launch" --subject "We shipped" --label edm --html-file ./campaign.html
sequenzy campaigns create "April Launch" --subject "We shipped" --blocks-file ./campaign-blocks.json
sequenzy campaigns create "Partner Update" --subject "We shipped" --html-file ./campaign.html --segment seg_123
sequenzy campaigns create "Partner Update" --subject "We shipped" --html-file ./campaign.html --target-lists-json '{"type":"lists","listIds":["list_123"]}'
sequenzy campaigns update camp_123 --subject "Updated subject" --label edm
sequenzy campaigns update camp_123 --blocks-file ./campaign-v2-blocks.json
sequenzy campaigns update camp_123 --reply-to support@example.com
sequenzy campaigns update camp_123 --reply-profile reply_123
sequenzy campaigns audience camp_123
sequenzy campaigns render camp_123 --out preview.html
sequenzy campaigns render camp_123 --subscriber sub_123 --out -
sequenzy campaigns schedule camp_123 --at "2026-06-01T14:00:00Z"
sequenzy campaigns schedule camp_123 --at "2026-06-01T14:00:00Z" --segment seg_123
sequenzy campaigns schedule camp_123 --at "2026-06-01T14:00:00Z" --target-lists-json '{"type":"all"}'
sequenzy campaigns schedule camp_123 --at "2026-06-01T14:00:00Z" --repeat monthly
sequenzy campaigns test camp_123 --to you@example.com
sequenzy campaigns cancel camp_123
sequenzy campaigns unschedule camp_123
sequenzy campaigns pause camp_123
sequenzy campaigns resume camp_123 --spread-over-hours 6
sequenzy campaigns delete camp_123 --yes
sequenzy campaigns duplicate camp_123 --mode ab_test
sequenzy campaigns duplicate camp_123 --mode variant --variant-id var_b
sequenzy campaigns resend-to-non-openers camp_123
```

Behavior:

- `campaigns list`: `GET /api/v1/campaigns`, optionally with `?status=...` and `?label=...`; results are paginated (default page size 50, `--limit` capped at 100), page with `--offset`
- `campaigns get`: `GET /api/v1/campaigns/:id`
- `campaigns audience`: `GET /api/v1/campaigns/:id/audience`; resolves list and segment names, recomputes the recipient count at read time, and warns when targeting is unset (scheduling then sends to every active subscriber)
- `campaigns render`: `GET /api/v1/campaigns/:id/render`; renders the email exactly as it would be sent. Personalize with `--subscriber <id>` or `--email <addr>` (plus `--first-name`, `--attr`, `--attrs-json`, `--var`, `--vars-json`), force a locale with `--locale`, apply auto-UTM with `--tracking`, pick an A/B variant with `--variant`, and write with `--out <file|->`. Unresolved merge tags are listed in the summary; per-send click redirects and the open pixel are added at send time, so they never appear in a render
- `campaigns create`: `POST /api/v1/campaigns`
- `campaigns update`: `PUT /api/v1/campaigns/:id`
- `campaigns schedule`: `POST /api/v1/campaigns/:id/schedule`
- `campaigns test`: `POST /api/v1/campaigns/:id/test`
- `campaigns cancel`: `POST /api/v1/campaigns/:id/cancel`
- `campaigns unschedule`: returns a scheduled campaign (and any recurrence) to an editable draft; only scheduled campaigns can be unscheduled
- `campaigns pause`: `POST /api/v1/campaigns/:id/pause`
- `campaigns resume`: `POST /api/v1/campaigns/:id/resume`
- `campaigns delete`: `DELETE /api/v1/campaigns/:id`
- `campaigns duplicate`: `POST /api/v1/campaigns/:id/duplicate`
- `campaigns resend-to-non-openers`: `POST /api/v1/campaigns/:id/resend-to-non-openers`; available 6 hours after a sent campaign finishes, creates a draft targeting the same audience plus a "didn't open this campaign" rule, and estimates the non-opener count - review the draft, then schedule it
- dashboard-aware responses include `url`, campaign review `previewUrl`, and `appUrls` when the company can be resolved

Caveats:

- list accepts `--status` and `--label <labels...>` filters
- create supports `name`, optional `subject` when `--prompt` is used (the generated subject is used unless `--subject` is also provided), `html`, `blocks`, `--prompt`, `--style`, `--tone`, and labels with `--label <labels...>`
- create and update also take `--preview-text` / `--preheader-text` for the inbox preview line and `--tracking-code` for a campaign UTM code
- create `--status sent` with `--sent-at` archives an already-sent campaign (for migrations); it does not send email or create per-recipient delivery history
- create can save the audience on the draft with `--segment <id>`, `--target-lists-json`, or `--target-lists-file`; `--segment` and the target list flags are mutually exclusive, and omitting all of them leaves targeting unset
- `campaigns create` and `campaigns get` print the saved audience on the `Audience` line and return it as `targetLists` with `--json`
- target list examples: `{"type":"all"}`, `{"type":"lists","listIds":["list_123"]}`, `{"type":"segment","segmentId":"seg_123"}`
- update supports `name`, `subject`, `html`, `blocks`, replacement labels with `--label <labels...>`, `--reply-to`, and `--reply-profile`
- update can replace the saved audience with `--segment` or `--target-lists-json`/`--target-lists-file`, or drop it with `--clear-target-lists` to choose at schedule time
- update takes `--campaign-data-json/-file` and `--computed-lists-json/-file` to upload personalization data for repeat blocks
- schedule requires `--at <datetime>` with a future ISO timestamp and a verified sending domain
- schedule can pass targeting with `--segment`, `--target-lists-json`, or `--target-lists-file`; omit them to reuse saved targeting or default to all active subscribers
- schedule `--repeat weekly|monthly` makes the campaign a recurring template: each run is duplicated and sent automatically, and audience membership is re-evaluated at every run; use `campaigns unschedule` to stop the series, or re-schedule without `--repeat` for a one-shot send
- `--spread-over-hours` accepts integers from 1 to 72 and takes precedence over send-time optimization (`--send-time-optimization`)
- `--prompt` generates draft campaign content through `POST /api/v1/generate/email`; do not combine it with HTML or block flags
- `--blocks-json` and `--blocks-file` pass Sequenzy block arrays through directly
- conditional email content is only available through block JSON, using block-level `condition` rules
- `--reply-to` resolves an existing reply profile by email and `--reply-profile` sets it directly by ID
- `--reply-to` and `--reply-profile` are mutually exclusive
- `campaigns get` now includes saved reply-to details when the campaign has a reply profile
- only draft campaigns can be updated through this API path
- there is no CLI command for immediate send; schedule with a near-future `--at` timestamp instead
- `cancel` works from scheduled, sending, paused, waiting_approval, and rejected statuses; it shows no confirmation prompt so a bad send can be stopped fast
- `pause` only works on a campaign in sending status; `resume` only works on a paused campaign
- `resume --spread-over-hours` accepts integers from 1 to 72 to spread the remaining delivery
- `delete` is blocked while the campaign is sending, scheduled, or paused; cancel it first
- `duplicate --mode campaign` copies the campaign email, `--mode ab_test` also copies the A/B test with all variants, and `--mode variant` (requires `--variant-id`) copies one variant's content as a plain campaign; the copy is always a new draft
- in the current backend checkout, `campaigns test` returns a success message path rather than a confirmed email send

MCP parity:

- `list_templates` and `list_campaigns` accept `label`
- `create_template`, `update_template`, `create_campaign`, and `update_campaign` accept `labels`
- `create_campaign` accepts `targetLists` (or the `segmentId` shorthand, mutually exclusive) to save the audience on the draft; `get_campaign` returns it as `targetLists`, `null` when targeting is still unset
- `update_campaign` accepts `name`, `subject`, `html`, `blocks`, `labels`, `replyTo`, and `replyProfileId`
- `schedule_campaign` accepts `campaignId`, `scheduledAt`, optional `targetLists`, `sendTimeOptimization`, and `spreadOverHours`
- `cancel_campaign`, `unschedule_campaign`, `pause_campaign`, `resume_campaign` (optional `spreadOverHours`), `delete_campaign`, and `duplicate_campaign` (optional `mode` and `variantId`) mirror the lifecycle commands
- `get_campaign_audience`, `render_email`, and `resend_campaign_to_non_openers` mirror `audience`, `render`, and `resend-to-non-openers`
- `replyTo` and `replyProfileId` are mutually exclusive
- MCP rejects calls that omit all update fields before hitting the API
- MCP rejects unsupported extra update fields before hitting the API

## Forms

```bash
sequenzy forms list --json
sequenzy forms update form_123 --headline "Join the newsletter" --button-text "Subscribe"
sequenzy forms update form_123 --list list_123 --tag tag_456 --duplicate-strategy merge
sequenzy forms update form_123 --blocks-file ./form-blocks.json
```

Behavior:

- `forms list`: `GET /api/v1/forms`; `forms update`: `PATCH /api/v1/forms/:id`
- updates are partial: only the options you pass change; other flags cover `--name`, `--success-message`, `--redirect-url`, `--theme-json/-file`, and `--clear-tags` to empty the tag targeting
- run `forms list --json` before replacing blocks and send the complete current block array; the form must retain exactly one email field and one submit button
- MCP parity: `list_forms` and `update_form`; MCP additionally has `create_form` and `get_form_embed` (embed snippet), which have no CLI command yet

## Landing Pages

```bash
sequenzy landing-pages list
sequenzy landing-pages get lp_123
sequenzy landing-pages create "Spring promo" --template default
sequenzy landing-pages create "Spring promo" --content-file ./page.json
sequenzy landing-pages update lp_123 --name "Spring promo v2" --content-file ./page-v2.json
sequenzy landing-pages publish lp_123
sequenzy landing-pages unpublish lp_123
sequenzy landing-pages duplicate lp_123 --name "Autumn promo"
sequenzy landing-pages delete lp_123
sequenzy landing-pages connect-domain pages.example.com
sequenzy landing-pages update-domain-settings --domain pages.example.com --verify
```

Behavior:

- CRUD maps to `GET/POST/PATCH/DELETE /api/v1/landing-pages[/:id]`, plus `/publish`, `/unpublish`, and `/duplicate` actions and `/api/v1/landing-pages/domain` for custom domains
- pages are created as drafts; publish only after reviewing. When `--content-json/-file` is omitted, a valid default page is generated from `--template`
- a duplicate is always a draft with its own slug, views, and conversions; without `--name` it is called "<original name> (copy)"
- custom domains require a CNAME record pointing to `pages.sequenzydns.com`
- MCP parity: `list_landing_pages`, `get_landing_page`, `create_landing_page`, `update_landing_page`, `publish_landing_page`, `unpublish_landing_page`, `duplicate_landing_page`, `delete_landing_page`, `connect_landing_page_domain`, and `update_landing_page_domain_settings`

## Sequences

```bash
sequenzy sequences list
sequenzy sequences get seq_123
sequenzy sequences create onboarding --trigger event_received --event-name signup.completed --goal "Guide new users to activation" --email-count 4
sequenzy sequences create onboarding --trigger contact_added --list-id list_123 --steps-file ./steps.json
sequenzy sequences create winback --trigger tag_added --tag-name cancelled --steps-file ./discount-steps.json
sequenzy sequences update seq_123 --steps-file ./sequence-updates.json
sequenzy sequences update seq_123 --branch-file ./branch.json
sequenzy sequences render seq_123 --out preview.html
sequenzy sequences test seq_123 --node-id node_email_2 --to you@example.com
sequenzy sequences enable seq_123
sequenzy sequences disable seq_123
sequenzy sequences duplicate seq_123 --name "Onboarding v2"
sequenzy sequences archive seq_123
sequenzy sequences unarchive seq_123
sequenzy sequences enrollments seq_123 --status active waiting
sequenzy sequences enrollments seq_123 --all --csv enrollments.csv
sequenzy sequences pause-enrollments seq_123
sequenzy sequences resume-enrollments seq_123
sequenzy sequences delete seq_123
sequenzy sequences enroll seq_123 --email one@example.com two@example.com
sequenzy sequences enroll seq_123 --emails-file ./vips.csv
sequenzy sequences enroll seq_123 --email one@example.com --target-node-id node_email_2
sequenzy sequences cancel-enrollments seq_123 --subscriber-id sub_123 --reason "Converted"
sequenzy sequences cancel-enrollments seq_123 --field-path order.id --field-values ord_123,ord_456
sequenzy sequences cancel-enrollments seq_123 --field-values price_123 --apply
```

Behavior:

- `sequences list`: `GET /api/v1/sequences` with `--status`, `--search`, `--label`, `--limit`, and `--offset` filters
- `sequences get`: `GET /api/v1/sequences/:id`
- `sequences create`: `POST /api/v1/sequences`
- `sequences update`: `PUT /api/v1/sequences/:id`
- `sequences render`: `GET /api/v1/sequences/:sequenceId/nodes/:nodeId/render`; same personalization and output flags as `campaigns render`, plus `--variant` for A/B nodes
- `sequences test --node-id <id> --to <email>` sends only the selected saved email step; it does not enable the sequence or enroll subscribers - use `sequences get` first to find the email step nodeId (MCP: `send_sequence_test_email`)
- `sequences enable`: `POST /api/v1/sequences/:id/enable`
- `sequences disable`: `POST /api/v1/sequences/:id/disable`
- `sequences duplicate`: `POST /api/v1/sequences/:id/duplicate`; email templates, graph topology, and sequence A/B tests are copied independently (MCP: `duplicate_sequence`)
- `sequences archive` / `unarchive` hide or restore a sequence without deleting it (MCP: `archive_sequence`, `unarchive_sequence`)
- `sequences enrollments`: `GET /api/v1/sequences/:id/enrollments`; defaults to active and waiting enrollments, filters by `--status`, `--node-id`, `--subscriber-id`, and `--email`, and exports with `--all --csv <file>`. `waitUntil` is when a waiting enrollment resumes. Get node IDs from `sequences get` or `stats --sequence` (MCP: `list_sequence_enrollments`)
- `sequences pause-enrollments` stops new entrants while existing active and waiting recipients keep moving (the sequence must be active; use `disable` to freeze current recipients too); `resume-enrollments` reopens entry (MCP: `pause_sequence_enrollments`, `resume_sequence_enrollments`)
- `sequences delete`: `DELETE /api/v1/sequences/:id`
- `sequences enroll`: `POST /api/v1/sequences/:id/enroll`
- `sequences cancel-enrollments`: `POST /api/v1/sequences/:id/enrollments/cancel`
- dashboard-aware responses include `url` on sequence records and `appUrls` on the top-level JSON when the company can be resolved

Goals and inbound webhooks:

- `sequences goals list|create|update|delete` manage conversion goals on `/api/v1/sequences/:id/goals[/:goalId]`; `create <sequenceId> <name>` and `update` accept `--active`/`--inactive` (MCP: `list_sequence_goals`, `create_sequence_goal`, `update_sequence_goal`, `delete_sequence_goal`)
- `sequences webhook get|configure|rotate <sequenceId>` manage the sequence's inbound trigger webhook; `configure` accepts `--field-mapping-json/-file` and `--sample-payload-json/-file` (with `--clear-*` variants), and `rotate` replaces the signing secret (MCP: `get_sequence_inbound_webhook`, `configure_sequence_inbound_webhook`, `rotate_sequence_inbound_webhook_secret`)

Caveats:

- CLI sequence creation supports either AI `--goal` mode or explicit `--steps-json` / `--steps-file` mode
- explicit create steps can include `{ "type": "create_discount" }`; emails after that action can reference `{{discount.code}}`, `{{discount.percentOff}}`, and related `discount.*` merge tags
- discount action sequences require a connected Stripe integration before activation
- `--email-count` is only meaningful with `--goal`
- `--email-count` accepts 1 to 10 generated emails
- trigger-specific options depend on `--trigger`
- updates accept either step payloads or email payloads via `--steps-*` or `--emails-*`, not both; entries can target a step by `nodeId`, `emailId`, or array order
- `--insert-steps-json/-file` adds new linear steps (including outbound webhook and SMS steps); `--node-updates-json/-file` and `--graph-edit-json/-file` edit and rewire existing nodes (MCP: `insert_sequence_step`, `update_sequence_node`, `update_sequence_nodes`, `edit_sequence_graph`)
- SMS steps use `{"type":"sms","text":"..."}` (optional `label`, `ineligibleAction skip|exit`, `imageUrls`); edit existing ones with `--sms-steps-json/-file`. Update Subscriber steps use `type="update_subscriber"` with attribute updates like `{{event.plan}}`; edit with `--subscriber-update-steps-json/-file`
- inserting steps or branches into an active sequence requires `--confirm-structural-change`
- `--stop-condition-json/-file` sets the sequence stop condition (e.g. `{"type":"has_tag","value":"customer"}`; `{"type":"none","value":null}` clears it). Stop conditions are re-evaluated before every step including the first, so `does_not_have_tag` / `removed_from_list` act as an allowlist: guarded-out contacts still enroll and are cancelled at the trigger node. Clearing the guard does not retry contacts it already cancelled - re-enroll them with `sequences enroll`, which skips the `one_time` check
- create and update share trigger flags (`--trigger`, `--list-id`, `--tag-name`, `--segment-id`, `--event-name`, `--integration-slug`/`--integration-event-key`, `--inactive-days`, `--min-count`/`--time-window-days`) plus `--property-filters-json/-file` to only start when event properties match (e.g. scope a purchase sequence to one product)
- `--enrollment-mode matching_field` with `--enrollment-field-path` (e.g. `order.id`, `product.providerVariantId`) enables product- or variant-specific event sequences; Shopify replenishment and back-in-stock sequences default to it
- sender identity is set per sequence with `--from-email`/`--from-name`/`--sender-profile-id` and `--reply-to`/`--reply-to-name`/`--reply-profile-id`; `--bcc-emails` blind-copies team inboxes on every sequence email (`--clear-bcc-emails` removes)
- `--sending-window-start/-end/-timezone` hold every email step until the allowed local send window; add `--sending-window-days weekdays` for working days only and `--clear-sending-window` to remove it
- `--pause-new-enrollments` / `--resume-new-enrollments` toggle entry from within `update`; `--label`/`--clear-labels` and `--user-cancellable`/`--not-user-cancellable` round out the metadata flags
- branch insertion uses `--branch-json` or `--branch-file` with condition types `has_tag`, `does_not_have_tag`, `in_list`, `in_segment`, `event_received`, `link_clicked`, and `field_*`; branch paths may set `targetNodeId`/`elseTargetNodeId` to wire directly to existing nodes, including the completion node
- branch condition fields are `tagId`/`tagName`, `listId`, `segmentId`/`segmentName`, `eventName`, `linkUrl`, `activityScope`, or `fieldName`/`fieldValue`; omit `linkUrl` to match any clicked link
- for `event_received` and `link_clicked`, set `activityScope` to `this_sequence`, `previous_email`, or `ever`; omitting it checks the contact's full history
- `enroll` takes exactly one email source: repeated `--email`, `--emails-json`, or `--emails-file`, with the same file formats as `lists add-subscribers` and the same 500-email batching
- `enroll` only enrolls active subscribers; unknown emails are reported as `notFound`, and inactive or already-enrolled subscribers count as `skipped`
- `enroll` requires the sequence to be accepting entrants (enabled and not paused for enrollment)
- `enroll` starts subscribers at the first step after the trigger unless `--target-node-id` points at a specific non-trigger node; the result reports `enrolled`, `skipped`, `notFound`, `targetNodeId`, and `scheduledFor`
- MCP uses `enroll_subscribers_in_sequence` with `emails` (max 500 per call) and optional `targetNodeId`
- `cancel-enrollments` requires a sequence ID and exactly one target: `--all`, `--subscriber-id`, `--subscriber-ids`, or `--field-values`; use `--all` to fully stop a live sequence when enrollments came from a segment and share no entry field value
- `--field-values` matches active/waiting enrollments by the stored entry event property at `--field-path`, or the sequence's configured `enrollmentFieldPath` when `--field-path` is omitted
- CLI cancellation is a dry run unless `--apply` is passed; use dry runs for field-value/bulk checks before mutating enrollments
- MCP uses `cancel_sequence_enrollments` with the same target rule; set `dryRun: false` to apply field-value cancellation

## A/B Tests

```bash
sequenzy ab-tests list
sequenzy ab-tests list --sequence seq_123
sequenzy ab-tests get ab_123
sequenzy ab-tests stats ab_123 --period 7d
sequenzy ab-tests stats ab_123 --start "2026-05-01T00:00:00Z" --end "2026-05-31T00:00:00Z"
sequenzy ab-tests restart ab_123 --source-variant var_b --test-type content --variant-count 3
sequenzy ab-tests update ab_123 --test-percentage 30 --winner-criteria click_rate
sequenzy ab-tests update-variant ab_123 var_b --subject "New subject"
sequenzy ab-tests update-variant ab_123 var_b --blocks-file ./variant-b.json
sequenzy ab-tests create camp_123 --test-percentage 30 --duration-minutes 120 --winner-criteria click_rate
sequenzy ab-tests create camp_123 --variants-json '[{"subject":"Alternative subject"}]'
sequenzy ab-tests add-variant ab_123 --subject "Alternative subject" --blocks-file ./variant.json
sequenzy ab-tests delete-variant ab_123 var_b --yes
sequenzy ab-tests delete ab_123 --yes
```

Behavior:

- `ab-tests list`: `GET /api/v1/ab-tests`, optionally with `?sequenceId=...` via `--sequence`
- `ab-tests get`: `GET /api/v1/ab-tests/:id`
- `ab-tests update`: `PATCH /api/v1/ab-tests/:id` with `--name`, `--test-percentage`, `--duration-minutes`, `--winner-criteria`, `--test-type`, and `--winner-threshold`; campaign tests use percentage/duration, sequence tests use `--test-type` and `--winner-threshold`; add `--confirm-live-change` when the settings affect an active test or a test with recorded activity (MCP: `update_ab_test`)
- `ab-tests stats`: `GET /api/v1/ab-tests/:id/stats`; bot opens/clicks are excluded by default, pass `--include-bots` for raw engagement
- `ab-tests restart`: `POST /api/v1/ab-tests/:id/restart`
- `ab-tests update-variant`: `PATCH /api/v1/ab-tests/:id/variants/:variantId`
- `ab-tests create`: `POST /api/v1/ab-tests`
- `ab-tests add-variant`: `POST /api/v1/ab-tests/:id/variants`
- `ab-tests delete-variant`: `DELETE /api/v1/ab-tests/:id/variants/:variantId`
- `ab-tests delete`: `DELETE /api/v1/ab-tests/:id`

Caveats:

- run `ab-tests get` first to discover variant IDs before targeting a variant
- `stats` uses `--period` (`1h`, `24h`, `7d`, `30d`, `90d`) or both `--start` and `--end`; custom ranges max at 90 days
- `restart` only applies to sequence A/B tests with a selected winner; options are `--source-variant`, `--test-type subject|content`, `--winner-threshold` (10-1000), and `--variant-count` (2-4 including control)
- `update-variant` accepts `--subject`, `--preview-text`, and either HTML or blocks flags, not both; only draft A/B tests can be edited
- `create` targets a campaign: the campaign must be in draft or rejected status and must not already have an A/B test
- `create` builds variant A automatically as the control from the campaign's email; extra variants from `--variants-json`/`--variants-file` use `{subject, previewText?, blocks?}` objects
- `create` accepts `--name`, `--test-percentage` (5-50, default 20), `--duration-minutes` (15-1440, default 240), and `--winner-criteria open_rate|click_rate` (default open_rate)
- `add-variant` requires `--subject` and only works while the test is in draft status; when a sequence test's parent sequence is active, add `--confirm-live-change`
- `delete-variant` cannot remove variant A (the protected control) and must leave at least 2 variants; a test holds at most 5 variants; when a sequence test's parent sequence is active, add `--confirm-live-change`
- `delete` is blocked for running tests, and the linked campaign must be draft or rejected
- `create`, `add-variant`, and `delete-variant` support campaign and sequence A/B tests (`create --automation-node` converts a sequence email node and needs `--confirm-live-change` on active sequences); `delete` supports campaign A/B tests only
- MCP parity: `list_ab_tests`, `get_ab_test`, `get_ab_test_stats`, `restart_ab_test`, `update_ab_test_variant`, `create_ab_test`, `add_ab_test_variant`, `delete_ab_test_variant`, and `delete_ab_test`

## AI Generation

```bash
sequenzy generate email "Welcome a new user to our analytics product"
sequenzy generate email "Product launch announcement" --style branded --tone friendly
sequenzy generate sequence "Onboard a new workspace admin" --count 4 --days 14
sequenzy generate subjects "April product launch" --count 8
sequenzy generate sms "Flash sale reminder" --count 3
```

Behavior:

- `generate email`: `POST /api/v1/generate/email`
- `generate sequence`: `POST /api/v1/generate/sequence`
- `generate subjects`: `POST /api/v1/generate/subjects`
- `generate sms`: `POST /api/v1/generate/sms`; drafts SMS copy for `type: "sms"` sequence steps and test sends
- `--json` returns the raw API response for agent/tool parsing

Caveats:

- generated content is draft content and should be reviewed before sending
- `generate sequence --count` accepts 1 to 10 emails
- `generate email` supports optional `--style` and `--tone`
- MCP parity: `generate_email`, `generate_sequence`, `generate_subject_lines`, and `generate_sms`

## Team

```bash
sequenzy team list
sequenzy team invite teammate@example.com --role admin
sequenzy team invite finance@example.com --role viewer --billing-access
sequenzy team cancel-invitation inv_123 --yes
```

Behavior:

- `team list`: `GET /api/v1/team`; returns the owner, members, and pending or expired invitations
- `team invite`: `POST /api/v1/team/invitations` with `{ email, role, canManageBilling? }`
- `team cancel-invitation`: `DELETE /api/v1/team/invitations/:invitationId`

Caveats:

- `--role` must be `admin` or `viewer`
- inviting and cancelling invitations requires owner or admin access; `--billing-access` can only be granted by the company owner
- existing Sequenzy users are added to the team immediately; new emails receive an invitation that expires after 14 days
- run `team list` first to find invitation IDs before cancelling
- `cancel-invitation` prompts for confirmation; pass `--yes` to skip
- MCP parity: `list_team_members`, `invite_team_member`, and `cancel_team_invitation`

## Inbox

```bash
sequenzy inbox list --status open --unread
sequenzy inbox list --search "refund" --page 2 --limit 50
sequenzy inbox get conv_123
sequenzy inbox reply conv_123 --text "Thanks for reaching out!"
sequenzy inbox reply conv_123 --html-file ./reply.html --subject "Re: your question"
sequenzy inbox reply conv_123 --text "Customer asked for a refund" --note
sequenzy inbox close conv_123
sequenzy inbox reopen conv_123
sequenzy inbox mark-read conv_123
```

Behavior:

- `inbox list`: `GET /api/v1/conversations` with optional `status` (`open`, `closed`, `all`), `search`, `unread`, `page`, and `limit` (1-100, default 20) query parameters
- `inbox get`: `GET /api/v1/conversations/:conversationId`; returns the conversation with its full message history
- `inbox reply`: `POST /api/v1/conversations/:conversationId/messages`
- `inbox close` and `inbox reopen`: `POST /api/v1/conversations/:conversationId/status` with `{ status: "closed" | "open" }`
- `inbox mark-read`: `POST /api/v1/conversations/:conversationId/read`; reports how many messages were marked read

Caveats:

- the inbox collects subscriber replies to campaigns, sequences, and transactional email
- `reply` requires a body from `--text` and/or `--html-file`
- `--note` adds an internal note instead of emailing the subscriber
- outbound replies are sent asynchronously by a worker, so the message starts in `pending` delivery status; they also reopen closed conversations
- MCP parity: `list_conversations`, `get_conversation`, `reply_to_conversation`, `update_conversation_status`, and `mark_conversation_read`

## Webhooks

```bash
sequenzy webhooks list
sequenzy webhooks create --name CI --url https://example.com/hook --event email.bounced subscriber.unsubscribed
sequenzy webhooks create --name "All events" --url https://example.com/hook
sequenzy webhooks update wh_123 --url https://example.com/new-hook
sequenzy webhooks update wh_123 --event email.bounced email.complained
sequenzy webhooks update wh_123 --disable
sequenzy webhooks delete wh_123 --yes
sequenzy webhooks test wh_123
sequenzy webhooks deliveries wh_123 --limit 50
sequenzy webhooks replay wh_123 del_456
```

Behavior:

- `webhooks list`: `GET /api/v1/webhooks`
- `webhooks create`: `POST /api/v1/webhooks` with `{ name, url, events? }`
- `webhooks update`: `PATCH /api/v1/webhooks/:id` with at least one of `--name`, `--url`, `--event`, `--enable`, or `--disable`
- `webhooks delete`: `DELETE /api/v1/webhooks/:id`; this permanently deletes the endpoint and its delivery history
- `webhooks test`: `POST /api/v1/webhooks/:id/test`
- `webhooks deliveries`: `GET /api/v1/webhooks/:id/deliveries`, optionally with `?limit=` (1-100)
- `webhooks replay`: `POST /api/v1/webhooks/:id/deliveries/:deliveryId/replay`

Caveats:

- valid event types are `email.sent`, `email.delivered`, `email.delivery_delayed`, `email.bounced`, `email.complained`, `email.opened`, `email.clicked`, `email.unsubscribed`, `subscriber.invalid`, `subscriber.updated`, `subscriber.unsubscribed`, `sequence.finished`, and `sequence.failed`; omit `--event` to use the default set
- `create` prints the signing secret exactly once and it cannot be retrieved later; surface it to the user immediately
- `--enable` and `--disable` are mutually exclusive; changing the URL or re-enabling resets the failure circuit breaker
- the webhook must be enabled to receive a test event
- run `webhooks deliveries <id>` first to find delivery IDs before replaying
- `delete` prompts for confirmation; pass `--yes` to skip
- MCP parity: `list_webhooks`, `create_webhook`, `update_webhook`, `delete_webhook`, `test_webhook`, `list_webhook_deliveries`, and `replay_webhook_delivery`

## API Keys

```bash
sequenzy api-keys create
sequenzy api-keys create --name "CI deploy key" --company comp_123
sequenzy api-keys create --preset transactional_sender --name "Order emails"
sequenzy api-keys create --scopes "subscribers:read,subscribers:write,events:write"
sequenzy api-keys list
sequenzy api-keys revoke key_123 --yes
```

Behavior:

- `create` sends `POST /api/v1/api-keys`; keys default to full access, restrict with `--preset` (`full_access`, `read_only`, `agent_safe`, `ai_drafting`, `data_ingest_safe`, `data_ingest_automations`, `transactional_sender`, `marketing_sender`) or explicit comma-separated `--scopes` (overrides `--preset`)
- `list` returns metadata only (ID, name, prefix, current-key marker); secrets and stored hashes are never returned
- `revoke` cannot be undone; run `api-keys list` first and verify the exact ID and prefix (`api-keys delete` is a compatibility alias for revoke)
- MCP parity: `create_api_key`, `list_api_keys`, `revoke_api_key`, and `delete_api_key`

Caveat:

- the plain API key is returned only at creation time; save it immediately

## Integrations

```bash
sequenzy integrations catalog --provider polar
sequenzy integrations connect polar --api-key $POLAR_KEY --webhook-secret $POLAR_WH_SECRET
sequenzy integrations list
sequenzy integrations get int_123
sequenzy integrations activity --status failed
sequenzy integrations sync int_123
sequenzy integrations enable-sync int_123
sequenzy integrations disable-sync int_123
sequenzy integrations pixel int_123
sequenzy integrations enable-pixel int_123
sequenzy integrations guide --framework nextjs
```

Behavior:

- `catalog`: `GET /api/v1/integrations/catalog`; works whether or not the provider is connected - its `connectFields` list what a provider needs, and its events are the ones sequences can trigger on
- `connect`: `POST /api/v1/integrations/connect`; covers API-key / webhook-secret providers (Polar, Paddle, Dodo, Whop, Creem, Chargebee, Clerk, PostHog, Affonso). OAuth and app-install providers (Stripe, Shopify, Supabase, GitHub, WooCommerce) connect and disconnect in the dashboard only. Reconnecting replaces stored credentials, and the response includes the webhook URL to configure at the provider
- pass secrets via environment variables instead of flags to keep them out of shell history; credentials, access tokens, and webhook secrets are never returned by read commands
- `list` / `get`: connected integrations and one integration's detail - `get` lists every event the provider emits, the tags each applies through sync rules, and which sequences trigger on it
- `activity`: `GET /api/v1/integrations/activity`; retained for 24 hours - use `--status failed` when an integration reports connected but contacts are not appearing
- `sync <id>` queues a catalog/backfill re-sync (Stripe, Polar, Paddle, Dodo, Creem, Chargebee, Whop); returns immediately, poll `integrations get` to watch `syncStatus`; fails with a conflict if a sync is already running
- `enable-sync` / `disable-sync` toggle bulk imports and backfills while keeping the connection, credentials, and live webhook delivery active
- `pixel <id>` (Shopify only) reads live pixel status from the store - check it before trusting product views, cart activity, or browse abandonment, because without the pixel those events never arrive and any sequence they trigger silently never fires; `enable-pixel` is idempotent, starts on the next storefront visit, and backfills nothing
- `guide` prints the same integration code examples the MCP `get_integration_guide` tool returns
- MCP parity: `connect_integration`, `list_integrations`, `list_integration_capabilities`, `get_integration`, `list_integration_activity`, `sync_integration`, `set_integration_sync_enabled`, `get_integration_pixel`, `activate_integration_pixel`, and `get_integration_guide`

## Sender Profiles And Tracking

```bash
sequenzy sender-profiles list
sequenzy sender-profiles update sender_abc123 --name "SnapCount"
sequenzy sender-profiles update reply_abc123 --name "SnapCount" --reply
sequenzy tracking settings
sequenzy tracking update --clicks on --strict-bot-filtering on
sequenzy tracking update --auto-utm on --utm-source sequenzy --utm-campaign "{{email.subject}}"
sequenzy tracking update --double-opt-in on
```

Behavior:

- `sender-profiles list`: `GET /api/v1/sender-profiles`; shows which profiles are account defaults and whether each sender address sits on a verified sending domain
- `sender-profiles update` renames one profile in place (`--reply` treats the ID as a reply-to profile) and never changes the account defaults - change defaults with `sequenzy companies update --sender-profile-id`; colliding renames are rejected with the ID of the profile already using that name; requires the `companies:manage` scope
- `tracking settings` / `tracking update`: `GET`/`PATCH /api/v1/tracking-settings`, covering open/click/unsubscribe tracking, `--strict-bot-filtering`, `--attribution-window-hours`, `--double-opt-in`, and auto-UTM (`--auto-utm` with `--utm-source/medium/campaign/content/term` templates, `--reset-utm`)
- tracking updates are partial (omitted flags keep current values) and apply to emails sent afterwards; UTM templates support placeholders such as `{{email.subject}}` and `{{link.text}}`
- `--double-opt-in on` makes new contacts start pending until they confirm; it needs a sender profile and provisions the confirmation email automatically; existing active contacts are unaffected
- reply tracking is set with `sequenzy companies update`; the dedicated click-tracking domain is configured in the dashboard
- MCP parity: `list_sender_profiles`, `update_sender_profile`, `get_tracking_settings`, and `update_tracking_settings`

## Shopify Automation Settings

```bash
sequenzy shopify settings get
sequenzy shopify settings update --cart-abandonment on --cart-delay-hours 4
sequenzy shopify settings update --browse-abandonment on --price-drop on --price-drop-min-percent 10
```

Behavior:

- `GET`/`PATCH /api/v1/shopify/automation-settings`; requires a connected Shopify integration, and `get` shows effective values (platform defaults apply until the store overrides them)
- browse abandonment emails shoppers who viewed a product but didn't buy; cart abandonment fires `ecommerce.cart_abandoned` with the full cart after inactivity; price drop alerts recent viewers when a product's price falls
- each family has delay/cooldown/threshold flags plus a `--reset-*` flag to return to defaults; updates are partial
- if no sequence handles `ecommerce.price_drop`, Sequenzy sends a default price-drop email to active subscribers
- MCP parity: `get_shopify_automation_settings` and `update_shopify_automation_settings`

## SMS

```bash
sequenzy sms settings
sequenzy sms send-test "+15550100" --text "Test from Sequenzy"
sequenzy generate sms "Flash sale reminder"
```

Behavior:

- `sms settings`: `GET /api/v1/sms/settings`; `readyToSend` in the JSON output tells you whether SMS sequence steps will actually send (add-on enabled, eligible plan, active number)
- `sms send-test`: `POST /api/v1/sms/test`; sends a real text and charges credits, limited to 5 test sends per hour, bypasses quiet hours, and is excluded from step stats
- add SMS steps to sequences with `type: "sms"` in `sequences create --steps-*` or `update --insert-steps-*`; steps only send when the add-on is enabled
- MCP parity: `get_sms_settings`, `send_test_sms`, and `generate_sms`

## Sync Rules

```bash
sequenzy sync-rules get
sequenzy sync-rules update --rules-file ./rules.json
sequenzy sync-rules update --reset
```

Behavior:

- `GET`/`PUT /api/v1/sync-rules`; sync rules map integration events to tags applied on subscribers
- `--rules-json`/`--rules-file` replace the rule set; `--reset` restores defaults
- MCP parity: `get_sync_rules` and `update_sync_rules`

## Notification Preferences

```bash
sequenzy account notifications get
sequenzy account notifications set --new-subscriber instant --campaign-completed instant
```

Behavior:

- `GET`/`PATCH /api/v1/notification-preferences`; these are the caller's own settings, not the workspace's - teammates keep their own
- new-subscriber notifications fall back to a daily summary automatically once a day passes the instant cap, and imports never trigger them
- `daily` is not valid for `--campaign-completed`: a campaign finishes once, so there is nothing to roll up
- MCP parity: `get_notification_preferences` and `update_notification_preferences`

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

## Feedback

```bash
sequenzy feedback "No command to bulk-delete campaigns by label" --category missing_capability
sequenzy feedback "Segment count and list output disagree for seg_123" --category bug --context "Auditing a re-engagement segment"
```

Behavior:

- sends `POST /api/v1/feedback`
- body shape is `{ message, source: "cli", category?, context? }`
- categories: `missing_capability`, `bug`, `docs`, `ux`, `praise`, `other` (default `other`)
- MCP equivalent: `submit_feedback` (message, optional category and context; source is set to `mcp` automatically)

Caveat:

- feedback is fire-and-forget: it is delivered straight to the Sequenzy team and there is no way to query it afterwards

## Commands To Treat As Unsupported

Treat these requested workflows as unsupported in the CLI even though related nouns exist:

- campaign immediate send; there is no "send now" command, so schedule with a near-future `--at` timestamp instead
- sending domain add/verify: no CLI command; use MCP `add_sending_domain` / `verify_sending_domain` or the dashboard, and `sequenzy urls --settings-tab domain` for the settings link
- form creation and embed snippets: CLI covers `forms list` and `forms update` only; use MCP `create_form` / `get_form_embed` or the dashboard
- image asset upload: MCP `upload_image_asset` only

## Operational Caveats

- prefer `SEQUENZY_API_KEY` for automation instead of interactive login
- use `--json` when another tool or agent needs structured output; dashboard-aware commands add `url`/`appUrls` fields when possible
- destructive commands (`delete`, `delete-variant`, `cancel-invitation`, and similar) prompt for confirmation; pass `--yes` (or `-y`) to skip, and note that `--yes` is required when stdin is not a TTY, which covers most agent and CI runs
- `campaigns cancel` deliberately skips the confirmation prompt so a bad send can be stopped fast
- `webhooks create` returns a one-time signing secret; surface it to the user immediately because it cannot be retrieved later
- when the user asks for a workflow outside the current CLI surface, say so directly, choose between dashboard or direct API use instead of inventing commands, and report the gap with `sequenzy feedback "..." --category missing_capability` (MCP: `submit_feedback`)
