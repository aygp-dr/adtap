# adtap — Meta-Prompt for Coding Agents

## What This Document Is

This is a **meta-prompt**: a specification that a coding agent (Claude Code, Cursor, Codex, etc.) should consume to build `adtap`, a CLI tool for interacting with the Google Ads API. The agent reading this should produce a Go CLI binary, not a Python wrapper. No MCP adapters. No dependency on google-ads-api-developer-assistant.

## Identity

- **Name**: `adtap`
- **One-liner**: Tap the Google Ads API from your terminal.
- **Org**: `aygp-dr` (GitHub)
- **Language**: Go
- **License**: Apache-2.0
- **Sibling tools**: `bd` (beads — issue tracker), `gt` (Gas Town — multi-agent workspace manager), `cprr` (conjecture tracker)

## Design Lineage

`adtap` inherits CLI conventions from two sources:

### 1. clig.dev (Command Line Interface Guidelines)

Read https://clig.dev/ and treat it as normative. Key points the agent MUST follow:

- Human-first output by default; `--json` flag for machine consumption
- Stderr for messages, stdout for data
- Exit codes: 0 success, 1 general error, 2 usage error
- `--help` on every subcommand, always useful
- `--verbose` / `--quiet` flags globally
- Config files in XDG paths (`~/.config/adtap/`)
- No interactive prompts in pipelines; detect TTY
- Color output respects `NO_COLOR` env var
- Version info via `adtap version` and `adtap --version`

### 2. bd/gt idiom conventions

The agent MUST study these patterns from sibling tools and replicate them:

| Pattern | bd/gt Example | adtap Equivalent |
|---|---|---|
| Health check | `bd doctor`, `gt doctor` | `adtap doctor` |
| Agent context dump | `bd prime`, `gt prime` | `adtap prime` |
| Quick start guide | `bd quickstart` | `adtap quickstart` |
| Onboarding for agents | `bd onboard` | `adtap onboard` |
| Init workspace | `bd init`, `gt init` | `adtap init` |
| JSON machine output | `bd list --json` | `adtap query --json` |
| Config management | `bd config`, `gt config` | `adtap config` |
| Quiet/verbose | `bd -q`, `bd -v` | `adtap -q`, `adtap -v` |
| Export formats | `bd export --format jsonl` | `adtap export -f jsonl` |
| Actor identity | `bd --actor` | `adtap --actor` (for audit) |

## Subcommand Structure

```
adtap
├── init              # Initialize adtap in current directory
├── quickstart        # Print getting-started guide (like bd quickstart)
├── onboard           # Emit AGENTS.md snippet for coding agents
├── prime             # Dump full context for AI agent consumption
├── doctor            # Verify credentials, connectivity, API version
├── version           # Print version, Go version, build info
│
├── config            # Manage configuration
│   ├── show          # Display current config (redacted secrets)
│   ├── set           # Set a config value
│   ├── path          # Print config file location
│   └── accounts      # List configured accounts
│
├── auth              # Authentication workflows
│   ├── login         # OAuth2 flow (opens browser, receives callback)
│   ├── status        # Show current auth state
│   ├── token         # Print current access token (for debugging)
│   └── revoke        # Revoke stored credentials
│
├── accounts          # Account discovery and inspection
│   ├── list          # List accessible accounts under MCC
│   ├── show          # Show account details
│   └── tree          # Show MCC hierarchy
│
├── query             # Execute GAQL queries (core command)
│   [ACCOUNT_ID] [GAQL]   # One-shot mode
│   --repl                 # Interactive REPL mode
│   -f, --format csv|json|jsonl|table|parquet
│   -o, --output FILE      # Write to file instead of stdout
│   --limit N              # Override LIMIT (default: 100)
│   --dry-run              # Parse and validate GAQL without executing
│   --explain              # Show query plan / resource usage estimate
│
├── inspect           # Schema exploration (no query execution)
│   ├── resources     # List all GAQL resources (campaign, ad_group, etc.)
│   ├── fields        # List fields for a resource
│   ├── metrics       # List available metrics
│   ├── segments      # List available segments
│   └── enums         # Show enum values for a field
│
├── export            # Batch export workflows
│   -f, --format jsonl|csv|parquet
│   --since DATE      # Incremental export since date
│   --template NAME   # Use predefined query template
│   --dest DIR        # Output directory
│
├── templates         # Manage predefined GAQL query templates
│   ├── list          # List available templates
│   ├── show          # Print template GAQL
│   ├── add           # Add custom template
│   └── run           # Execute a template
│
└── completions       # Shell completion scripts
    ├── bash
    ├── zsh
    └── fish
```

