---
name: gh-draft-pr
description: Produces a gh CLI command to create a draft PR using the repo's PR template, filling the body from the current branch's commits and conversation context. Use when the user asks for a draft PR, create draft PR, open a PR with gh, or wants a PR command following the template.
---

# Draft PR with GitHub CLI

## When to Use

Apply when the user wants a **command** to create a **draft** pull request with the GitHub CLI (`gh`), using the **repository's PR template** and improving the description using **branch commits** and/or **chat context**.

## Workflow

### 0. Which repository?

In a **multi-root workspace**, if it's unclear which project to create the PR for, **ask the user first** (e.g. list workspace folder names). Run all git/gh commands from that project's root.

### 1. Locate PR template

From the chosen repo root, look for a pull request template:

- `.github/pull_request_template.md`
- `.github/PULL_REQUEST_TEMPLATE.md` (or `.MD`)

If none exists, use a minimal body (summary + how to test). If you asked which project to use, use the template from the **chosen repo**.

### 2. Gather branch and commit context

From the repo root:

```bash
git rev-parse --show-toplevel
git branch --show-current
git log origin/main..HEAD --oneline
# or: git log main..HEAD --oneline   if origin/main doesn't exist
```

Use the default branch (`main` or `master`) that the PR would target. Prefer `origin/main` / `origin/master` when available.

**Extract ticket from branch name** (e.g. `PROJ-194-ui-mismatch` → `PROJ-194`, `feature/CR-123` → `CR-123`) for "Closes" / "Related Ticket" in the template. Match first `PREFIX-NUMBER` (uppercase letters + hyphen + digits).

### 3. Fill the template

- **Summary / What & Why**: Use **bullet points** (one point per change or theme), derived from **commit messages** and **recent chat** (what was implemented or fixed). Do not use a single paragraph.
- **Related Ticket / Closes**: Use the ticket ID from the branch; link to Jira if the template uses `https://acme.atlassian.net/browse/XXX`.
- **How to Test**: From commits, file changes, and chat (routes, endpoints, flags, expected behavior). Leave placeholders if unknown.
- **Checklists**: Leave checkboxes as-is (unchecked) unless chat or commits clearly indicate something is done.
- **Screenshots / Additional notes**: Leave placeholders or brief notes if chat provides them.

**Remove template placeholders when you fill a section:** If you wrote actual content for a section (e.g. Summary, How to Test), **delete** that section’s instruction/pointer lines from the template (e.g. "> Provide a concise explanation...", "> Briefly describe how you tested..."). Only keep placeholder text (e.g. "> [screenshot.png]", "> Any reviewer notes...") in sections you did **not** fill with details.

Keep the same section headers and structure as the template so the body is valid for the repo.

### 4. Write body to a file and give the command

- Write the **filled body** to a file in the repo root, e.g. `.pr-body-draft.md` (or `pr-draft-body.md`). Use a path that is gitignored or that the user can delete later.
- Prefer a **single, copy-pastable command** that creates the draft PR.

**Standard form:**

```bash
gh pr create --draft --title "Brief title from branch/commits" --body-file .pr-body-draft.md
```

- **Title**: Short, imperative or descriptive (e.g. from first commit or branch name). Can use `--fill` to take title/body from commits, but then body is overridden by `--body-file`; so prefer an explicit `--title` that matches the work.
- **Body**: Always use `--body-file` with the filled template file so the PR follows the repo template.

If the user prefers not to leave a file on disk, use stdin:

```bash
gh pr create --draft --title "Brief title" --body-file -
```

Then show the filled body and tell them to paste it when the command prompts, or use a heredoc (e.g. `<< 'PRBODY' ... PRBODY`) in the same shell command.

### 5. Optional: base branch and repo

If the PR should target a different base:

```bash
gh pr create --draft --base develop --title "..." --body-file .pr-body-draft.md
```

If `gh` is not authenticated or repo is missing, remind the user to run `gh auth status` and push the branch first.

## Example

User is on branch `PROJ-194-ui-mismatch-in-email-management` in `admin-dashboard`, with 3 commits. Template has "Summary", "Related Ticket (Closes PROJ-XX)", "How to Test", checklists.

1. Repo root: `admin-dashboard`, template: `.github/pull_request_template.md`.
2. Branch: `PROJ-194-ui-mismatch-in-email-management`, ticket: `PROJ-194`, commits: e.g. "feat(PROJ-194): add notification CC modals", "feat(PROJ-194): wire API for notification CC list".
3. Fill template: Summary as bullet points (e.g. "- Adds notification CC add/edit/delete modals", "- Adds standalone notification CC list section", "- Fixes modal description and avatar styling"); Closes PROJ-194; How to Test from chat or "Settings → Notification CC section".
4. Write body to `admin-dashboard/.pr-body-draft.md`.
5. Output:

```bash
cd path/to/repo && gh pr create --draft --title "PROJ-194: UI mismatch in email management" --body-file .pr-body-draft.md
```

## Summary

- **Template**: Use the repo’s `.github/pull_request_template.md` (or variant).
- **Body**: Fill it from branch commits + chat; keep section structure. Write the **Summary** as bullet points, not a paragraph. **Remove** template instruction/pointer lines in any section you filled; leave placeholders only in sections you did not fill.
- **Ticket**: From branch name (e.g. PROJ-194, CR-123) for Closes/Related ticket.
- **Command**: `gh pr create --draft --title "..." --body-file <path>` (or `--body-file -` and provide body separately).
- **Multi-repo**: Ask which project if unclear; run commands from that repo root.
