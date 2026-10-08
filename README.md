# skills

Claude Code skills from the Stratdeck repo.

- [`address-pr`](address-pr/SKILL.md): address every review comment on a pull request, then verify, push and reply to
  each thread.
- [`changes-mode`](changes-mode/SKILL.md): a fast UI iteration loop that defers tests and e2e until
  `/changes-mode done`.

## Use

Copy a skill's folder into `.claude/skills/` in your project (or `~/.claude/skills/` for every project), then run it
with `/<skill-name>`.

These were written for Stratdeck, a Next.js app in `apps/web`, so they name its paths, scripts and tools. Adjust those
to your repo.
