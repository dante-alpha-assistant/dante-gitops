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
- `description` — full requirements
- `repo` — target repository URL (from Coding Task section)

Known repos:
- `dante-alpha-assistant/queue-dashboard` — Dashboard frontend + Express backend
- `dante-alpha-assistant/task-dispatcher` — Task dispatcher service
- `dante-alpha-assistant/dante-gitops` — K8s manifests + ArgoCD configs

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
  git clone https://x-access-token:${GH_TOKEN}@github.com/<org>/<repo-name>.git
  cd <repo-name>
fi
```

## Step 4: Create a Feature Branch
Branch naming: `feat/<short-kebab-description>` or `fix/<short-kebab-description>`
```bash
git checkout -b feat/<branch-name>
```

## Step 5: Make the Changes
- Read existing code first to understand patterns
- Make focused changes matching the task description
- Follow existing code style
- Run build/lint if available (`npm run build`, `npm run lint`)

## Step 6: Commit and Push
```bash
git add -A
git commit -m "<type>: <description>"
git push -u origin <branch-name>
```
Types: feat, fix, refactor, chore, docs

## Step 7: Create a PR
```bash
gh pr create --repo <org>/<repo-name> \
  --title "<task title>" \
  --body "## Summary
<what changed and why>

## Changes
- <list changes>

## Task
- ID: <task_id>
- Priority: <priority>" \
  --base main
```

## Step 8: Update Task Status
Use the curl commands from the dispatch message.
- On success: include PR number and summary of changes
- On failure: include specific error message

## Multi-Repo Tasks
Work on each repo sequentially. Create a PR in each. Reference related PRs in descriptions.

## Error Handling
- Auth errors → check GH_TOKEN, report failure
- Repo not found → report failure with URL
- Unclear requirements → report failure with what's unclear
- Build fails → include output in failure report

## Common Mistakes to AVOID
- ❌ Editing files without cloning the repo first
- ❌ Cloning to /tmp/ — use workspace
- ❌ Committing to main directly
- ❌ Forgetting git pull before branching
- ❌ Not running build/tests before committing
- ❌ Generic commit messages
- ❌ Leaving task in_progress — ALWAYS update status
