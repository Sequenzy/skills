---
name: sequenzy-email-marketing
description: Operate Sequenzy email marketing through the Sequenzy connector. Use when the user wants to inspect or change subscribers, lists, tags or segments, draft, review, schedule or analyze campaigns, build or edit automated sequences, edit templates, send transactional email, check sending domains, triage inbox conversations, or read delivery and engagement stats in their Sequenzy account.
---

# Sequenzy email marketing

Sequenzy is an email marketing platform for SaaS and ecommerce teams. This skill works through the Sequenzy connector (the Sequenzy MCP server), which the user signs in to with their Sequenzy account. Every action runs as that user, limited to the companies and permission level they approved at sign-in.

## Ground rules

1. The connector's tool list (names, descriptions and input schemas) is the source of truth for what you can do. Do not promise a workflow that has no tool; say so and point the user to the Sequenzy dashboard instead.
2. If a tool reports that sign-in is required, ask the user to reconnect Sequenzy from their connector settings. Never ask the user to paste a password or any other credential into the chat.
3. Inspect before you change anything. Read the current object, show the user what will change, then write.
4. Anything that sends email, enrolls contacts, activates an automation or deletes data needs the user's explicit go-ahead in this conversation, with the recipients or scope spelled out. Tools are annotated as read-only, destructive or open-world; treat destructive and open-world tools as needing that approval.
5. Never invent IDs, email addresses, metrics or customer quotes. Look them up.

## Choosing the company

Start with `get_account`. It lists the companies the user can reach, their role, which scopes their role blocks (`roleRestrictedScopes`) and whether any delivery workflow is available (`canSendLive`). When there is more than one company, confirm which one the user means, then call `select_company` or pass `companyId` on each call.

Marketers can manage marketing content, subscribers, campaigns and sequences, including test sends. They cannot send transactional email or manage team, billing, integrations, sending identities or workspace settings. When a tool refuses for a role reason, explain that rather than looking for a workaround.

## Common workflows

### Subscribers, lists, tags and segments

- Find people with `search_subscribers` and `get_subscriber`; `get_subscriber_activity` shows what they received and did.
- Add or update one contact with `add_subscriber` / `update_subscriber`. These fire the automations the user has set up (welcome sequences, tag triggers), so mention that before adding real contacts.
- For bulk work, use `create_subscriber_import`, `add_subscribers_to_list`, `bulk_add_subscriber_tags` or `start_subscriber_operation`, and report counts and failures from the result.
- Build segments with `create_segment`; check the size with `get_segment_count` before using one as an audience.
- "Why did this person stop getting email?": check `get_subscriber` (status, unsubscribes), `list_recipient_suppressions` (bounces and complaints) and `get_subscriber_activity`. Remove a suppression only when the user confirms the address is valid again.

### Campaigns

1. Draft with `create_campaign` (or `create_campaign_for_audience` for a selection the user already made). `generate_email` and `generate_subject_lines` write copy in the company's brand style.
2. Review with `render_email` and run `check_email`. Fix every `error` issue, especially `links.broken`; `restricted` links only refused an automated check and need a manual look.
3. Confirm the audience with `get_campaign_audience` and send a test with `send_test_email`. A test is a real email that is queued, not proof of delivery; check it with `get_email_send`.
4. Schedule with `schedule_campaign` only after the user approves the content, audience and time. There is no "send now": schedule a time a few minutes ahead. `unschedule_campaign`, `pause_campaign` and `cancel_campaign` stop it.
5. Report results with `get_campaign_stats` and `get_link_click_stats`.

Campaign A/B tests use the `*_ab_test*` tools. Select a winner or change a running test only when the user asks.

### Sequences (automations)

- Start from `list_sequence_templates` / `create_sequence_from_template`, or describe the goal to `generate_sequence`. Generated sequences hold at most 10 emails.
- Edit with `get_sequence`, `edit_sequence_emails`, `update_sequence_node` and `edit_sequence_graph`. Show the flow with `preview_sequence`.
- Before activating, run `simulate_sequence` and read its `warnings`, and run `check_email` on each email. Use the same sample contact across steps when you render or test.
- Activate with `enable_sequence` only when the user asks. Leave new sequences inactive for review by default. Enrolling existing contacts (`enroll_sequence_audience`, `enroll_subscribers_in_sequence`) is a separate decision that sends real email; past events are never replayed.

### Templates and transactional email

- Templates: `list_templates`, `get_template`, `create_template`, `update_template`. Conditional content needs block JSON (`get_email_block_schema`), not raw HTML.
- Transactional email: `list_transactional_emails` and `send_email`. Pass one stable idempotency key per logical email and keep the returned `emailSendId`. Queued does not mean delivered; check with `get_email_send`.

### Sending setup and deliverability

- `list_websites` and `check_website` show sending-domain status; `verify_sending_domain` runs a fresh DNS check. DNS verification and readiness to send are separate outcomes.
- Relay DNS records exactly as returned. Do not suggest removing records unless the tool result says they are no longer needed.
- `get_sending_status` explains a paused account; resume only when the user asks and the cause is resolved.

### Stats

`get_stats` gives an overview; `get_stats_breakdown`, `get_campaign_stats`, `get_sequence_stats` and `get_transactional_stats` go deeper. Say which date range you used. Opens and clicks are unique per email send, and rates divide by delivered email.

### Inbox

Read with `list_conversations` and `get_conversation`. Draft replies for the user to approve before calling `reply_to_conversation`, since the reply goes to a real person.

## Errors and retries

Failed tools return an error envelope with a reason and often a fix. Read it before retrying. Reads are safe to retry. Writes are never retried automatically: check whether the first attempt took effect, then repeat with the same input and idempotency key. Honor `retryAfterSeconds` when present.

## Dashboard links

When you create, change or schedule something the user may want to review, give them the link from the tool result (`url`, `previewUrl` or `appUrls`), or generate one with `get_app_urls`. The dashboard lives at `https://sequenzy.com`:

- campaign editor: `/dashboard/company/{companyId}/campaign/{campaignId}` (add `?step=review` for the review page)
- sequence editor: `/dashboard/company/{companyId}/sequences/{sequenceId}`
- email editor: `/dashboard/company/{companyId}/emails/{emailId}`
- settings: `/dashboard/company/{companyId}/settings?tab={tab}`, with tabs such as `domain`, `tracking`, `integrations`, `goals` and `team`

Integrations, team invitations, billing and other account administration are best done in the dashboard; point the user there.

## Feedback

Call `submit_feedback` only when the user asks you to send feedback. Tell them first that it goes to the Sequenzy team with their company name and account email attached, and leave out subscriber data and email content.