## What `adtap init` Must Do

When a user runs `adtap init` in a project directory:

1. Create `.adtap/` directory with:
   - `config.toml` — account configuration (developer token, customer IDs, OAuth client refs)
   - `templates/` — directory for custom GAQL templates
   - `exports/` — default export output directory
   - `.gitignore` — ignoring secrets, token cache, exports

2. Generate `AGENTS.md` (or append to existing) with a section like:

```markdown
## adtap — Google Ads API Access

This project has Google Ads API access configured via `adtap`.

### Quick Reference for Agents

- `adtap doctor` — verify credentials and API connectivity
- `adtap inspect resources` — list queryable GAQL resources
- `adtap inspect fields campaign` — show fields for a resource
- `adtap query <ACCOUNT_ID> "SELECT campaign.name, metrics.clicks FROM campaign"` — run a GAQL query
- `adtap templates list` — see predefined queries
- `adtap prime` — get full context dump for AI consumption

### Conventions

- All data output goes to stdout; messages to stderr
- Use `--json` for machine-parseable output
- Use `--dry-run` to validate queries without executing
- Exports land in `.adtap/exports/` by default
- GAQL reference: https://developers.google.com/google-ads/api/fields/v18/overview

### Credentials

Credentials are stored in `~/.config/adtap/credentials.json` (NOT in the repo).
The developer token and OAuth client are configured per-account in `.adtap/config.toml`.
```

3. Print a quickstart checklist to stderr:
   - [ ] Google Ads developer token obtained
   - [ ] Google Cloud project with Ads API enabled
   - [ ] OAuth2 client ID created (desktop application type)
   - [ ] Run `adtap auth login` to authenticate
   - [ ] Run `adtap doctor` to verify setup

## What `adtap prime` Must Emit

`prime` is consumed by coding agents at the start of a session. It must emit (to stdout) a self-contained context block:

```
TOOL: adtap v0.1.0
PURPOSE: Google Ads API CLI — query, inspect, export
AUTH_STATUS: authenticated | expired | not_configured
ACTIVE_ACCOUNT: 123-456-7890 (My Account Name)
API_VERSION: v18
CONFIG_PATH: /path/to/.adtap/config.toml
CREDENTIALS: configured (token expires 2026-02-27T12:00:00Z)

AVAILABLE_COMMANDS:
  adtap query <ACCT> <GAQL>     Execute GAQL query
  adtap inspect resources        List queryable resources
  adtap inspect fields <RES>     Fields for a resource
  adtap templates list           Predefined query templates
  adtap export --template <T>    Batch export

GAQL_QUICK_REFERENCE:
  SELECT campaign.name, metrics.clicks FROM campaign WHERE segments.date DURING LAST_7_DAYS
  SELECT ad_group.name, metrics.impressions FROM ad_group ORDER BY metrics.impressions DESC LIMIT 10

TEMPLATES_AVAILABLE:
  campaigns-overview    Campaign performance last 30 days
  keywords-top          Top keywords by clicks
  ad-performance        Ad-level metrics with creative info
  budget-utilization    Budget spend vs. allocation
  search-terms          Search terms report

NOTES:
  - GAQL is NOT SQL. No JOINs, no subqueries. One resource per query.
  - Fields must be compatible (check `adtap inspect fields <resource>`)
  - Metrics require a date segment (implicit or explicit)
  - Rate limits: ~15,000 requests/day at basic access level
```

## What `adtap quickstart` Must Print

Model this on `bd quickstart` output. Conversational, scannable, no fluff:

```
adtap - Tap the Google Ads API

PREREQUISITES
  1. Google Ads manager account (for developer token)
  2. Google Cloud project with "Google Ads API" enabled
  3. OAuth2 desktop client credentials (client_id + client_secret)
  4. Developer token from API Center in Google Ads UI

  Full setup guide: https://developers.google.com/google-ads/api/docs/get-started

SETUP
  adtap init                          Initialize in current directory
  adtap config set developer-token <TOKEN>
  adtap config set client-id <ID>
  adtap config set client-secret <SECRET>
  adtap config set customer-id <CUSTOMER_ID>
  adtap auth login                    Opens browser for OAuth2 flow
  adtap doctor                        Verify everything works

EXPLORE
  adtap accounts list                 See accessible accounts
  adtap inspect resources             What can you query?
  adtap inspect fields campaign       What fields does campaign have?

QUERY
  adtap query 1234567890 "SELECT campaign.name FROM campaign"
  adtap query 1234567890 --repl       Interactive GAQL prompt
  adtap query 1234567890 "SELECT campaign.name FROM campaign" -f jsonl -o campaigns.jsonl

EXPORT
  adtap export --template campaigns-overview -f parquet --dest ./data/
  adtap export --template keywords-top --since 2026-01-01

PIPELINE USAGE
  adtap query 1234567890 "SELECT ..." -f jsonl | jq '.campaign.name'
  adtap export -f parquet --dest /stage/ && snowsql -f load_ads.sql
```

