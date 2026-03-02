# coding-task — Dispatched Coding Task Skill

## When to Use

This skill applies when you receive a message containing:
- A JSON block with `task_id`, `title`, `description`
- The header `## Task Assigned:`
- A `## Coding Task` section with repo/branch info

**If you see these markers, follow this skill step by step.**

## Step 1: Parse the Task

Extract from the JSON payload:
- `task_id` — unique identifier (used for branch naming and status updates)
- `title` — short task title (used for PR title)
- `description` — full requirements
- `repo` — target repository (from Coding Task section or description)
- `type` — task type (coding, ops, etc.)
- `priority` — urgency level

Known repos (all under `dante-alpha-assistant`):
| Repo | Path in workspace | Description |
|------|-------------------|-------------|
| `queue-dashboard` | `queue-dashboard/` | Dashboard frontend (Vite) + Express backend |
| `task-dispatcher` | `task-dispatcher/` | Task dispatcher service (Node.js) |
| `dante-gitops` | `dante-gitops/` | K8s manifests + ArgoCD configs |

## Step 2: Setup Git

```bash
git config --global user.email "dante-neo-assistant@proton.me"
git config --global user.name "Neo"
```

## Step 3: Prepare the Repo

**CRITICAL: NEVER edit files without preparing the repo first. NEVER clone to /tmp/.**

```bash
cd /root/.openclaw/workspace

# If repo exists in workspace, update it
if [ -d "<repo-name>" ]; then
  cd <repo-name>
  git fetch origin
  git checkout main
  git pull origin main
else
  # Clone fresh — use GH_TOKEN for auth
  git clone https://x-access-token:${GH_TOKEN}@github.com/dante-alpha-assistant/<repo-name>.git
  cd <repo-name>
fi
```

**For multi-repo tasks:** Repeat for each repo involved.

## Step 4: Create a Feature Branch

```bash
# Use task_id first 8 chars for uniqueness
git checkout -b feat/<short-kebab-description>
```

Branch naming conventions:
- `feat/<description>` — new features
- `fix/<description>` — bug fixes
- `chore/<description>` — maintenance, config changes
- `refactor/<description>` — code restructuring

## Step 5: Understand Before Editing

Before making changes:
1. **Read the relevant files** — `ls`, `find`, `cat` to understand structure
2. **Check existing patterns** — follow the codebase's conventions
3. **Verify file paths exist** — never edit a file without confirming it's there

## Step 6: Make the Changes

- Keep changes focused on the task requirements
- Follow existing code style and patterns
- Add comments for non-obvious logic
- Handle errors appropriately

## Step 7: Build and Test

```bash
# For queue-dashboard
cd queue-dashboard && npm install && npm run build

# For task-dispatcher
cd task-dispatcher && npm install && node -e "require('./index.js')" 2>&1 | head -5

# For dante-gitops — validate YAML
# (no build step, just check syntax)
```

**Always verify your changes compile/build before committing.**

## Step 8: Commit and Push

```bash
git add -A
git commit -m "<type>: <description>"
git push -u origin HEAD
```

Commit message format: `<type>: <description>` 
Types: `feat`, `fix`, `refactor`, `chore`, `docs`

## Step 9: Create a Pull Request

```bash
gh pr create \
  --repo dante-alpha-assistant/<repo-name> \
  --title "<task title>" \
  --body "## Summary
<What changed and why>

## Changes
- <List key changes>

## Task
- **ID:** <task_id>
- **Priority:** <priority>
- **Dispatched by:** <dispatched_by>

## Testing
<How to verify the changes>" \
  --base main
```

**For multi-repo tasks:** Create a PR in each repo, cross-reference them.

## Step 10: Update Task Status

Use the curl commands provided in the dispatch message.

**On success:** Include in `result.summary`:
- PR number and repo (e.g., "PR #29 on queue-dashboard")
- Brief description of what changed
- List of files modified

**On failure:** Include in `error`:
- Specific error message
- What step failed
- What you tried

## Error Handling

| Error | Action |
|-------|--------|
| `git push` auth failure | Verify `GH_TOKEN`, try `git remote set-url origin https://x-access-token:${GH_TOKEN}@github.com/...` |
| Repo not found | Report failure with the URL attempted |
| Build fails | Include build output in failure report |
| Unclear requirements | Report failure explaining what's ambiguous |
| Merge conflicts | Report failure, list conflicting files |

## Common Mistakes to AVOID

- ❌ Editing files without cloning/pulling the repo first
- ❌ Cloning repos to `/tmp/` — always use workspace
- ❌ Committing directly to `main`
- ❌ Forgetting `git pull` before branching (causes conflicts)
- ❌ Not building/testing before committing
- ❌ Generic commit messages — be specific
- ❌ Leaving task as `in_progress` — ALWAYS update status to done or failed
- ❌ Assuming file paths — always `ls`/`find` to verify
