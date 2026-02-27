# CLI Flag Compatibility Matrix

Reference document for adtap CLI flag interactions. Input for test harness (`adtap-2x5`).

**Per:** L7 Engineering Review NEW-01

---

## 1. Global Flags

These flags apply to all commands.

| Flag | Short | Type | Default | Description |
|------|-------|------|---------|-------------|
| `--json` | | bool | false | Machine-readable JSON output |
| `--verbose` | `-v` | bool | false | Verbose output to stderr |
| `--quiet` | `-q` | bool | false | Suppress non-error output |
| `--actor` | | string | "" | Actor identity for audit trail |
| `--config` | | path | auto | Config file path override |

### Global Flag Conflicts

| Flag A | Flag B | Result | Exit Code |
|--------|--------|--------|-----------|
| `--verbose` | `--quiet` | ERROR: mutually exclusive | 2 |
| `--json` | `--format X` | ERROR: conflicting format spec | 2 |

---

## 2. Command: `query`

Execute GAQL queries against Google Ads API.

### Positional Arguments

| Position | Name | Required | Description |
|----------|------|----------|-------------|
| 1 | ACCOUNT_ID | No* | 10-digit customer ID |
| 2 | GAQL | No* | GAQL query string |

*Required unless `--repl` is used.

### Flags

| Flag | Short | Type | Default | Description |
|------|-------|------|---------|-------------|
| `--format` | `-f` | enum | table | Output format: table, csv, json, jsonl, parquet |
| `--output` | `-o` | path | stdout | Write output to file |
| `--limit` | | int | 100 | Override LIMIT clause |
| `--dry-run` | | bool | false | Validate query without executing |
| `--explain` | | bool | false | Show query plan / resource estimate |
| `--repl` | | bool | false | Interactive GAQL prompt |

### Query Flag Conflicts

| Flag/Arg A | Flag/Arg B | Result | Exit Code |
|------------|------------|--------|-----------|
| `--repl` | ACCOUNT_ID (positional) | ERROR: repl is interactive, no positional args | 2 |
| `--repl` | GAQL (positional) | ERROR: repl is interactive, no positional args | 2 |
| `--repl` | `--output FILE` | ERROR: repl writes interactively to stdout | 2 |
| `--dry-run` | `--output FILE` | WARNING: no data to write (proceed) | 0 |
| `--json` | `--format csv` | ERROR: conflicting format specification | 2 |
| `--json` | `--format jsonl` | ERROR: conflicting format specification | 2 |
| `--json` | `--format parquet` | ERROR: conflicting format specification | 2 |
| `--format parquet` | stdout (no -o) | ERROR: parquet requires file output | 2 |

### Query Requirements

| Scenario | Requirement | Exit Code on Violation |
|----------|-------------|------------------------|
| One-shot mode | ACCOUNT_ID required | 2 |
| One-shot mode | GAQL required | 2 |
| REPL mode | No positional args | 2 |

---

## 3. Command: `export`

Batch export workflows with predefined templates.

### Flags

| Flag | Short | Type | Default | Description |
|------|-------|------|---------|-------------|
| `--format` | `-f` | enum | jsonl | Output format: jsonl, csv, parquet |
| `--since` | | date | - | Incremental export since date |
| `--template` | | string | - | Predefined query template name |
| `--dest` | | path | .adtap/exports | Output directory |

### Export Flag Conflicts

| Flag A | Flag B | Result | Exit Code |
|--------|--------|--------|-----------|
| `--since` | template without date support | ERROR: template must support incremental | 4 |
| `--format parquet` | streaming output | ERROR: parquet requires file output | 2 |

### Export Requirements

| Scenario | Requirement | Exit Code on Violation |
|----------|-------------|------------------------|
| Export | `--template` or GAQL required | 2 |
| Incremental | Template must support `--since` | 4 |

---

## 4. Command: `auth`

Authentication subcommands.

