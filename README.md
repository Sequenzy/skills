# Sequenzy Skills

[![Agent Skill](https://img.shields.io/badge/Agent%20Skill-SKILL.md-6f42c1)](skills/sequenzy-email-marketing/SKILL.md)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-ready-111827)](#install)
[![Codex](https://img.shields.io/badge/Codex-ready-111827)](#install)
[![Hermes](https://img.shields.io/badge/Hermes-ready-111827)](#install)

Versioned AI-agent skills for operating [Sequenzy](https://sequenzy.com) email marketing workflows from Claude Code, Codex, Hermes, and other SKILL.md-compatible agents.

## Why this exists

Agents are increasingly asked to do real lifecycle marketing work, not just draft copy. This repo gives them a precise operating guide for Sequenzy so they can inspect account state, manage subscribers and segments, create campaigns or sequences, generate email drafts, send transactional emails, and surface review URLs without guessing command names or API payloads.

The installable skills are:

- [`sequenzy-email-marketing`](skills/sequenzy-email-marketing/SKILL.md): primary skill for operating campaigns, sequences, subscribers, segments, templates, transactional sends, stats, websites, and API keys through Sequenzy's CLI/API surface.
- [`sequenzy`](skills/sequenzy/SKILL.md): concise compatibility alias for broad Sequenzy references; use the explicit email-marketing skill when both apply.

## Install

Install the explicit email-marketing skill from GitHub:

```bash
npx skills add Sequenzy/skills --skill sequenzy-email-marketing
```

Or install the concise compatibility alias:

```bash
npx skills add Sequenzy/skills --skill sequenzy
```

Install for a specific agent when your `skills` CLI supports agent adapters:

```bash
# Claude Code
npx skills add Sequenzy/skills --skill sequenzy-email-marketing -a claude-code -y

# OpenAI Codex
npx skills add Sequenzy/skills --skill sequenzy-email-marketing -a codex -y

# Local checkout / development
npx skills add . --skill sequenzy-email-marketing -a codex -y
```

Hermes users can copy or sync the skill into their Hermes skills directory, then load it by name:

```bash
hermes skills list | grep sequenzy-email-marketing
```

## Quick start for agents

1. Load [`skills/sequenzy-email-marketing/SKILL.md`](skills/sequenzy-email-marketing/SKILL.md).
2. Read [`references/use-cases.md`](skills/sequenzy-email-marketing/references/use-cases.md) before non-trivial mutations.
3. Authenticate with either:

```bash
sequenzy login
# or, for automation
export SEQUENZY_API_KEY=...
```

4. Verify access before mutating anything:

```bash
sequenzy whoami
sequenzy account
```

## Example agent tasks

```text
Use Sequenzy to draft a 4-email onboarding sequence for new trial users, then give me the dashboard URL to review it.
```

```text
Check last 30 days campaign stats, identify the weakest campaign, and suggest subject-line improvements without sending anything.
```

```text
Add user@example.com as a subscriber with the tags beta and founder, then show the saved subscriber profile.
```

```text
Create a saved segment for users who purchased Pro at least 3 times and preview its count before we use it in a campaign.
```

## Discovery

This repo includes a machine-readable discovery file at [`.well-known/skills/index.json`](.well-known/skills/index.json). Catalogs and agents can use it to find the canonical skill path, compatibility notes, and install command.

## Repository layout

```text
.well-known/skills/index.json
skills/
  sequenzy/
    SKILL.md
    agents/openai.yaml
    references/
  sequenzy-email-marketing/
    SKILL.md
    agents/openai.yaml
    references/
      command-reference.md
      use-cases.md
```

## Contributing

Keep the installable skill inside `skills/<skill-name>/` and treat `SKILL.md` as the source of truth for triggers, supported workflows, caveats, and verification steps. Reference files should contain longer command examples and decision trees so the main skill stays readable.

When adding a new skill, also update `.well-known/skills/index.json` so catalogs can discover it.
