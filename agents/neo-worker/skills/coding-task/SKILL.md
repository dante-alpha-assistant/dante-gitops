# Coding Task Skill

This skill handles dispatched coding tasks from the task-dispatcher. Follow these steps exactly.

## Detection

This skill applies when you receive a message containing:
- A JSON block with `task_id`, `title`, `description`, `type: "coding"`
- The header `## Task Assigned:`
- Instructions to update task status via Supabase API

## Workflow

### 1. Parse the Task

Extract from the JSON payload:
- `task_id` — unique task identifier
- `title` — short task title
- `description` — full task description with requirements
- `type` — should be "coding"
- `priority` — urgency level
- `repo` — target repository (if provided)
- `branch` — branch convention (if provided)

If `repo` is not in the payload, identify it from the description. Known repos:
- `dante-alpha-assistant/queue-dashboard` — Dashboard frontend + Express backend
- `dante-alpha-assistant/task-dispatcher` — Task dispatcher service  
- `dante-alpha-assistant/dante-gitops` — Kubernetes manifests + ArgoCD configs

### 2. Clone or Update the Repo

**CRITICAL: Never edit files without cloning first. Never clone to /tmp/.**

```bash
cd /root/.openclaw/workspace

# If repo dir exists, pull latest
if [ -d "<repo-name>" ]; then
  cd <repo-name>
  git fetch origin
  git checkout main
  git pull origin main
else
  git clone https://github.com/<org>/<repo-name>.git
  cd <repo-name>
fi
```

### 3. Create a Feature Branch

Branch naming: `feat/<short-kebab-description>` or `fix/<short-kebab-description>`

```bash
git checkout -b feat/<branch-name>
```

### 4. Make the Changes

- Read existing code to understand patterns before editing
- Make the changes described in the task description
- Follow existing code style and conventions
- Test if possible (run linters, type checks, etc.)

### 5. Commit and Push

```bash
git add -A
git commit -m "<type>: <description> (task: <task_id_short>)"
git push origin <branch-name>
```

Commit message types: `feat`, `fix`, `refactor`, `chore`, `docs`

### 6. Create a Pull Request

Use the `gh` CLI:

```bash
gh pr create \
  --repo <org>/<repo-name> \
  --title "<task title>" \
  --body "## Summary
<describe what changed and why>

## Changes
- <list key changes>

## Task
- ID: <task_id>
- Priority: <priority>
- Dispatched by: <dispatched_by>

## Testing
<describe how to verify>" \
  --base main
```

### 7. Update Task Status

**On success** — use the curl command provided in the dispatch message. Replace the summary with what you actually did, including the PR number.

**On failure** — use the failure curl command. Include the specific error message.

## Multi-Repo Tasks

If the task spans multiple repos:
1. Work on each repo sequentially
2. Create a PR in each repo
3. Reference related PRs in each PR description
4. Report all PRs in the task status update

## Error Handling

- If `git push` fails with auth errors: check GH_TOKEN is set, report failure
- If the repo doesn't exist: report failure with the repo URL tried
- If you can't understand the requirements: report failure with what's unclear
- If tests fail: include test output in the failure report

## Environment

- `GH_TOKEN` — GitHub personal access token (available as env var)
- Git user: `Neo <dante-neo-assistant@proton.me>`
- Workspace: `/root/.openclaw/workspace/`
- kubectl: `/tools/kubectl`