### Subcommand: `auth login`

| Flag | Type | Default | Description |
|------|------|---------|-------------|
| `--browser` | bool | auto | Open browser for OAuth flow |
| `--no-browser` | bool | false | Print URL, don't open browser |

### Subcommand: `auth status`

| Flag | Type | Default | Description |
|------|------|---------|-------------|
| `--json` | bool | false | JSON output |

### Subcommand: `auth revoke`

| Flag | Type | Default | Description |
|------|------|---------|-------------|
| `--confirm` | bool | false | Skip confirmation prompt |

### Auth Flag Conflicts

| Subcommand | Flag A | Flag B | Result | Exit Code |
|------------|--------|--------|--------|-----------|
| login | `--browser` | `--no-browser` | ERROR: mutually exclusive | 2 |

---

## 5. Command: `config`

Configuration management.

### Subcommand: `config show`

| Flag | Type | Default | Description |
|------|------|---------|-------------|
| `--json` | bool | false | JSON output |
| `--reveal` | bool | false | Show unredacted secrets |

### Subcommand: `config set`

| Positional | Required | Description |
|------------|----------|-------------|
| KEY | yes | Configuration key |
| VALUE | yes | Configuration value |

### Config Flag Conflicts

None specific to config commands.

---

## 6. Command: `inspect`

Schema exploration (no query execution).

### Subcommands

| Subcommand | Description | Arguments |
|------------|-------------|-----------|
| `resources` | List all GAQL resources | none |
| `fields` | List fields for a resource | RESOURCE (required) |
| `metrics` | List available metrics | RESOURCE (optional) |
| `segments` | List available segments | RESOURCE (optional) |
| `enums` | Show enum values | FIELD (required) |

### Shared Flags

| Flag | Short | Type | Default | Description |
|------|-------|------|---------|-------------|
| `--json` | | bool | false | JSON output |
| `--filter` | | string | - | Filter by pattern |

### Inspect Requirements

| Subcommand | Requirement | Exit Code on Violation |
|------------|-------------|------------------------|
| `fields` | RESOURCE argument required | 2 |
| `enums` | FIELD argument required | 2 |

---

## 7. Command: `accounts`

Account discovery and inspection.

### Subcommands

| Subcommand | Description | Arguments |
|------------|-------------|-----------|
| `list` | List accessible accounts | none |
| `show` | Show account details | ACCOUNT_ID (optional) |
| `tree` | Show MCC hierarchy | none |

### Shared Flags

| Flag | Type | Default | Description |
|------|------|---------|-------------|
| `--json` | bool | false | JSON output |

---

## 8. Command: `templates`

Manage predefined GAQL query templates.

### Subcommands

| Subcommand | Description | Arguments |
|------------|-------------|-----------|
| `list` | List available templates | none |
| `show` | Print template GAQL | TEMPLATE (required) |
| `add` | Add custom template | NAME FILE |
| `run` | Execute a template | TEMPLATE (required) |

### Templates `run` Flags

| Flag | Short | Type | Default | Description |
|------|-------|------|---------|-------------|
| `--account` | `-a` | string | default | Account to run against |
| `--format` | `-f` | enum | table | Output format |
| `--output` | `-o` | path | stdout | Output file |

### Template Requirements

| Subcommand | Requirement | Exit Code on Violation |
|------------|-------------|------------------------|
| `show` | TEMPLATE argument required | 2 |
| `run` | TEMPLATE argument required | 2 |
| `add` | NAME and FILE arguments required | 2 |

---

## 9. TTY Detection Behavior

| Condition | Behavior |
|-----------|----------|
| stdout is TTY | Color output enabled, table format default |
| stdout is pipe | No color, streaming-friendly output |
| stdin is TTY + `--repl` | Interactive mode allowed |
| stdin is pipe + `--repl` | ERROR: repl requires TTY |
| `NO_COLOR` env set | Color disabled regardless of TTY |

