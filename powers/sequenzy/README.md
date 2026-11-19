# Sequenzy power for Kiro

[Sequenzy](https://sequenzy.com) is an email marketing platform for SaaS and ecommerce teams. This power lets Kiro work in your Sequenzy account: look up and update subscribers, build segments, draft and schedule campaigns, edit automated sequences and templates, send transactional email, and read delivery and engagement stats.

## What the power contains

- **`mcp.json`**: the hosted Sequenzy MCP server at `https://api.sequenzy.com/v1/mcp`. Kiro connects to it directly; nothing is installed or run on your machine. Kiro names the server `power-sequenzy-sequenzy`.
- **`sequenzy-email-marketing` skill**: teaches Kiro the Sequenzy workflows, which tool to use for each task, and the checks to run before anything sends email or deletes data.

Kiro activates the power when your message mentions Sequenzy or an email marketing task, such as a campaign, sequence, newsletter or subscriber list. It stays unloaded the rest of the time.

## Install

1. In Kiro, open the **Powers** panel and select **Add Custom Power**.
2. Select **Import power from GitHub** and enter `https://github.com/Sequenzy/skills/tree/main/powers/sequenzy`.
3. Ask something about your Sequenzy account, for example "Which Sequenzy companies can I access?".

The first time Kiro calls the server, your browser opens the Sequenzy sign-in page. Sign in, pick the permission level Kiro gets, and approve. Kiro keeps the sign-in for later sessions. To sign out, remove the power or revoke the connection from your Sequenzy account.

## Before you install

- Every tool call goes to `api.sequenzy.com` and acts on your Sequenzy account, limited to the companies and permission level you approved at sign-in.
- Tools are annotated as read-only or as changing data, so Kiro can ask for your approval before anything that writes, sends or deletes.
- Emails you send or schedule go to real recipients and count toward your Sequenzy plan. AI email generation uses your plan's AI allowance.
- The power asks for no key or password and reads nothing from your machine. It sends no telemetry.

## Example requests

- "Show me last week's campaigns and which one had the best click rate."
- "Create a segment of subscribers who opened an email in the last 30 days but never purchased."
- "Draft a three-email welcome sequence for new trial users and leave it inactive for me to review."
- "Why did this subscriber stop receiving emails?"

## Requirements

A Sequenzy account. You can create one for free at [sequenzy.com](https://sequenzy.com).

## Documentation

- MCP setup and tool reference: [docs.sequenzy.com/concepts/mcp](https://docs.sequenzy.com/concepts/mcp)
- API reference: [docs.sequenzy.com/api-reference](https://docs.sequenzy.com/api-reference)

## Privacy policy and support

- Privacy policy: [sequenzy.com/privacy](https://sequenzy.com/privacy)
- Terms of service: [sequenzy.com/terms](https://sequenzy.com/terms)
- Support: [hello@sequenzy.com](mailto:hello@sequenzy.com) or [sequenzy.com/contact](https://sequenzy.com/contact)

## License

MIT. See [LICENSE](LICENSE).
