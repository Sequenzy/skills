# Sequenzy for Claude and Hermes Agent

[Sequenzy](https://sequenzy.com) is an email marketing platform for SaaS and ecommerce teams. This plugin lets Claude or Hermes Agent work in your Sequenzy account: look up and update subscribers, build segments, draft and schedule campaigns, edit automated sequences and templates, send transactional email, and read delivery and engagement stats.

## What the plugin contains

- **Sequenzy MCP connector** (`.mcp.json` for Claude, `mcp.json` for Hermes Agent): the remote server at `https://api.sequenzy.com/v1/mcp`. You sign in with your Sequenzy account through OAuth the first time the agent uses it, and pick the permission level it gets. Every tool is annotated as read-only or as changing data, so the agent can ask for your approval before anything that writes, sends or deletes.
- **`sequenzy-email-marketing` skill**: instructions that teach the agent the Sequenzy workflows, which tool to use for each task, and the checks to run before sending or deleting anything.

The plugin has no hooks or scripts. In Claude it installs nothing and runs no local server; in Hermes Agent it runs the `mcp-remote` sign-in bridge described below. Apart from its own sign-in token, it reads no credentials from your machine: the connector signs in with OAuth.

The skill's reference files also document the optional [Sequenzy CLI](https://docs.sequenzy.com) for people who run it themselves in a terminal. Those examples mention CLI environment variables such as `SEQUENZY_API_KEY`, which the CLI reads on your machine when you run it. The plugin does not use them.

## Install in Hermes Agent

You need Hermes Agent 0.21.6 or newer and Node.js 20.18.1 or newer with `npx` on your `PATH`.

From the Hermes plugin catalog:

```bash
hermes plugins install sequenzy
hermes plugins enable sequenzy
```

Start a new session. The first time the server starts, your browser opens the Sequenzy sign-in page. Sign in, pick the permission level Hermes gets, and approve. You sign in once; the token is kept for later sessions. If the tools don't show up after you sign in, run `/reload-mcp`.

The Hermes package is `plugin.json`, `mcp.json` and the skill in `skills/sequenzy-email-marketing/`. It ships no code of its own. Before you install, know that:

- `mcp.json` runs the pinned `mcp-remote@0.15.2` through `npx`, which downloads it from npm on first use. It opens a local callback port for the sign-in and stores the OAuth token in `~/.mcp-auth`.
- Every tool call goes to `api.sequenzy.com` and acts on your Sequenzy account with the permission level you approved. The server decides the tool set at runtime, so the pinned commit doesn't cover it.
- `plugin.json` marks the server `untrusted`. Hermes asks before every tool the server doesn't mark read-only (creating or changing subscribers, campaigns, sequences and templates, sending email, deleting anything), and refuses those tools in cron and other unattended runs.
- Emails you send or schedule go to real recipients and count toward your Sequenzy plan. AI email generation uses your plan's AI allowance.
- The package sends no telemetry.

On a server or another machine without a browser, skip the plugin and add the server to `~/.hermes/config.yaml` with a personal API key (it starts with `seq_user_`) from **Account Settings → API Keys** in Sequenzy:

```yaml
mcp_servers:
  sequenzy:
    url: "https://api.sequenzy.com/v1/mcp"
    headers:
      Authorization: "Bearer ${SEQUENZY_API_KEY}"
```

Put `SEQUENZY_API_KEY=...` in your Hermes environment file. The key works like a password for your Sequenzy account, so don't paste it into a chat.

## Install in Codex

You need Node.js 20.18.1 or newer with `npx` on your `PATH`. Add this repository as a plugin marketplace, then install the plugin:

```bash
codex plugin marketplace add Sequenzy/skills
codex plugin add sequenzy@sequenzy
```

In the ChatGPT desktop app, the plugin then shows up under the `Sequenzy` source in the Plugins Directory. Start a new session after installing.

Codex reads the same portable package as Hermes Agent: `plugin.json`, `mcp.json` and the skill in `skills/sequenzy-email-marketing/`. `.codex-plugin/plugin.json` only adds the listing details Codex shows (name, description, icon, links and starter prompts). The connector runs the pinned `mcp-remote@0.15.2` sign-in bridge, so the first time Codex uses it your browser opens the Sequenzy sign-in page. The bridge registers itself as "Hermes Agent (Sequenzy)", which is the name you see on the approval page. You can limit or turn off the server's tools in `~/.codex/config.toml` under `[plugins."sequenzy@sequenzy".mcp_servers.sequenzy]`.

Without a browser, connect the server directly with a personal API key (it starts with `seq_user_`) instead of the plugin:

```bash
export SEQUENZY_API_KEY=seq_user_...
codex mcp add sequenzy --url https://api.sequenzy.com/v1/mcp --bearer-token-env-var SEQUENZY_API_KEY
```

## Install in Kiro

This folder is also a Kiro power in the Agent Plugins format. You need Node.js 20.18.1 or newer with `npx` on your `PATH`.

1. Open the **Powers** panel and select **Add Custom Power**.
2. Select **Import power from GitHub** and enter `https://github.com/Sequenzy/skills/tree/main/plugins/sequenzy`.
3. Mention Sequenzy or an email-marketing task in chat. Kiro activates the power from the keywords in `plugin.json` and loads the skill and the MCP server.

The first time the server starts, your browser opens the Sequenzy sign-in page through the same `mcp-remote` bridge described above. Kiro names the server `power-sequenzy-sequenzy`.

## Example requests

- "Show me last week's campaigns and which one had the best click rate."
- "Create a segment of subscribers who opened an email in the last 30 days but never purchased."
- "Draft a three-email welcome sequence for new trial users and leave it inactive for me to review."
- "Why did this subscriber stop receiving emails?"

## Data handling

The plugin sends requests only to Sequenzy (`api.sequenzy.com`), using the account you signed in with. What the agent reads or changes is limited to the account and permission level you granted during sign-in. Sequenzy stores and processes that data under its [privacy policy](https://sequenzy.com/privacy) and [terms](https://sequenzy.com/terms). The skill itself stores nothing.

## Requirements

A Sequenzy account. You can create one for free at [sequenzy.com](https://sequenzy.com).

## Documentation and support

- MCP setup and tool reference: [docs.sequenzy.com/concepts/mcp](https://docs.sequenzy.com/concepts/mcp)
- API reference: [docs.sequenzy.com/api-reference](https://docs.sequenzy.com/api-reference)
- Support: hello@sequenzy.com

## Privacy policy and support

- Privacy policy: [sequenzy.com/privacy](https://sequenzy.com/privacy)
- Terms of service: [sequenzy.com/terms](https://sequenzy.com/terms)
- Support: [hello@sequenzy.com](mailto:hello@sequenzy.com) or [sequenzy.com/contact](https://sequenzy.com/contact)

## License

MIT. See [LICENSE](LICENSE).