## Configuration File Format

`.adtap/config.toml`:

```toml
api_version = "v18"
default_account = "main"

[accounts.main]
developer_token = ""          # from Google Ads API Center
customer_id = "123-456-7890"  # target account
login_customer_id = ""        # MCC account (if using MCC)

[accounts.main.oauth]
client_id = ""                # from Google Cloud Console
client_secret = ""            # from Google Cloud Console
# refresh_token stored in ~/.config/adtap/credentials.json, NOT here

[defaults]
format = "table"              # table|json|jsonl|csv|parquet
limit = 100
output_dir = ".adtap/exports"

[templates_dir]
path = ".adtap/templates"
```

## GAQL Template Format

Templates live as `.gaql` files in `.adtap/templates/`:

```sql
-- name: campaigns-overview
-- description: Campaign performance summary for last 30 days
-- requires: customer_id
SELECT
  campaign.id,
  campaign.name,
  campaign.status,
  campaign_budget.amount_micros,
  metrics.impressions,
  metrics.clicks,
  metrics.ctr,
  metrics.average_cpc,
  metrics.cost_micros,
  metrics.conversions,
  metrics.cost_per_conversion
FROM campaign
WHERE segments.date DURING LAST_30_DAYS
  AND campaign.status != 'REMOVED'
ORDER BY metrics.cost_micros DESC
```

## Google Ads API Interaction — How adtap Talks to Google

The agent building this MUST understand:

### Authentication Flow
1. User provides developer token (static, from Google Ads UI)
2. User provides OAuth2 client_id + client_secret (from Google Cloud Console)
3. `adtap auth login` opens browser → Google consent screen → redirect to localhost callback → receives authorization code → exchanges for refresh_token + access_token
4. refresh_token persists in `~/.config/adtap/credentials.json`
5. access_token auto-refreshes from refresh_token on each API call

### API Call Pattern
- REST endpoint: `https://googleads.googleapis.com/v18/customers/{customer_id}/googleAds:searchStream`
- Method: POST
- Headers: `Authorization: Bearer <access_token>`, `developer-token: <dev_token>`, `login-customer-id: <mcc_id>` (if applicable)
- Body: `{"query": "<GAQL string>"}`
- Response: streaming JSON (newline-delimited result batches)

The agent SHOULD use the Google Ads API REST interface directly (gRPC-JSON transcoding), NOT the Python/Java/etc client libraries. This keeps the Go binary dependency-free from Google's generated client stubs.

### Key Constraint: GAQL Is Not SQL
- One resource per query (no JOINs)
- Fields must be from compatible field groups
- `FROM` clause determines the resource
- Metrics require date context
- Resource metadata available at: `https://googleads.googleapis.com/v18/googleAdsFields/{field_name}`
- Full field list: `https://googleads.googleapis.com/v18/googleAdsFields:search` with query body

### Field Metadata Introspection (for `adtap inspect`)
```
POST https://googleads.googleapis.com/v18/googleAdsFields:search
Body: {"query": "SELECT name, category, data_type, selectable, filterable, sortable, selectable_with FROM google_ads_fields WHERE name LIKE 'campaign%'"}
```

This is how `adtap inspect` works WITHOUT needing a local schema dump or MCP adapter. The Google Ads API itself exposes its own metadata as a queryable resource.

## Dependencies and Build

### Go Dependencies (expected)
- `github.com/spf13/cobra` — subcommand structure
- `github.com/BurntSushi/toml` — config parsing
- `github.com/chzyer/readline` or `github.com/peterh/liner` — REPL
- `golang.org/x/oauth2` — OAuth2 flow
- `github.com/fatih/color` — terminal color (respecting NO_COLOR)
- `github.com/olekukonko/tablewriter` — table output
- Standard library `net/http` for API calls (no google-api-go-client needed)
- `github.com/xitongsys/parquet-go` — parquet output (optional, can defer)

