# AGENTS.md - ADTAP Agent Instructions

## Project Overview

ADTAP (Ads API Tools And Patterns) is a toolkit for working with Google Ads API v23.

## HARD RULES (Non-Negotiable)

### Git Staging: NEVER use broad add commands

```bash
# FORBIDDEN - will add sensitive files, secrets, API data, PII
git add -A
git add .
git add --all

# REQUIRED - always add specific files by name
git add file1.py file2.md
git add path/to/specific/file.py
```

**This is a core mandate across every agent, every session, every operation.**

### Never Commit Secrets

- API keys, tokens, credentials
- `.env` files (any variation)
- `credentials.json` or service account files
- Private keys (`.pem`, `.key`)

### Never Commit Live Data

- Data pulled from Google Ads API must NEVER be committed
- Use `data/` or `tmp/` directories (gitignored) for API responses
- Sanitize any examples before committing

### Never Commit PII

- Customer IDs should be anonymized in examples
- Email addresses, names, phone numbers
- Any personally identifiable information from API responses

## PII Auditor Recommendation

**This project should include a PII auditor/scrubber as core functionality.**

Recommended implementation:
1. Pre-commit hook to scan for PII patterns
2. Scrubber utility for sanitizing API responses
3. Audit log for any PII that was detected and scrubbed

Patterns to detect:
- Email addresses
- Phone numbers
- Customer IDs (replace with synthetic IDs)
- Names
- Addresses
- Credit card patterns

## Workflow: Stacked Diffs

Use stacked diffs for feature development:

```bash
# Create feature branch
git checkout -b feat/feature-name

# Work in small, reviewable commits
git add specific-file.py
git commit -m "feat(scope): description"

# Stack changes logically
git add next-file.py
git commit -m "feat(scope): next logical step"

# Rebase to keep history clean
git rebase -i main
```

## Git Notes for Context

Use git notes to document conjectures, hypotheses, and deviations:

```bash
# Add note to most recent commit
git notes add -m "Hypothesis: This approach may need revision when v24 releases"

# Add note to specific commit
git notes add <commit-sha> -m "Deviation: Using non-standard pattern because..."

# View notes
git log --show-notes
```

Note categories:
- `Hypothesis:` - Unverified assumptions
- `Conjecture:` - Theoretical approaches
- `Deviation:` - Intentional departures from standard patterns
- `TODO:` - Known technical debt

## Key Concepts

### Google Ads API v23

- **API Version**: v23 (January 2026 release)
- **Transport**: gRPC with Protocol Buffers
- **Authentication**: OAuth 2.0 + Developer Token

### Entity Hierarchy

```
Customer (Manager Account)
└── Campaign
    └── AdGroup
        └── AdGroupAd
            └── Ad (with Assets)
```

### Core Services

| Service | Purpose |
|---------|---------|
| GoogleAdsService | Unified search/mutate operations |
| CampaignService | Campaign CRUD |
| AdGroupService | AdGroup CRUD |
| AdGroupAdService | Ad CRUD |
| CampaignBudgetService | Budget management |

### Key Patterns

1. **Mutate Operations**: Use `GoogleAdsService.Mutate` for batch operations with temporary resource names
2. **Search**: Use GAQL (Google Ads Query Language) with `GoogleAdsService.Search` or `SearchStream`
3. **Partial Failure**: Enable `partial_failure=true` for batch operations that should continue on individual errors

## Environment Variables

Required environment variables (see `.env.template`):

```bash
GOOGLE_APPLICATION_CREDENTIALS  # Path to service account JSON
GOOGLE_PROJECT_ID               # GCP project ID
GOOGLE_ADS_DEVELOPER_TOKEN      # 22-char developer token
```

## Developer Token Access Levels

| Level | Description |
|-------|-------------|
| Test Account | Test accounts only, no production access |
| Basic | Standard production access after approval |
| Standard | Full capabilities, higher rate limits |

Apply at: https://ads.google.com/aw/apicenter

## File Conventions

- `docs/*.org` - Documentation in Org-mode format
- `.env.template` - Environment template (never commit actual `.env`)
- `vendor/` - Git submodules (reference implementations)
- Mermaid diagrams embedded in org files

## Common Tasks

### Adding New Diagrams

Add Mermaid diagrams to `docs/google-ads-api-v23-diagrams.org`:

```org
#+begin_src mermaid :file images/diagram-name.png
graph TD
    A --> B
#+end_src
```

### Updating References

Add links to `docs/references.org` under appropriate sections.

## API Resources

- Reference: https://developers.google.com/google-ads/api/reference/rpc/v23/overview
- Protos: https://github.com/googleapis/googleapis/tree/master/google/ads/googleads
- MCP Server: https://github.com/googleads/google-ads-mcp

## Status Enums (Common)

All major entities use these status values:
- `UNSPECIFIED` - Not set
- `UNKNOWN` - Unknown value (error state)
- `ENABLED` - Active
- `PAUSED` - Paused (not serving)
- `REMOVED` - Soft deleted

## Do Not

- Use `git add .` or `git add -A`
- Commit `.env` or credentials files
- Commit live API data
- Commit PII
- Use deprecated API versions (v10-v19 are sunset)
- Hardcode customer IDs or tokens

## Landing the Plane (Session Completion)

**When ending a work session**, you MUST complete ALL steps below. Work is NOT complete until `git push` succeeds.

**MANDATORY WORKFLOW:**

1. **File issues for remaining work** - Create issues for anything that needs follow-up
2. **Run quality gates** (if code changed) - Tests, linters, builds
3. **Update issue status** - Close finished work, update in-progress items
4. **PUSH TO REMOTE** - This is MANDATORY:
   ```bash
   git pull --rebase
   bd sync
   git push
   git status  # MUST show "up to date with origin"
   ```
5. **Clean up** - Clear stashes, prune remote branches
6. **Verify** - All changes committed AND pushed
7. **Hand off** - Provide context for next session

**CRITICAL RULES:**
- Work is NOT complete until `git push` succeeds
- NEVER stop before pushing - that leaves work stranded locally
- NEVER say "ready to push when you are" - YOU must push
- If push fails, resolve and retry until it succeeds
