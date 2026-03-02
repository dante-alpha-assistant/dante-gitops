# coding-task — Dispatched Coding Task Skill

## When to Use

This skill applies when you receive a message containing:
- A JSON block with `task_id`, `title`, `type: "coding"`
- The header `## Task Assigned:`
- A `## Coding Task` section with repo/branch info

**If you see these markers, follow this skill exactly.**

## Step 1: Parse the Task

Extract from the JSON payload:
- `task_id` — unique identifier
- `title` — short title
- `description` — full requirements
- `repo` — target repository (check the Coding Task section too)
- `priority` — urgency level

Known repos (all under `dante-alpha-assistant`):
- `queue-dashboard` — Dashboard frontend + Express backend (tasks.dante.id)
- `task-dispatcher` — Task dispatcher service
- `dante-gitops` — Kubernetes manifests + ArgoCD configs

If the task description mentions a repo not listed here, use the full GitHub URL from the description.

## Step 2: Setup Git

```bash
git config --global user.email "dante-neo-assistant@proton.me"
git config --global user.name "Neo"
```

## Step 3: Clone or Update the Repo

**CRITICAL: NEVER edit files without cloning first. NEVER clone to /tmp/.**

```bash
cd /root/.openclaw/workspace

# If repo dir exists, pull latest
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

## Step 5: Understand Before Editing

Before making any changes:
1. Read the relevant files to understand existing patterns
2. Check the project structure (`ls`, `find`)
3. Look at recent commits for style conventions (`git log --oneline -10`)
4. If the repo has `package.json`, check scripts (`cat package.json | grep -A5 scripts`)

## Step 6: Make the Changes

- Make focused changes matching the task description
- Follow existing code style and conventions
- Keep changes minimal — don't refactor unrelated code
- Add comments for complex logic

## Step 7: Build and Test

Before committing, verify your changes:

```bash
# For Node.js projects
npm install  # only if package.json changed
npm run build  # if build script exists
npm run lint   # if lint script exists

# For K8s manifests (dante-gitops)
# Validate YAML syntax
cat <file>.yaml | python3 -c "import sys,yaml; yaml.safe_load(sys.stdin)"
```

If build/lint fails, fix the issues before committing.

## Step 8: Commit and Push

```bash
git add -A
git commit -m "<type>: <description>"
git push -u origin <branch-name>
```

Commit types: `feat`, `fix`, `refactor`, `chore`, `docs`

## Step 9: Create a Pull Request

```bash
gh pr create \
  --repo dante-alpha-assistant/<repo-name> \
  --title "<task title>" \
  --body "## Summary
<what changed and why>

## Changes
- <list key changes>

## Task
- ID: <task_id>
- Priority: <priority>
- Dispatched by: <dispatched_by>

## Testing
<how to verify the changes>" \
  --base main
```

## Step 10: Update Task Status

Use the curl commands from the dispatch message:
- **On success**: Replace `DESCRIBE WHAT YOU DID` with a real summary including the PR number and repo
- **On failure**: Replace `DESCRIBE WHAT WENT WRONG` with the specific error

Example success summary: `"Created feat/add-search for queue-dashboard. Added search filter to task list. PR #29 on dante-alpha-assistant/queue-dashboard."`

## Multi-Repo Tasks

If the task spans multiple repos:
1. Work on each repo sequentially
2. Create a PR in each repo
3. Reference related PRs in each PR description
4. Report ALL PRs in the task status update

## Error Handling

- **Auth errors** → Verify GH_TOKEN: `echo $GH_TOKEN | head -c4`. Report failure if missing.
- **Repo not found** → Double-check the org/repo name. Report failure with URL tried.
- **Merge conflicts** → Pull latest main, rebase, resolve. If complex, report failure.
- **Unclear requirements** → Report failure explaining what's ambiguous.
- **Build fails** → Include build output in failure report.

## Common Mistakes to AVOID

- ❌ Editing files without cloning the repo first
- ❌ Cloning to `/tmp/` — always use workspace `/root/.openclaw/workspace/`
- ❌ Committing directly to `main`
- ❌ Forgetting `git pull` before creating a branch
- ❌ Not running build/tests before committing
- ❌ Generic commit messages — be specific
- ❌ Leaving task `in_progress` — ALWAYS update status when done
- ❌ Forgetting to include PR number in the status update
