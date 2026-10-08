---
name: address-pr
description:
  Address every review comment on a pull request, however small, then verify, push and reply to each thread. Use once a
  PR is raised and reviewed, or when the user says to address, fix or resolve the PR comments.
---

# Address PR comments

Code quality and correctness matter more than the size of the diff. Every comment gets a fix or a reasoned reply:
nothing is waved through because it is a nit, marked "minor", or said to be non-blocking.

`/address-pr` takes an optional PR number or URL; without one, use the PR for the current branch
(`gh pr view --json number`).

## 1. Gather

Read everything before changing anything:

- Reviews: `gh pr view <n> --json reviews`
- Top-level comments: `gh pr view <n> --json comments`
- Inline threads: `gh api repos/{owner}/{repo}/pulls/<n>/comments`. Keep each `id` for the reply.
- Failing checks: `ccd_pr` `get_status`, then the failing job's log. A red check is a comment too.

Skip deploy bots and status tables (Vercel previews and similar). Everything else, including bullet points buried in a
review summary and remarks marked FYI, is an item.

## 2. Triage

List every item with one decision:

- **Fix**: the default, for bugs, nits, naming, consistency, missing tests and docs alike.
- **Fix, and look wider**: when the comment names one instance of a pattern, check for the same problem elsewhere in the
  diff and fix those too.
- **Decline**: only when the change would be wrong or would contradict a product or engineering rule in
  `stratdeck-product.md`. The reply must say why, with evidence (a file, a test, a rule), not an opinion.
- **Deliberate tradeoff**: when the reviewer flags something the PR intends. Say what was traded and why it is
  acceptable. Fix it anyway if it's cheap to fix.

Before fixing, confirm each claim against the code. Reviewers are sometimes wrong, and sometimes understate: check
whether the real failure is worse than described, and fix what is really there.

## 3. Fix

- Follow the repo's engineering rules: existing patterns, strong types, no new abstractions, sparse comments.
- For every correctness fix, add or tighten a test that covers it, and prove the test catches the bug: run it once with
  the fix reverted and see it fail, then restore the fix. Mention this in the reply.
- Keep fixes to what the comments ask for. File unrelated problems you notice as separate tasks rather than widening the
  PR.

## 4. Verify

On the touched files: `npx prettier --check`, `npx eslint`, `npx tsc --noEmit -p apps/web`, and Vitest for the touched
modules. Run the Playwright specs that cover the change (`tools/e2e-affected.js`, or the specific specs when a
suite-wide file forces everything), with no dev server holding the port. Report failures with their output. Do not push
red.

## 5. Push and reply

- One commit per round, `fix(<scope>): …`, whose body lists what each comment led to. The pre-push hook runs `ci-fast`;
  let it finish.
- Reply on each inline thread (`gh api …/pulls/<n>/comments/<id>/replies`) with the commit SHA, what changed, and how it
  was verified.
- Post one PR comment summarising every item and its outcome, including declined ones and why.
- Never merge or enable auto-merge, and never dispatch a deploy; the user does those.

## 6. Loop

A push can trigger a fresh review. Check the PR again once CI finishes. If new comments or failing checks appear, run
another round. Stop when every item has a fix or a reply and the checks are green or pending, then tell the user what
was fixed, what was declined, and what still needs their decision.