### Build
- `go build -o adtap ./cmd/adtap`
- Cross-compile: `GOOS=freebsd GOARCH=amd64` (hydra), `GOOS=darwin GOARCH=arm64` (mini)
- Version injection via `-ldflags`

## Project Layout

```
adtap/
├── cmd/
│   └── adtap/
│       └── main.go
├── internal/
│   ├── auth/          # OAuth2 flow, token management
│   ├── config/        # TOML config loading, XDG paths
│   ├── api/           # Google Ads REST client (searchStream, fields)
│   ├── gaql/          # GAQL parsing, validation, template rendering
│   ├── output/        # Formatters: table, json, jsonl, csv, parquet
│   ├── repl/          # Interactive GAQL prompt
│   └── inspect/       # Field metadata queries
├── templates/         # Built-in GAQL templates (embedded via go:embed)
├── docs/
│   ├── AGENTS.md      # Template for generated AGENTS.md
│   └── QUICKSTART.md  # Content for quickstart command
├── .goreleaser.yml
├── go.mod
├── go.sum
├── LICENSE
├── README.md
└── CLAUDE.md          # Instructions for Claude Code working on this repo
```

## CLAUDE.md for the adtap Repo

The agent should generate a `CLAUDE.md` that future agents working IN the adtap repo will read:

```markdown
# CLAUDE.md — adtap

## What Is This
adtap is a Go CLI for querying the Google Ads API via GAQL.
It follows clig.dev conventions and bd/gt CLI idioms.

## Build & Test
  go build -o adtap ./cmd/adtap
  go test ./...
  go vet ./...

## Key Design Rules
- stdout is for data, stderr is for messages
- --json flag on every command that outputs structured data
- No interactive prompts unless TTY detected
- Config in TOML, credentials separate from config
- REST API calls only, no gRPC stubs, no Python client libraries
- Templates are .gaql files with SQL-style comments for metadata
- `adtap prime` must always work, even if not authenticated (report status)

## Working on Commands
- All commands are cobra commands in cmd/adtap/
- Shared flags defined in root command
- API client initialized lazily (don't fail on import if no credentials)

## Testing
- Unit tests for GAQL parsing, config loading, output formatting
- Integration tests gated behind ADTAP_INTEGRATION=1 env var
- Use httptest for mocking Google Ads API responses
- Test fixtures in testdata/
```

## Non-Goals (Things adtap Does NOT Do)

- **No campaign management** — adtap is read-only. No creating/updating/deleting campaigns, ads, or budgets. That's a different tool with different risk profile.
- **No MCP server** — adtap is a CLI, not a protocol adapter. Agents use it via shell execution.
- **No Snowflake integration** — adtap exports files (jsonl, parquet). Loading into Snowflake is a separate pipeline concern.
- **No ML/analytics** — adtap extracts. Analysis happens downstream.
- **No Google Ads UI automation** — API only.
- **No billing management** — read-only queries against reporting and metadata endpoints only.

## Success Criteria

An agent consuming this prompt should produce a working binary where:

1. `adtap version` prints version info
2. `adtap quickstart` prints the getting-started guide
3. `adtap init` creates `.adtap/` directory and `AGENTS.md`
4. `adtap onboard` emits an AGENTS.md snippet to stdout
5. `adtap prime` emits structured context (works even without auth)
6. `adtap doctor` checks config, credentials, and API reachability
7. `adtap config show` displays configuration (redacted)
8. `adtap auth login` performs OAuth2 desktop flow
9. `adtap inspect resources` lists GAQL resources from API metadata
10. `adtap inspect fields campaign` shows fields for a resource
11. `adtap query <ACCT> <GAQL>` executes a query and prints results
12. `adtap query <ACCT> --repl` opens interactive GAQL prompt
13. `adtap templates list` shows built-in templates
14. `adtap export --template campaigns-overview -f jsonl` produces output

Items 1–6 should work in the first development pass. Items 7–14 are the full scope.

## Downstream Context

`adtap` is the extraction tier in a planned pipeline:

```
adtap (extract) → staging (jsonl/parquet) → Snowflake (persist) → dbt (transform) → ML (analyze)
```

The repos `adwall` and `adbudget` under `aygp-dr` are related work in the same problem space. The agent should not duplicate their scope but should ensure `adtap`'s export formats are compatible with whatever those tools consume.
