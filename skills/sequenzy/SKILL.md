---
name: sequenzy
description: Agent guide for operating Sequenzy. Use when Codex needs to authenticate, inspect identity, manage subscribers, send a transactional email, read delivery stats, or decide whether a requested Sequenzy workflow is currently supported. Prefer the CLI when it is implemented, and fall back to the dashboard or direct API use when the current CLI surface is only partial.
---

# Sequenzy

## Overview

Use this skill when the task is to operate Sequenzy, not to change Sequenzy's source code. Prefer the `sequenzy` CLI for supported workflows and explicitly call out when a requested workflow is not wired in the current implementation.

## Ground Rules

1. Treat `packages/cli/src/index.tsx` as the source of truth for which commands are actually wired.
2. Treat `packages/cli/src/commands/` and `packages/cli/src/api.ts` as the source of truth for behavior, payload shape, and API routes.
3. Do not promise support for commands that only appear in docs or `--help` text without an attached action handler.
4. Prefer `sequenzy login` for interactive auth and `SEQUENZY_API_KEY` for automation.
5. Prefer inspection before mutation whenever the workflow allows it.

## Supported Workflows

Read [references/use-cases.md](references/use-cases.md) before executing anything non-trivial. The currently implemented CLI flows are:

- login and logout
- session check with `whoami`
- stats overview or stats by campaign/sequence ID
- subscribers `list`, `add`, `get`, and `remove`
- send one transactional email by template or raw HTML

## Unsupported Or Placeholder Workflows

The CLI currently advertises extra nouns such as `campaigns`, `sequences`, `templates`, `tags`, `lists`, `segments`, `account`, `websites`, and `generate`, but the current command tree does not attach handlers for them. Treat these as unsupported until the implementation changes.

## Execution Pattern

1. Check auth first with `sequenzy whoami` or by verifying `SEQUENZY_API_KEY` is set.
2. Pick the narrowest command that matches the use case.
3. Validate IDs, recipient email, subject, template, or HTML input before issuing a mutation.
4. Surface CLI limitations directly instead of inventing a workaround.
5. If the workflow is unsupported in the CLI, say whether the next-best path is the Sequenzy dashboard or direct API use.

## References

- [references/command-reference.md](references/command-reference.md): exact command shapes, env vars, behavior, and caveats.
- [references/use-cases.md](references/use-cases.md): decision trees and examples for the most common agent tasks.
