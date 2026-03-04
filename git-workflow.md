# Git Workflow

## Branches

| Type | Convention | Example |
|------|-----------|---------|
| Customer work | `<initials>/<customer>/<description>` | `rb/abrigo/bam-allocation` |
| Shared changes | `<initials>/shared/<description>` | `rb/shared/sync-prep-skill` |

## Commits

**Format**: `[<scope>] <imperative description>`

| Scope | When | Example |
|-------|------|---------|
| Customer name | Work in a customer folder | `[abrigo] Add BAM SQL allocation dimension` |
| `shared` | Skills, rules, templates | `[shared] Update sync-prep skill with JSP integration` |
| `docs` | Documentation | `[docs] Add new customer setup guide` |
| `infra` | Repo config, .gitignore | `[infra] Update .gitignore for CSV exclusion` |

## When to Commit

- **After every customer work session** — your CLAUDE.md updates should be committed
- **After updating open items** — so other FAMs can see current state
- **After building/modifying CostFormation** — the change subfolder with YAML + comments
- **After running sync prep** — if you updated Recent Activity

## What Requires a PR

| Change Type | PR Required? | Reviewers |
|------------|-------------|-----------|
| Customer CLAUDE.md updates | No | — |
| Customer JSP/OPP updates | No | — |
| Customer CostFormation changes | No | — |
| New/modified skills | Yes | 1 reviewer |
| New/modified rules | Yes | 1 reviewer |
| Template changes | Yes | 1 reviewer |
| Documentation changes | Yes | 1 reviewer |
| Root CLAUDE.md changes | Yes | 1 reviewer |

**Rationale**: Customer work is time-sensitive and FAM-owned — PRs would slow things down. Shared infrastructure (skills, rules, templates) affects the whole team, so a review ensures quality.

## Commit Directly to Main

For customer work, commit directly to main:

```bash
git add customers/<customer>/
git commit -m "[<customer>] <description>"
git push
```

## PR Workflow for Shared Changes

```bash
# Create a branch
git checkout -b rb/shared/improve-sync-prep

# Make changes
# ...

# Commit
git add .claude/skills/customer-sync-prep/
git commit -m "[shared] Add JSP progress to sync prep brief"

# Push and create PR
git push -u origin rb/shared/improve-sync-prep
gh pr create --title "Add JSP progress to sync prep brief" \
  --body "Adds JSP goal status to the sync prep output"
```

## Handling Conflicts

Since multiple FAMs may update customers simultaneously:

1. **Pull before working**: `git pull` at the start of each session
2. **Commit frequently**: Small, focused commits reduce conflict scope
3. **Customer folders rarely conflict**: Each FAM typically owns different customers
4. **If conflicts occur**: Resolve in favor of the most recent update (check timestamps in Recent Activity)

## What NOT to Commit

- `.env` files or API keys
- Raw customer data (CSV, XLSX files)
- Large binary files
- Temporary analysis scripts (use `.gitignore`)
- Personal notes (use your own notes outside the repo)
