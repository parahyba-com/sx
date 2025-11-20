---
applyTo: '**'
---

# HOW TO DEVELOP

## Development Step by Step

### 1. Create a Branch

- **NEVER commit directly to `master`**: All changes must go through a branch and PR workflow
- **Always create an issue first** to describe the feature, bug, or task
  - Use the `#github` tool to create, read, and manage issues on GitHub
- **Create branch LOCALLY using terminal commands** (not GitHub tools)
- **Branch name must reference the issue**: `<issue-number>-short-description`
  - Example: `123-add-user-authentication`, `456-fix-login-bug`, `789-update-docs`
- The branch creates an isolated space to work without affecting the main code
- Allows collaborators to review your work before integration

**Command (use terminal):**
```bash
git checkout -b <issue-number>-branch-name
```

**Example:**
```bash
git checkout -b 123-add-oauth-login
```

### 2. Make Changes

- Work on your branch making the necessary changes
- **Frequent and descriptive commits**: each commit should represent an isolated and complete change
- **ALL commits must be done LOCALLY using git commands in terminal** (not GitHub tools)
- Follow **Conventional Commits** as specified below

#### Commit Format (Conventional Commits)

```
<type>(<scope>): <subject>

[optional body]

[optional footer]
```

**Valid commit types:**
- **feat**: A new feature for the user (correlates with MINOR in SemVer)
- **fix**: A bug fix for the user (correlates with PATCH in SemVer)
- **docs**: Documentation only changes
- **style**: Changes that do not affect the meaning of the code (white-space, formatting, missing semi-colons, etc)
- **refactor**: A code change that neither fixes a bug nor adds a feature
- **perf**: A code change that improves performance
- **test**: Adding missing tests or correcting existing tests
- **build**: Changes that affect the build system or external dependencies (example scopes: gulp, broccoli, npm)
- **ci**: Changes to CI configuration files and scripts (example scopes: Travis, Circle, BrowserStack, SauceLabs)
- **chore**: Other changes that don't modify src or test files
- **revert**: Reverts a previous commit

**Scope (Optional):**
Indicate the affected area: `ui`, `api`, `auth`, `db`, `config`, etc.

**Subject Rules:**
- Use imperative mood: "add", not "added" or "adds"
- Must be lowercase (not Start-Case, PascalCase, UPPERCASE, or Sentence case)
- No period at the end
- Maximum 100 characters for the entire header (type + scope + subject)
- One commit = one logical change (facilitates reversions)

**Correct examples:**
```bash
feat(auth): add OAuth login support
fix(ui): resolve mobile menu overflow
docs: update installation instructions
refactor(api): simplify user validation logic
perf(db): optimize query with index
```

**Incorrect examples:**
```bash
Added new feature
Fixed stuff
Update
WIP
changes
```

**Body (Optional):**
- Explain **what** and **why**, not **how**
- Maximum 100 characters per line
- Separate from subject with a blank line

**Footer (Optional):**
- Reference issues: `Closes #123` or `Fixes #456`
- Note breaking changes: `BREAKING CHANGE: ...`
- Maximum 100 characters per line

**Breaking changes:**
Use `!` after type/scope or add `BREAKING CHANGE:` in the footer:
```bash
feat(api)!: remove deprecated endpoints

BREAKING CHANGE: v1 endpoints no longer available
```

**Commands (use terminal for all git operations):**
```bash
git add .
git commit -m "type(scope): subject"
git push origin branch-name
```

**Important practices:**
- Small and frequent commits (continuous backup)
- Separate branch for each set of unrelated changes
- Regular push to remote (backup and team visibility)
- **All git operations (add, commit, push) must be done in terminal**

### 3. Create a Pull Request (PR)

- **MANDATORY: Ensure an issue exists before opening the PR**
  - Every PR must be linked to an issue that describes the problem/feature
  - If no issue exists, create one first before proceeding with the PR
  - The issue number should already be part of your branch name from step 1
- **Use `#github` tool to create, read, and manage Pull Requests** (this is beyond local git)
- Open a PR when you want feedback or when changes are ready
- Mark as **Draft** if still in progress
- Include in the PR:
  - **Clear summary** of changes
  - **Problem it solves** (with link to issue - REQUIRED)
  - **Images/screenshots** when relevant
  - **Additional context** to facilitate review
- **Link the related issue** using keywords like `Closes #123`, `Fixes #456` (REQUIRED)
- Add comments on specific lines if needed
- Request specific reviewers or teams with `@mention`

**Automatic checks:**
- CI/CD pipelines will run
- Automated tests
- Linters and formatters
- Status checks must pass before merge

### 4. Respond to Review Comments

- Reviewers will leave questions, comments, and suggestions
- Respond to comments and make the requested changes
- Continue making commits and pushing to the same branch
- The PR will be automatically updated with new commits
- Discuss and clarify doubts directly in the PR
- Mark conversations as resolved after implementing changes

**Review cycle:**
```
Commit → Push → Review → Adjustments → Commit → Push → Approval
```

### 5. Merge the Pull Request

- After approval, merge the PR
- The branch will be integrated into the default branch
- Complete history (commits and comments) is preserved on GitHub
- **Resolve conflicts** if necessary before merging
- **Check status checks**: all must be passing
- Respect branch protection rules (minimum number of approvals, etc.)

**Merge types (prefer rebase):**
- **Rebase and merge** (PREFERRED): reapplies commits linearly, keeps clean history
- **Squash and merge**: consolidates commits into one (use for messy commit history)
- **Merge commit**: keeps all commits (avoid when possible, creates merge commits)

### 6. Delete the Branch

- After merging, **delete the branch** immediately
- Indicates that the work is complete
- Avoids confusion with obsolete branches
- Keeps the repository organized

**Don't worry:**
- PR history and commits are not lost
- You can restore the branch if needed
- You can revert the PR if something goes wrong

**Command (after merge):**
```bash
git branch -D branch-name
git push origin --delete branch-name
```

## Visual Flow Summary

```
1. master → 2. create branch → 3. make commits → 4. open PR
                                                    ↓
6. delete branch ← 5. merge approved ← review and adjustments
```

## Important Tips

- Keep PRs small and focused (easier to review)
- **Update your branch with rebase regularly** to avoid large conflicts:
  ```bash
  git fetch origin
  git rebase origin/master
  ```
- **Prefer rebase over merge** for cleaner linear history
- Review your own code before requesting review from others
