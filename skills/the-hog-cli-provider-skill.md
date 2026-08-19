---
name: thehog-cli
description: "The Hog CLI for running The Hog API workflows from local agents."
author: "The Hog"
license: "Apache-2.0"
argument-hint: "<command> [args] | install cli|mcp"
allowed-tools: "Read Bash"
metadata:
  openclaw:
    requires:
      bins:
        - thehog
---

# The Hog CLI

## Prerequisites: Install the CLI

This skill drives the `thehog` binary. **You must verify the CLI is installed before invoking any command from this skill.** If it is missing, install it first:

1. Download a release archive from GitHub Releases:
   ```bash
   https://github.com/The-Hog/the-hog-cli/releases
   ```
2. Extract the archive and put `thehog` on `$PATH`.
3. Verify: `thehog --version`

If `--version` reports "command not found" after install, the install step did not put the binary on `$PATH`. Do not proceed with skill commands until verification succeeds.

Public API reference for The Hog.

## Command Reference

**companies** — Manage companies

- `thehog companies search` — Async company discovery. Returns operationId; poll GET /api/operations/:id.

**deep-research** — Manage deep research

- `thehog deep-research` — Start a deep research job with a prompt and JSON Schema.

**enrichments** — Manage enrichments

- `thehog enrichments get` — Check the status of an enrichment request and retrieve the result once it completes.
- `thehog enrichments submit` — Enrich one contact or a batch of contacts with requested fields such as verified email, phone, and signals.

**monitor_events** — Manage monitor events

- `thehog monitor-events <id>` — List events found by a monitor.

**monitors** — Manage monitors

- `thehog monitors create` — Create a recurring monitor that runs on a schedule and stores matching events for later review.
- `thehog monitors delete` — Delete a monitor by ID.
- `thehog monitors get` — Retrieve a monitor by ID.
- `thehog monitors list` — List monitors for your organization.
- `thehog monitors update` — Update a monitor configuration, schedule, or status.

**operations** — Manage operations

- `thehog operations <id>` — Check the status of a background operation and retrieve its result once it completes.

**people** — Manage people

- `thehog people search` — Async people discovery. Returns operationId; poll GET /api/operations/:id.

**platform** — Manage platform

- `thehog platform batch-scrape-web-pages` — Queue a batch scrape job and poll the returned operation URL for per-URL results.
- `thehog platform crawl-web-site` — Queue an asynchronous website crawl. Poll the returned operation URL for status and results.
- `thehog platform find-linked-in-companies` — Find LinkedIn company URLs from website domains or URLs.
- `thehog platform get-instagram-post` — Fetch details for an Instagram post or reel URL.
- `thehog platform get-instagram-profile` — Fetch public profile details for an Instagram username.
- `thehog platform get-linked-in-company` — Fetch public company details for a LinkedIn slug or URL.
- `thehog platform get-linked-in-profile` — Fetch public profile details for a LinkedIn username.
- `thehog platform get-tik-tok-profile` — Fetch public profile details and recent videos for a TikTok username.
- `thehog platform list-instagram-followers` — Fetch followers for an Instagram username.
- `thehog platform list-instagram-following` — Fetch accounts followed by an Instagram username.
- `thehog platform list-instagram-post-comments` — Fetch comments for an Instagram post or reel URL.
- `thehog platform list-instagram-posts` — Fetch recent posts for an Instagram username.
- `thehog platform list-linked-in-company-posts` — Fetch recent posts for a LinkedIn company page.
- `thehog platform list-linked-in-post-comments` — Fetch comments for one or more LinkedIn post URLs.
- `thehog platform list-linked-in-post-reactions` — Fetch reactions for one or more LinkedIn post URLs.
- `thehog platform list-linked-in-profile-comments` — Fetch recent LinkedIn posts a public profile has commented on.
- `thehog platform list-linked-in-profile-posts` — Fetch recent posts for a LinkedIn profile.
- `thehog platform list-linked-in-profile-reactions` — Fetch recent LinkedIn posts a public profile has reacted to.
- `thehog platform scrape-web-page` — Fetch a web page and return its readable text content in a stable response shape.
- `thehog platform search-linked-in-keyword-posts` — Search LinkedIn posts by keyword.
- `thehog platform search-web` — Search the web and return normalized results in a stable response shape.

**search_resource** — Manage search resource

- `thehog search-resource get-search-result` — Check the status of a search and retrieve the result once it completes.
- `thehog search-resource list-searches` — List previous searches for your organization.
- `thehog search-resource submit-search` — Run a search across supported web and social sources.


### Finding the right command

When you know what you want to do but not which command does it, ask the CLI directly:

```bash
thehog which "<capability in your own words>"
```

`which` resolves a natural-language capability query to the best matching command from this CLI's curated feature index. Exit code `0` means at least one match; exit code `2` means no confident match — fall back to `--help` or use a narrower query.

## Auth Setup
Run `thehog auth setup` to print the URL and steps for getting a key (add `--launch` to open the URL). Then set:

```bash
export THE_HOG_ACCESS_KEY="<your-key>"
export THE_HOG_SECRET_KEY="<your-secret>"
```

