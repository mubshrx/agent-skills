---
name: commit-staged
description: Generate conventional commit messages (subject and body) from staged changes, extracting the ticket ID from the current branch name. Use when the user wants to create a commit, write a commit message, commit staged changes, or asks for a commit with conventional format (e.g. feat(PROJ-123)).
---

# Conventional Commit from Staged Changes

## When to Use

Apply this skill when the user asks to create a commit, write a commit message, or commit staged changes, especially when they mention conventional commits or a ticket number in the branch.

## Workflow

### 0. Multi-root workspace: which project?

The user often has **multiple projects** in a single VS Code/Cursor workspace (e.g. a `.code-workspace` with several folders). Each folder is usually its own git repo.

- **Do not run any commands** to detect which project has staged changes.
- If it's **unclear which project** to generate the commit for, **ask the user at the start** (before running any git commands). Use the **AskQuestion** tool when available, with the workspace folder names as options; otherwise ask in chat (e.g. "Which project should I use for this commit?" and list the folder names).
- Only after the project is known, run git commands from that project's root (e.g. `cd path/to/project && git ...` or use the `working_directory` / `cwd` for that path).

### 1. Get branch name and staged changes

```bash
git branch --show-current
git diff --staged --stat
git diff --staged
```

Run these from the chosen project root. Use the output to infer scope and ticket. If no changes are staged, tell the user to stage first (`git add`).

### 2. Extract ticket from branch name

Branch names often look like: `PROJ-194-ui-mismatch-in-email-management`, `feature/JIRA-123-add-login`, `fix/ABC-456`.

- Match the first segment that looks like `PREFIX-NUMBER` (e.g. `PROJ-194`, `JIRA-123`). Common patterns: uppercase letters + hyphen + digits.
- Use that as the scope in parentheses: `(PROJ-194)`, `(JIRA-123)`.
- If no ticket pattern is found, omit the scope or use a short scope from the change (e.g. module or area).

### 3. Choose conventional type

From the staged diff, pick one:

| Type       | Use when                                   |
| ---------- | ------------------------------------------ |
| `feat`     | New feature or user-facing capability      |
| `fix`      | Bug fix                                    |
| `chore`    | Build, tooling, deps, config, no app logic |
| `docs`     | Documentation only                         |
| `style`    | Formatting, whitespace, no logic change    |
| `refactor` | Code change that is not a fix or feature   |
| `test`     | Adding or updating tests                   |
| `perf`     | Performance improvement                    |

Default to `feat` for new behavior, `fix` for defect fixes, `chore` for everything else when unclear.

### 4. Write the subject line

- Format: `type(scope): imperative subject`
- Scope = ticket ID from branch (e.g. `PROJ-194`).
- Subject: lowercase, imperative ("add" not "added"), no period at end, ~50 characters or less.
- Example: `feat(PROJ-194): fix email management UI mismatch`

### 5. Write the body (optional)

- Add a body if the change needs explanation (what changed and why).
- **Put each item on its own line** (no bullets); use newlines inside the body string.
- Wrap long lines at ~72 characters.

### 6. Output the git command

**Always output a copy-pastable `git commit` command**, not just the message text. Use one `-m` for the subject; if there is a body, add a second `-m` with the body. Use real newlines inside the body string so each item is on a separate line. In a multi-root workspace, prefix with `cd <project-path> && ` so the command runs in the chosen project, or tell the user to run it from that folder.

- Subject only: `git commit -m "type(scope): subject"`
- Subject + body (multiple items on separate lines): use one `-m` with newlines inside the quoted string.

## Examples

**Example 1**

- Branch: `PROJ-194-ui-mismatch-in-email-management`
- Staged: Changes in email management modals and notification CC list UI.

```bash
git commit -m "feat(PROJ-194): align email management UI with design" -m "Update notification CC modals and list section to match specs
Fix avatar and remitter sections styling"
```

**Example 2**

- Branch: `fix/PROJ-200-login-timeout`
- Staged: Increase auth timeout and retry logic.

```bash
git commit -m "fix(PROJ-200): increase login timeout and add retry" -m "Prevent session expiry during slow networks by extending timeout and retrying once on failure."
```

**Example 3**

- Branch: `chore-deps`
- No ticket in branch; staged: package.json and lockfile.

```bash
git commit -m "chore(deps): bump axios to 1.6.0"
```

## Checklist

- [ ] If multi-root and project unclear, ask user which project at the start (do not run commands to detect)
- [ ] Read branch name and staged diff from the chosen project
- [ ] Ticket/scope from branch (or omit if none)
- [ ] Type chosen from content (feat/fix/chore/etc.)
- [ ] Subject: `type(scope): imperative`, &lt; ~50 chars
- [ ] Body only if needed; one item per line; wrap at 72 chars
- [ ] Output a copy-pastable `git commit` command (not just the message)
