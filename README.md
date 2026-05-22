# skills-market

A public market of Claude Code skills — reusable, portable, no personal infra wired in. Drop any of these into `~/.claude/skills/` and they work out of the box.

## What's a skill?

A Claude Code skill is a Markdown file (`SKILL.md`) with YAML frontmatter that defines a reusable workflow Claude can invoke with a slash command. See the [official docs](https://docs.claude.com/en/docs/claude-code/skills) for the full spec.

## Skills in this repo

| Skill | What it does |
|-------|--------------|
| [`work-logs`](skills/work-logs/SKILL.md) | Write a dated work log into `docs/work logs/` in the current repo. |
| [`screenshot`](skills/screenshot/SKILL.md) | Capture full-page + per-section screenshots of a running localhost page. |
| [`ss`](skills/ss/SKILL.md) | Find the newest screenshot on Desktop/Downloads, analyze it, then delete it. |
| [`production-lint`](skills/production-lint/SKILL.md) | Run `tsc`, `eslint`, `build`, and `depcheck` in order before deploying. |
| [`inspect`](skills/inspect/SKILL.md) | Deep-inspect the current repo — architecture, data flow, patterns. |
| [`checklist`](skills/checklist/SKILL.md) | Parse a natural-language prompt into ordered tasks, approve, execute. |

## Install

Copy any skill into your local skills directory:

```bash
git clone https://github.com/iudofia/skills-market.git
cp -r skills-market/skills/work-logs ~/.claude/skills/
```

Or symlink for live updates:

```bash
ln -s "$(pwd)/skills-market/skills/work-logs" ~/.claude/skills/work-logs
```

Then in any Claude Code session, invoke with the slash command (e.g. `/work-logs`).

## Contributing

PRs welcome. Two rules:

1. **No personal infra.** No hardcoded paths, no specific repo names, no private services. Everything should work for someone who just cloned the repo.
2. **Self-contained.** Each skill lives in its own directory under `skills/<name>/`. No cross-skill dependencies unless documented.

## License

MIT — see [LICENSE](LICENSE).
