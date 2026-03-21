# Use Cases

## Pick The Right Flow

### "Log into Sequenzy on this machine"

Use:

```bash
sequenzy login
```

Then verify:

```bash
sequenzy whoami
```

Prefer `SEQUENZY_API_KEY` instead when the task is fully non-interactive or running in CI.

## "Check whether I am authenticated"

Use:

```bash
sequenzy whoami
```

Interpretation:

- success means a local API key is available
- failure means the agent should ask for login or a `SEQUENZY_API_KEY`

Remember that this is local-state validation, not a fresh server-side account lookup.

## "Show me delivery performance"

Use:

```bash
sequenzy stats
sequenzy stats --period 30d
sequenzy stats --campaign camp_123
sequenzy stats --sequence seq_123
```

Choose:

- plain `stats` for account-level overview
- `--campaign` when the user gives a campaign ID
- `--sequence` when the user gives a sequence ID

Ask for the missing ID instead of guessing.

## "Add or update a subscriber"

What works today:

```bash
sequenzy subscribers add user@example.com --tag premium --attr name=John
sequenzy subscribers get user@example.com
```

Guidance:

- use `add` for single-recipient creation
- use repeated `--attr key=value` pairs for metadata
- use only one `--tag` value unless the CLI implementation is fixed

What does not work well today:

- bulk import
- advanced subscriber workflows across many records

For those, prefer the Sequenzy dashboard or direct API use.

## "Remove a subscriber"

Use:

```bash
sequenzy subscribers get user@example.com
sequenzy subscribers remove user@example.com
```

Use `--hard` only when the task explicitly requires permanent deletion:

```bash
sequenzy subscribers remove user@example.com --hard
```

## "Send one transactional email"

Template flow:

```bash
sequenzy send user@example.com --template welcome --var name=John
```

Raw HTML flow:

```bash
sequenzy send user@example.com --subject "Status update" --html-file ./email.html --var orderId=123
```

Checklist:

1. Confirm recipient email.
2. Confirm template ID or HTML source.
3. Confirm subject when sending raw HTML.
4. Confirm merge variables as `key=value`.

This command is for one-off transactional send behavior, not bulk campaign sends.

## "Create or manage campaigns"

Do not rely on the CLI yet.

Reason:

- `campaigns` appears in help/docs
- the current command tree does not attach action handlers for those subcommands

Preferred fallback:

- use the dashboard
- use direct API calls only if the task explicitly allows it and the relevant API is available

## "Manage sequences or templates"

Treat these as unsupported in the current CLI for the same reason as campaigns: the nouns are declared, but the handlers are not wired.

## "Generate email content with AI"

Treat `sequenzy generate ...` as unsupported for now. The command names exist, but the current CLI does not attach handlers.

If the user still needs copy, generate it outside the CLI and clearly state that the Sequenzy CLI path is not implemented yet.
