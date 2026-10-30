# Sequenzy for Claude

[Sequenzy](https://sequenzy.com) is an email marketing platform for SaaS and ecommerce teams. This plugin lets Claude work in your Sequenzy account: look up and update subscribers, build segments, draft and schedule campaigns, edit automated sequences and templates, send transactional email, and read delivery and engagement stats.

## What the plugin contains

- **Sequenzy MCP connector** (`.mcp.json`): the remote server at `https://api.sequenzy.com/v1/mcp`. You sign in with your Sequenzy account through OAuth the first time Claude uses it, and pick the permission level Claude gets. Every tool is annotated as read-only or as changing data, so Claude can ask for your approval before anything that writes, sends or deletes.
- **`sequenzy-email-marketing` skill**: instructions that teach Claude the Sequenzy workflows, which tool to use for each task, and the checks to run before sending or deleting anything.

The plugin has no hooks, scripts or local servers, and it installs no packages.

## Example requests

- "Show me last week's campaigns and which one had the best click rate."
- "Create a segment of subscribers who opened an email in the last 30 days but never purchased."
- "Draft a three-email welcome sequence for new trial users and leave it inactive for me to review."
- "Why did this subscriber stop receiving emails?"

## Data handling

The plugin sends requests only to Sequenzy (`api.sequenzy.com`), using the account you signed in with. What Claude reads or changes is limited to the account and permission level you granted during sign-in. Sequenzy stores and processes that data under its [privacy policy](https://sequenzy.com/privacy) and [terms](https://sequenzy.com/terms). The skill itself stores nothing.

## Requirements

A Sequenzy account. You can create one for free at [sequenzy.com](https://sequenzy.com).

## Documentation and support

- MCP setup and tool reference: [docs.sequenzy.com/concepts/mcp](https://docs.sequenzy.com/concepts/mcp)
- API reference: [docs.sequenzy.com/api-reference](https://docs.sequenzy.com/api-reference)
- Support: hello@sequenzy.com

## License

MIT. See [LICENSE](LICENSE).