Or persist credentials in `~/.config/thehog/config.toml`.

Run `thehog doctor` to verify setup.

## Agent Mode

Add `--agent` to any command. Expands to: `--json --compact --no-input --no-color --yes`.

- **Pipeable** — JSON on stdout, errors on stderr
- **Filterable** — `--select` keeps a subset of fields. Dotted paths descend into nested structures; arrays traverse element-wise. Critical for keeping context small on verbose APIs:

  ```bash
  thehog companies --query example-value --agent --select id,name,status
  ```
- **Previewable** — `--dry-run` shows the request without sending
- **Offline-friendly** — sync/search commands can use the local SQLite store when available
- **Non-interactive** — never prompts, every input is a flag
- **Explicit retries** — use `--idempotent` only when an already-existing create should count as success, and `--ignore-missing` only when a missing delete target should count as success

### Response envelope

Commands that read from the local store or the API wrap output in a provenance envelope:

```json
{
  "meta": {"source": "live" | "local", "synced_at": "...", "reason": "..."},
  "results": <data>
}
```

Parse `.results` for data and `.meta.source` to know whether it's live or local. A human-readable `N results (live)` summary is printed to stderr only when stdout is a terminal AND no machine-format flag (`--json`, `--csv`, `--compact`, `--quiet`, `--plain`, `--select`) is set — piped/agent consumers and explicit-format runs get pure JSON on stdout.

## Agent Feedback

When you (or the agent) notice something off about this CLI, record it:

```
thehog feedback "the --since flag is inclusive but docs say exclusive"
thehog feedback --stdin < notes.txt
thehog feedback list --json --limit 10
```

Entries are stored locally at `~/.local/share/thehog/feedback.jsonl`. They are never POSTed unless `THEHOG_FEEDBACK_ENDPOINT` is set AND either `--send` is passed or `THEHOG_FEEDBACK_AUTO_SEND=true`. Default behavior is local-only.

Write what *surprised* you, not a bug report. Short, specific, one line: that is the part that compounds.

## Output Delivery

Every command accepts `--deliver <sink>`. The output goes to the named sink in addition to (or instead of) stdout, so agents can route command results without hand-piping. Three sinks are supported:

| Sink | Effect |
|------|--------|
| `stdout` | Default; write to stdout only |
| `file:<path>` | Atomically write output to `<path>` (tmp + rename) |
| `webhook:<url>` | POST the output body to the URL (`application/json` or `application/x-ndjson` when `--compact`) |

Unknown schemes are refused with a structured error naming the supported set. Webhook failures return non-zero and log the URL + HTTP status on stderr.

## Named Profiles

A profile is a saved set of flag values, reused across invocations. Use it when a scheduled agent calls the same command every run with the same configuration - HeyGen's "Beacon" pattern.

```
thehog profile save briefing --json
thehog --profile briefing companies --query example-value
thehog profile list --json
thehog profile show briefing
thehog profile delete briefing --yes
```

Explicit flags always win over profile values; profile values win over defaults. `agent-context` lists all available profiles under `available_profiles` so introspecting agents discover them at runtime.

## Async Jobs

For endpoints that submit long-running work, the generator detects the submit-then-poll pattern (a `job_id`/`task_id`/`operation_id` field in the response plus a sibling status endpoint) and wires up three extra flags on the submitting command:

| Flag | Purpose |
|------|---------|
| `--wait` | Block until the job reaches a terminal status instead of returning the job ID immediately |
| `--wait-timeout` | Maximum wait duration (default 10m, 0 means no timeout) |
| `--wait-interval` | Initial poll interval (default 2s; grows with exponential backoff up to 30s) |

Use async submission without `--wait` when you want to fire-and-forget; use `--wait` when you want one command to return the finished artifact.

## Exit Codes

| Code | Meaning |
|------|---------|
| 0 | Success |
| 2 | Usage error (wrong arguments) |
| 3 | Resource not found |
| 4 | Authentication required |
| 5 | API error (upstream issue) |
| 7 | Rate limited (wait and retry) |
| 10 | Config error |

## Argument Parsing

Parse `$ARGUMENTS`:

1. **Empty, `help`, or `--help`** → show `thehog --help` output
2. **Starts with `install`** → ends with `mcp` → MCP installation; otherwise → see Prerequisites above
3. **Anything else** → Direct Use (execute as CLI command with `--agent`)

## MCP Server Installation

Install `thehog-mcp` from the same GitHub release archive as `thehog`, then register it:

```bash
claude mcp add thehog-cli \
  -e THE_HOG_ACCESS_KEY="$THE_HOG_ACCESS_KEY" \
  -e THE_HOG_SECRET_KEY="$THE_HOG_SECRET_KEY" \
  -- thehog-mcp
```

Verify: `claude mcp list`

## Direct Use

1. Check if installed: `which thehog`
   If not found, offer to install (see Prerequisites at the top of this skill).
2. Match the user query to the best command from the Unique Capabilities and Command Reference above.
3. Execute with the `--agent` flag:
   ```bash
   thehog <command> [subcommand] [args] --agent
   ```
4. If ambiguous, drill into subcommand help: `thehog <command> --help`.
