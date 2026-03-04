# FAM Operating System — Team Instructions

## How This Repo Works
- Each customer has a folder under `customers/` with a standardized `CLAUDE.md`
- Always read a customer's `CLAUDE.md` before doing any work in their folder
- Use `/customer-sync-prep`, `/cost-investigation`, etc. for standardized workflows
- Commit customer context updates after every work session

## Python / AWS Lambda
- Prefer native Python libraries over third-party packages to avoid zip/Docker deployments
- Use the `logging` module, not `print()` statements
- Include explicit timeouts on all network calls
- Use type hints for function signatures
- **Always build Lambda packages using Docker with the matching Lambda runtime:**
  - Match the Python version to your Lambda runtime (check in AWS Console or CloudWatch logs)
  - Use `--platform linux/amd64` on Apple Silicon Macs
  ```bash
  # Check your Lambda's Python version first, then use matching image
  docker run --rm --platform linux/amd64 --entrypoint pip \
    -v "$(pwd)/build/package:/asset" \
    public.ecr.aws/lambda/python:3.11 \
    install --target /asset <dependencies>
  ```

## Code Style
- Keep functions focused and single-purpose
- Write clear error messages that include relevant context (IDs, names, etc.)

## Git Conventions
- Branch: `<initials>/<customer>/<description>` (e.g., `rb/abrigo/bam-allocation`)
- Commit: `[<scope>] <description>` (e.g., `[abrigo] Add BAM SQL allocation dimension`)
- Customer work: commit directly to main (no PR required)
- Shared changes (skills, rules, templates): PR required, one reviewer

## AWS & Cost Analysis
- When asked about AWS pricing, billing, or cost attribution, always consult the actual AWS Cost and Usage Report (CUR) documentation or specific data sources first rather than providing generic pricing information.

## Data Analysis
- For data analysis tasks, ask clarifying questions about the intended audience (internal vs customer-facing) and specific use case before starting analysis.

## API & Data Queries
- When working with CloudZero API data, always offer to display results in both table format (for quick scanning) and raw JSON (for verification).

## Domain-Specific Rules

| Domain | Trigger | Rule File |
|--------|---------|-----------|
| CostFormation | `**/*.cz.yaml` | `.claude/rules/costformation.md` |
| CloudZero API | Context: CloudZero API requests | `.claude/rules/cloudzero-api.md` |
| AnyCost | `**/anycost/**` | `.claude/rules/anycost.md` |
| Kubernetes | `**/k8s/**` | `.claude/rules/k8s.md` |
| Customer Work | `customers/**` | `.claude/rules/customer-context.md` |
| Engagement | Context: sync prep, escalation, cadence | `.claude/rules/customer-engagement.md` |

## REQUIRED
- **NEVER use `rm -rf`** — use macOS trash instead
- **NEVER commit API keys, credentials, or .env files**
- **NEVER modify a customer's CostFormation file directly** — create change sub-folders
