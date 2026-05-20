# GitHub Skill

How to use Git and GitHub CLI (`gh`) for branching, commits, and pull requests in this project.

## Setup

Git and `gh` CLI are pre-configured. The remote is already set.

- Main branch: **main**
- Git user: **songkeik-alt**

## Branch strategy

Create a feature branch per Jira ticket:

```bash
git checkout -b feature/SCRUM-38-login-screen-ui
```

Naming convention: `feature/<ticket-key>-<short-description>`

## Committing changes

### Pre-commit checklist

1. Run `git status` to see all changed/untracked files
2. Run `git diff` to review staged and unstaged changes
3. Run `git log --oneline -5` to match commit message style
4. Never commit files with secrets (`.env`, credentials, API keys)

### Commit message format

```bash
git commit -m "$(cat <<'EOF'
Add login screen UI with hero section and form fields

Co-Authored-By: Claude Opus 4.6 <noreply@anthropic.com>
EOF
)"
```

- Start with a verb: Add, Update, Fix, Remove, Refactor
- Keep the first line under 72 characters
- Add detail in the body if needed
- Always include the Co-Authored-By trailer

### Stage specific files

Prefer explicit file staging over `git add .`:

```bash
git add iOS-MCP-xcode26/LoginView.swift
git add iOS-MCP-xcode26/AuthStore.swift
git add iOS-MCP-xcode26/Assets.xcassets/icon_mail.imageset/
```

## Creating a pull request

### Steps

1. Push the branch:
   ```bash
   git push -u origin feature/SCRUM-38-login-screen-ui
   ```

2. Create PR with `gh`:
   ```bash
   gh pr create --title "Add login screen UI (SCRUM-38)" --body "$(cat <<'EOF'
   ## Summary
   - Implement login screen with hero section, email/password fields, and sign-in button
   - Add demo account auto-fill (nutty@gmail.com / 123456)
   - Local auth via UserDefaults, navigates to MapScreen on success

   ## Test plan
   - [ ] Login screen renders matching Figma design (≥80% visual match)
   - [ ] Email and password fields accept input
   - [ ] Password is masked by default, eye icon toggles visibility
   - [ ] Sign In validates credentials and navigates to map
   - [ ] "Use demo account" auto-fills and signs in
   - [ ] Keyboard does not obscure input fields

   ## Jira
   SCRUM-38

   ## Visual match score
   88% — SF Pro used in place of Instrument Sans (not bundled)

   Generated with [Claude Code](https://claude.com/claude-code)
   EOF
   )"
   ```

### PR title format

- Under 70 characters
- Include ticket key: `Add login screen UI (SCRUM-38)`
- Use verb prefix: Add, Update, Fix, Remove

### PR body template

```markdown
## Summary
<1-3 bullet points describing what changed and why>

## Test plan
<Bulleted checklist of verification steps>

## Jira
<Ticket key>

## Visual match score
<Score>% — <any intentional deviations>
```

## Reviewing PRs

### View PR details

```bash
gh pr view 123
gh pr diff 123
gh api repos/<owner>/<repo>/pulls/123/comments
```

### Check CI status

```bash
gh pr checks 123
```

## Working with issues

```bash
# List open issues
gh issue list

# View an issue
gh issue view 42

# Create an issue
gh issue create --title "Bug: ..." --body "..."
```

## Useful git commands

```bash
# View changes since branching from main
git diff main...HEAD

# View commit history for current branch
git log main..HEAD --oneline

# Stash work in progress
git stash push -m "wip: login screen"
git stash pop

# Check what branch tracks what remote
git branch -vv
```

## Safety rules

- Never force-push to `main`
- Never use `--no-verify` to skip hooks
- Never use `git reset --hard` without user confirmation
- Never amend published commits without asking
- Always create new commits instead of amending after hook failures
- Confirm with the user before pushing or creating PRs