---

## 10. Exit Codes

| Code | Category | Description |
|------|----------|-------------|
| 0 | SUCCESS | Command completed successfully |
| 1 | GENERAL_ERROR | Unspecified error |
| 2 | USAGE_ERROR | Invalid command usage / flag conflict |
| 3 | AUTH_ERROR | Authentication failed |
| 4 | API_ERROR | Google Ads API error |
| 5 | CONFIG_ERROR | Configuration invalid |
| 6 | IO_ERROR | File/network error |
| 7 | VALIDATION_ERROR | Input validation failed |

---

## 11. Test Matrix for adtap-2x5

### Global Flag Tests

```
TEST: global_verbose_quiet_conflict
  INPUT:  adtap -v -q version
  EXPECT: exit 2, stderr contains "mutually exclusive"

TEST: global_json_format_conflict
  INPUT:  adtap --json query --format csv 123 "SELECT..."
  EXPECT: exit 2, stderr contains "conflicting format"
```

### Query Command Tests

```
TEST: query_repl_with_positional_account
  INPUT:  adtap query --repl 1234567890
  EXPECT: exit 2, stderr contains "repl is interactive"

TEST: query_repl_with_output_file
  INPUT:  adtap query --repl -o output.json
  EXPECT: exit 2, stderr contains "repl writes to stdout"

TEST: query_dryrun_with_output_warning
  INPUT:  adtap query --dry-run -o output.json 123 "SELECT..."
  EXPECT: exit 0, stderr contains "warning"

TEST: query_parquet_requires_file
  INPUT:  adtap query --format parquet 123 "SELECT..."
  EXPECT: exit 2, stderr contains "parquet requires file"

TEST: query_missing_account
  INPUT:  adtap query "SELECT campaign.id FROM campaign"
  EXPECT: exit 2, stderr contains "account ID required"

TEST: query_missing_gaql
  INPUT:  adtap query 1234567890
  EXPECT: exit 2, stderr contains "GAQL required"
```

### Auth Command Tests

```
TEST: auth_login_browser_conflict
  INPUT:  adtap auth login --browser --no-browser
  EXPECT: exit 2, stderr contains "mutually exclusive"

TEST: auth_repl_non_tty
  INPUT:  echo "" | adtap query --repl
  EXPECT: exit 2, stderr contains "requires TTY"
```

### Export Command Tests

```
TEST: export_since_unsupported_template
  INPUT:  adtap export --template simple-query --since 2026-01-01
  EXPECT: exit 4, stderr contains "template must support"

TEST: export_missing_template
  INPUT:  adtap export -f jsonl
  EXPECT: exit 2, stderr contains "template required"
```

### Valid Combination Tests

```
TEST: valid_query_all_options
  INPUT:  adtap query 1234567890 "SELECT..." -f jsonl -o out.jsonl --limit 50
  EXPECT: exit 0 (or API error, not usage error)

TEST: valid_verbose_with_json
  INPUT:  adtap -v --json query 1234567890 "SELECT..."
  EXPECT: exit 0, verbose to stderr, json to stdout

TEST: valid_quiet_mode
  INPUT:  adtap -q query 1234567890 "SELECT..."
  EXPECT: exit 0, minimal stderr
```

---

## 12. Compatibility Summary Table

Quick reference for all flag conflicts:

| Command | Flag Pair | Conflict Type |
|---------|-----------|---------------|
| (global) | `-v` + `-q` | Mutual exclusion |
| (global) | `--json` + `--format` | Semantic overlap |
| query | `--repl` + positional args | Mode conflict |
| query | `--repl` + `-o FILE` | Mode conflict |
| query | `--format parquet` + stdout | Technical requirement |
| auth login | `--browser` + `--no-browser` | Mutual exclusion |
| export | `--since` + incompatible template | Feature dependency |

---

*Generated for adtap CLI v0.1.0-alpha / Google Ads API v23*
