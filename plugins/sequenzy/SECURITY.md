# Security policy

## Supported versions

Only the latest version of the Sequenzy plugin on the `main` branch of [Sequenzy/skills](https://github.com/Sequenzy/skills) is supported. The plugin ships no executable code of its own: it contains manifests, a skill, and MCP configuration that points at the hosted Sequenzy server (`https://api.sequenzy.com/v1/mcp`). Security fixes to the server apply to every installed copy without an update.

## Reporting a vulnerability

Email **hello@sequenzy.com** with the subject line `Security report`. Include:

- what you found and where (plugin file, skill instruction, MCP tool or API endpoint);
- steps to reproduce, and the impact you expect;
- whether the issue is already public.

Please don't open a public GitHub issue for a vulnerability and don't test against Sequenzy accounts or data that aren't yours. We confirm receipt within three business days, keep you updated while we investigate, and credit you in the fix notes if you want.

## Scope

In scope: the files in this plugin folder, the hosted MCP server at `api.sequenzy.com`, and its OAuth sign-in. Out of scope: third-party tools the plugin uses, such as `mcp-remote` or the agent host itself; report those to their maintainers.
