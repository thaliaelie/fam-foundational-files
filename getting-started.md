# Getting Started with fam-os

## What is fam-os?

fam-os is the CS team's shared operating system — a Git repo that gives every FAM a standardized way to manage customers with Claude Code. It provides:

- **Customer context** (CLAUDE.md files) that give Claude instant knowledge about any customer
- **Skills** (reusable workflows) for common tasks like sync prep, cost investigation, and CostFormation
- **Rules** (domain expertise) that auto-load based on what you're working on
- **Templates** for onboarding new customers with consistent structure

## Setup

### 1. Clone the repo

```bash
git clone <repo-url> ~/fam-os
cd ~/fam-os
```

### 2. Verify Claude Code works

```bash
claude
```

You should see the root CLAUDE.md load. Verify by asking: "What are the git conventions for this repo?"

### 3. Install the CloudZero MCP plugin

The Cost Analyst MCP plugin enables Claude to query CloudZero data directly:

```bash
claude plugin marketplace add cost-analyst
```

Verify it's installed:
```bash
claude plugin list
```

### 4. Read the system

Start with these files to understand how everything fits together:
- `CLAUDE.md` — Team-wide instructions (you're inheriting these)
- `customers/_example-customer/CLAUDE.md` — See what a fully populated customer looks like
- `customers/_example-customer/joint-success-plan.md` — See a filled-out JSP
- `customers/_example-customer/onboarding-project-plan.md` — See a filled-out OPP

### 5. Try the skills

Navigate to the example customer and test the key workflows:

```bash
cd customers/_example-customer
```

Then in Claude Code:
- `/customer-sync-prep` — Generates a sync brief
- `/customer-health-check` — Scores adoption health

### 6. Set up your first real customer

If you're inheriting existing customers, their folders should already exist. If not:

```bash
# Use the onboarding skill to create a new customer workspace
/onboarding-kickoff
```

Or manually:
```bash
mkdir customers/<customer-name>
cp templates/customer-claude-md.md customers/<customer-name>/CLAUDE.md
cp templates/joint-success-plan.md customers/<customer-name>/joint-success-plan.md
cp templates/onboarding-project-plan.md customers/<customer-name>/onboarding-project-plan.md
```

Then edit the CLAUDE.md to fill in the customer's details.

### 7. Validate your setup

Run a sync prep on your first customer:
```
/customer-sync-prep
```

If it produces useful output, you're good to go.

## Daily Workflow

1. **Before a sync**: Navigate to the customer folder, run `/customer-sync-prep`
2. **During a sync**: Reference the brief, note action items
3. **After a sync**: Update CLAUDE.md (Recent Activity, Open Items), commit
4. **For CostFormation work**: Use `/costformation-builder` to create changes safely
5. **For cost questions**: Use `/cost-investigation` to drill into anomalies

## Key Skills

| Skill | When to Use |
|-------|------------|
| `/customer-sync-prep` | Before any customer meeting |
| `/cost-investigation` | When a customer reports unexpected costs |
| `/costformation-builder` | When building or modifying dimensions |
| `/telemetry-builder` | When setting up allocation or metric streams |
| `/customer-health-check` | For QBR prep or periodic reviews |
| `/handoff-generator` | When transferring a customer to another FAM |
| `/onboarding-kickoff` | When setting up a new customer |
| `/jsp-tracker` | When reviewing JSP progress |
| `/allocation-validation` | After uploading allocation telemetry |
| `/telemetry-validation` | After uploading any telemetry |

## Getting Help

- Check `docs/` for guides on specific topics
- Look at `customers/_example-customer/` for reference implementations
- Read `.claude/rules/` for domain-specific reference material
- Ask in the team Slack channel for questions about conventions
