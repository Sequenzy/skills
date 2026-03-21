# Sequenzy Skills

Versioned AI-agent skills for operating Sequenzy.

## Install

If this repository is published as `sequenzy/skills`, install the main skill with:

```bash
npx skills add sequenzy/skills --skill sequenzy
```

For a local checkout:

```bash
npx skills add . --skill sequenzy -a codex -y
```

## Repository Layout

```text
skills/
  sequenzy/
    SKILL.md
    agents/openai.yaml
    references/
```

## Notes

- Keep the installable skill inside `skills/<skill-name>/`
- Keep `SKILL.md` as the source of truth for triggers and workflow
- Add more skills to `skills/` over time without changing the repo name
