---
name: changes-mode
description:
  Fast UI iteration loop. Claude starts the dev server, the user checks each edit in their own browser; Claude edits,
  lints, counts the changes, and reports, deferring tests and e2e until `/changes-mode done`.
---

# Changes mode

The user is about to make a run of small changes and will test each one on their own localhost. Speed of the loop
matters more than proof per edit.

## `/changes-mode` (start)

First, start the dev server (the equivalent of `npm run dev`) with `preview_start`, reusing it if it is already up:

- In a worktree, before starting: `next dev` reads only the worktree's own `apps/web/.env.local`, not the main
  checkout's root `.env`, so sign-in silently does nothing without the Google keys. Append each of `AUTH_GOOGLE_ID`,
  `AUTH_GOOGLE_SECRET` and `POSTGRES_URL` that `apps/web/.env.local` lacks, copied from the main checkout's root `.env`
  (`git rev-parse --path-format=absolute --git-common-dir` gives its `.git`) with `grep >>` so the values are never
  printed. Confirm `git check-ignore apps/web/.env.local` first. If a server was already up without them, restart it.
- Use the `web` entry in `.claude/launch.json` when the user has a database configured, otherwise the file-backed entry
  (`POSTGRES_URL=`, `DATABASE_URL=`, `next dev --port 3000`). Add a platform-appropriate entry if none exists.
- Tell the user the URL. Every page is public; watchlists and other account features need a real Google sign-in on
  localhost.
- Set the change counter to 0.

Then, until told otherwise:

- Make the edit, run Prettier and ESLint on the touched files, and report in two or three lines what changed and where.
- Count each user request that lands as an edit as one change. When the counter reaches 10, and again at every further
  10, end the report with: "10 changes have now been made, so it may be time to run `/changes-mode done` and open a PR."
- Do not run Vitest or Playwright. The dev server holds port 3000, so Playwright cannot start anyway.
- Do not commit or push unless asked.
- Keep a running list of touched files and of any test, fixture or e2e rule you know will need updating. Do not update
  tests yet unless the edit breaks a type or a contract.
- Ask nothing that can wait. Batch questions and raise them with the report.

## `/changes-mode done` (finish)

Run the deferred verification once, in this order, and report failures with output:

1. `npx prettier --check` and `npx eslint` on the touched files.
2. `npx tsc --noEmit` from `apps/web`, where TypeScript is installed (delete `apps/web/.next` first if routes moved).
3. Vitest for the touched modules.
4. Update the tests and fixtures from the running list.
5. Stop the dev server with `preview_stop`, then run the affected Playwright specs via `tools/e2e-affected.js`.
6. `node tools/check-comment-density.js` on the diff.

Then summarise what was changed across the whole run, how many changes were made, and what still needs the user's
decision.
