# coding-task — Dispatched Coding Task Skill

## When to Use
This skill applies when you receive a message containing:
- A JSON block with `task_id`, `title`, `description`
- The header `## Task Assigned:`
- A `## Coding Task` section with repo/branch info

**If you see these markers, follow this skill exactly.**

## Step 1: Parse the Task
Extract from the JSON payload:
- `task_id` — unique identifier
- `title` — short title
- `description` — full requirements (may be null — use title as guide)
- `acceptance_criteria` — specific criteria to meet (may be null)
- `repo` — target repository (from Coding Task section, or infer from context)

Known repos (all under `dante-alpha-assistant`):
- `queue-dashboard` — Dashboard frontend (React/Vite) + Express backend
- `task-dispatcher` — Task dispatcher service (Node.js)
- `dante-gitops` — K8s manifests + ArgoCD configs

## Step 2: Setup Git
```bash
git config --global user.email "dante-neo-assistant@proton.me"
git config --global user.name "Neo"
```

## Step 3: Clone or Update the Repo
**CRITICAL: NEVER edit files without cloning first. NEVER clone to /tmp/.**

```bash
cd /root/.openclaw/workspace
if [ -d "<repo-name>" ]; then
  cd <repo-name>
  git fetch origin
  git checkout main
  git pull origin main
else
  git clone https://x-access-token:${GH_TOKEN}@github.com/dante-alpha-assistant/<repo-name>.git
  cd <repo-name>
fi
```

## Step 4: Create a Feature Branch
Branch naming: `feat/<short-kebab-description>` or `fix/<short-kebab-description>`
```bash
git checkout -b feat/<branch-name>
```

## Step 5: Understand Before Changing
- Read key files to understand the project structure
- Check existing patterns, conventions, and code style
- Look at recent commits: `git log --oneline -10`
- If the project has a README, read it

## Step 6: Make the Changes
- Keep changes focused on the task requirements
- Follow existing code patterns and conventions
- Add comments for non-obvious logic
- Handle errors properly — no silent failures
- If acceptance_criteria are provided, verify each one is met

## Step 7: Verify Your Work
```bash
# If the project has a package.json:
npm install  # or npm ci
npm run build 2>&1  # Check for build errors
npm run lint 2>&1   # Check for lint errors (if available)
npm test 2>&1       # Run tests (if available)
```
**Don't skip verification.** Broken code wastes everyone's time and gets failed in QA.

## Step 8: Commit and Push
```bash
git add -A
git commit -m "<type>: <concise description>

Task: <task_id>"
git push -u origin <branch-name>
```
Commit types: `feat`, `fix`, `refactor`, `chore`, `docs`, `test`

## Step 9: Create a PR
```bash
gh pr create --repo dante-alpha-assistant/<repo-name> \
  --title "<type>: <task title>" \
  --body "## Summary
<What this PR does and why>

## Changes
- <Change 1>
- <Change 2>

## Testing
<How to verify this works — steps or commands>

## Task
- ID: <task_id>
- Priority: <priority>" \
  --base main
```

## Step 10: Update Task Status
Use the curl commands from the dispatch message:
- **On success:** Set status to `qa_testing`, include the PR URL and a summary
- **On failure:** Set status to `failed`, include specific error message

**NEVER leave the task without updating status.**

## Multi-Repo Tasks
If a task requires changes across multiple repos:
1. Work on each repo sequentially
2. Create a PR in each repo
3. Use consistent branch naming across repos
4. Reference all PRs in the task status update

## Error Handling
| Error | Action |
|-------|--------|
| Auth/token error | Check GH_TOKEN, report failure |
| Repo not found | Report failure with URL |
| Build fails | Include build output in failure report |
| Unclear requirements | Report failure explaining what's unclear |
| Merge conflicts | Resolve if simple, report as blocker if not |

## Common Mistakes to AVOID
- ❌ Editing files without cloning the repo first
- ❌ Cloning to /tmp/ — always use workspace
- ❌ Committing to main directly
- ❌ Forgetting `git pull` before branching (stale base)
- ❌ Not running build/lint before pushing
- ❌ Generic commit messages like "update files"
- ❌ Leaving task status unchanged — ALWAYS update it
- ❌ Huge PRs with unrelated changes — stay focused
