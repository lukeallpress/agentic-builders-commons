---
name: pp-sdpc
description: "SDPC Resource Registry 1.0 in one CLI: every read surface, a local mirror for jurisdiction, renewal, and inventory questions, and TEC's registry write steps (each needs explicit user OK). Trigger phrases: `does Kahoot have a DPA in Idaho`, `check our app inventory against SDPC`, `what changed in the SDPC registry this week`, `which DPAs expire this semester`, `export the district's approved apps list`, `use sdpc`, `run sdpc`."
author: "ryangravette-bsd"
license: "Apache-2.0"
argument-hint: "<command> [args] | install cli|mcp"
allowed-tools: "Read Bash"
metadata:
  openclaw:
    requires:
      bins:
        - sdpc-pp-cli
---

# SDPC Resource Registry — Printing Press CLI

*Shared by Ryan Gravette, Boise School District, on the Agentic Builders skills shelf. This is the
agent skill generated with the CLI by the Printing Press, with Boise's local-build notes removed.
The CLI is not yet in the public Printing Press library; the matching cookbook recipe describes how
it was printed so you can print your own. Boise is district 15032 in the Idaho (ID) alliance; the
examples use those values.*

## Prerequisites

`sdpc-pp-cli` must be on PATH. Verify with `sdpc-pp-cli doctor`.


sdpc-pp-cli wraps the Student Data Privacy Consortium's Registry 1.0 read surfaces: the Premium JSON API, the public per-district listings, and the logged-in national search, typeahead, and statewide DPA pages. It also performs TEC's submission workflow: creating Exhibit Es, filing New Requests, assigning TEC, requesting new resources, and setting statuses, each write previewed by default and gated behind --yes. Sync agreements into SQLite, then ask coverage, renewals, changes, gaps, and check questions offline; pipeline fetches statewide request queues live.

## When to Use This CLI

Use this CLI whenever a task involves student data privacy agreements in the SDPC Resource Registry 1.0: checking whether a vendor has a DPA usable in a state, exporting a district's approved-app list, tracking renewals, diffing the registry between syncs, matching an app inventory against the registry, or pulling signed agreement PDFs. It covers sdpc.a4l.org only.

**Reads and writes.** Most commands only read. These commands write to shared registry records
that TEC, SDPC staff, and other districts can see: `exhibit-e create`, `request submit` (and
`request submit --assign-tec`), `request assign`, `request status`, and `request new-resource`.
Each previews by default and needs `--yes` to submit; under `--agent` or `--no-input` a write
without `--yes` is refused. Run a write only after the user has seen the `--dry-run` preview and
given an explicit OK for that one write. `exhibit-e list`, `exhibit-e verify`, `doctor`, and every
lookup, listing, national, alliance, and my-district command are read-only.

## Anti-triggers

Do not use this CLI for:
- Do not use it to upload or delete agreements, change an agreement type (the guardian-release wizard), or upload a resource logo; those stay in the web UI
- Do not run any write command (listed under "Reads and writes") without the user's explicit OK after a preview
- Do not use it for Registry 2.0 at registry.a4l.org
- Do not use it for Common Sense, 1EdTech, or Lightspeed privacy ratings; it only knows SDPC agreements
- Do not use it to manage SDPC user accounts or passwords

## Unique Capabilities

These capabilities aren't available in any other tool for this API.

### Local state that compounds
- **`renewals`** — List agreements expiring inside any window for any district or state, with the 01/01/2099 placeholder dates set aside so they never hide a real expiry. Reads the local agreements mirror: run sync --resources agreements first (Premium membership).

  _Reach for this when an agent needs expiring DPAs beyond the site's fixed 90-day own-district page._

  ```bash
  sdpc-pp-cli renewals --within 180d --state ID --agent
  ```
- **`changes`** — Show what changed in the registry between syncs: new agreements, status flips to Expired, Declined, or Vendor Unresponsive, new Exhibit E files, and edited expiry dates. The first run records a baseline; run sync --resources agreements before each changes run (Premium membership).

  _Reach for this for a weekly what-changed digest instead of re-reading search pages._

  ```bash
  sdpc-pp-cli changes --since 7d --state ID --agent
  ```
- **`pipeline`** — Show how many days each open request has sat in its progress stage, grouped by stage for one state plus your own district. Statewide rows are fetched live for --state and need the Chrome session from auth login --chrome; own-district rows come from the local my-district mirror.

  _Reach for this when an alliance coordinator asks what is stuck in the queue._

  ```bash
  sdpc-pp-cli pipeline --state ID --older-than 30d --agent
  ```

### Jurisdiction intelligence
- **`coverage`** — See, per state, whether a vendor or resource already has a usable agreement, who originated it, whether Exhibit E is offered, and any declines with reasons. Reads the local agreements mirror: run sync --resources agreements first (Premium membership).

  _Reach for this when the question is whether a vendor's DPA can be subscribed to in a specific state._

  ```bash
  sdpc-pp-cli coverage "Kahoot!" --state ID --agent
  ```
- **`check`** — Match a Lightspeed, GoGuardian, or Clever app export against the registry and print MATCH, GAP, or AMBIGUOUS with the best in-state agreement for every row. Reads the local agreements mirror: run sync --resources agreements first (Premium membership).

  _Reach for this when an agent is handed a whole app inventory rather than one vendor._

  ```bash
  sdpc-pp-cli check --file apps.csv --state ID --agent
  ```
- **`gaps`** — List the resources a peer district or the state network has active agreements for that your district does not, with the peer's agreement type and Exhibit E link. Needs local rows: a prior districts listing run for both districts, or a synced agreements mirror, or synced alliance active-dpas pages for state mode.

  _Reach for this when comparing a district against a neighbor or the state's Exhibit E offers._

  ```bash
  sdpc-pp-cli gaps --district-id 15032 --against district:14029 --agent
  ```

### TEC submission workflow
- **`request assign`** — Assign a Progress Admin (default TEC) to one of your district's requests on Manage Agreements and click Update, through your browser session. Picks the row by --data-id, --software-id, or exact --resource name, replays that row's real Update form (every hidden field, the option chosen by label), re-reads the page, and only reports ok when the row shows the requested admin. Asks before writing; --yes for scripts, --dry-run to preview the exact POST.

  _Reach for this after filing a New Request so TEC is notified electronically (TEC SDPC Workflow step 3)._

  ```bash
  sdpc-pp-cli request assign --software-id 444 --yes --agent
  ```

## Discovery Signals

This CLI was generated with browser-observed traffic context.
- Capture coverage: 5 API entries from 36 total network entries
- Protocols: rest_json (75% confidence), html_scrape (55% confidence)
- Auth signals: chrome_mcp_session

## Command Reference

**agreements** — Agreement records from the Premium JSON API (api_read.php); one object per district x resource x agreement

- `sdpc-pp-cli agreements` — List agreement records (Premium API).

**alliance** — Statewide alliance lists served by DataTables server-side paging endpoints (JSON pages of HTML cell fragments; --all walks start/length)

- `sdpc-pp-cli alliance active-dpas` — Active (signed) DPAs in a state alliance ('View Active State DPAs').
- `sdpc-pp-cli alliance all-dpas` — All DPAs (active and inactive) in a state alliance ('View All State DPAs').
- `sdpc-pp-cli alliance negotiations` — Contracts currently in negotiation nationwide ('View Contracts in Negotiation'; 312 records at capture
- `sdpc-pp-cli alliance new-requests` — Statewide new agreement requests ('View Statewide New Requests').
- `sdpc-pp-cli alliance participants` — Participating districts nationwide ('View Participating Districts'; no state parameter).
- `sdpc-pp-cli alliance resources` — Resource catalog with company and website ('View All Resources').

**dictionary** — Per-state data-element dictionary (metadictionary.php), public

- `sdpc-pp-cli dictionary` — Data elements and definitions for a state alliance: table with Family / Name / Description columns (68 rows for UT).

**district** — One district's agreement page inside its state alliance (district_search.php); the link target of every District Name cell on the search pages

- `sdpc-pp-cli district` — Every agreement/request of one district in its alliance. Columns: Logo | Software Name (link search_resource.php?

**districts** — Public per-district custom resource listing pages (no login). Entire table is in one response; DataTables only pages client-side.

- `sdpc-pp-cli districts listing` — Public 'Online Resources' listing for one district: every vetted resource with company, agreement status, agreement type
- `sdpc-pp-cli districts research-listing` — Public Research DPA listing for one district (same layout as `listing`).

**files** — Download signed-agreement, Exhibit E/addendum and supplemental-info PDFs (public; directory listings are 403 but individual files are 200 application/pdf)

- `sdpc-pp-cli files addendum` — Download an Exhibit E / addendum PDF from /addendums/ ({districtID}_{softwareID}_{YYYY-MM-DD}.pdf, e.g.
- `sdpc-pp-cli files agreement` — Download a PDF from /agreements/: signed agreements ({YYYY-MM-DD}_{districtID}_{softwareID}_signed_agreement_file.pdf, .
- `sdpc-pp-cli files supplemental` — Download a supplemental-info PDF from /supplemental_info/ ({YYYY-MM-DD}_{districtID}_{softwareID}_supplemental_info.

**lookup** — Typeahead lookups (Select2 JSON helpers behind the national search forms): resolve a partial name to the ids the search commands need

- `sdpc-pp-cli lookup company` — Look up companies/vendors by partial name.
- `sdpc-pp-cli lookup district` — Look up districts by partial name. id is the numeric districtID (e.g. 15032), text is '<district name>, <ST> <country>'.
- `sdpc-pp-cli lookup resource` — Look up resources (software) by partial name. id is the numeric softwareID (e.g. 444 = IXL), text is the resource name.
- `sdpc-pp-cli lookup vendor` — Look up vendor-member resources by partial name (only vendors that joined the registry as members

**my_district** — The logged-in account's own district pages (districtID must match the login); read-only views of the 'Your District's Agreements' and 'Your Resources' menus

- `sdpc-pp-cli my-district agreements` — Manage Agreements table for your district.
- `sdpc-pp-cli my-district expired` — Your district's expired agreements. Capture returned 'There were no agreements found for your district.
- `sdpc-pp-cli my-district expiring` — Your district's active agreements expiring within 90 days.
- `sdpc-pp-cli my-district export` — Open the logged-in Export District Data to Excel page for your district.
- `sdpc-pp-cli my-district prepared-exhibit-es` — Prepared Exhibit Es awaiting signatures for your district.
- `sdpc-pp-cli my-district resources` — Resources your district has added ('Manage Resources').

**national** — National agreement search (search_national.php result pages; resource named national because search is reserved by the generator). Server-rendered HTML tables parsed by hand-coded table parsers into SearchRow; results can run to thousands of rows

- `sdpc-pp-cli national by-company` — All agreements nationwide for one company. --company-name must be the exact company id string from `lookup company` (e.
- `sdpc-pp-cli national by-district` — All agreements of one district (national view; no State column, no Report/Flag form).
- `sdpc-pp-cli national by-resource` — All districts nationwide with an agreement for one resource (softwareID). EXPENSIVE: IXL (444) returns 1,376 rows / 2.
- `sdpc-pp-cli national by-state` — Agreement listing for one state alliance ('Search <ST> Database').
- `sdpc-pp-cli national by-vendor` — Same table as by-resource but scoped to vendor-member resources (ids from `lookup vendor`)
- `sdpc-pp-cli national custom` — Search agreements by grade level and/or content area (the 'Custom Search' form).

**reference** — Alliance reference pages: company successor (rename/merger) list, agreement type definitions, and the alliance home page

- `sdpc-pp-cli reference agreement-types` — Agreement type definitions for an alliance: the alliance-specific types plus the shared 'Other Agreement Types' table.
- `sdpc-pp-cli reference alliance` — Alliance home page: alliance name, the 'About' blurb, the participating-districts link, and standard-agreement template links.
- `sdpc-pp-cli reference successors` — Company Successor List: which current company now owns former company names (63 rows for ID).

**request** — TEC workflow step 3: add a New Request for a catalog resource and assign TEC as its Progress Admin (writes; asks before submitting, --yes for scripts, --dry-run previews the exact form)

- `sdpc-pp-cli request assign` — Assign a Progress Admin (default TEC) to one of your district's requests on Manage Agreements and click Update; verified from a fresh page read.

**resources** — Resource (software) catalog helpers exposed by the public Digital Resource Request Form (read-only GET steps; never submit the form)

- `sdpc-pp-cli resources catalog` — EXPENSIVE (~1.
- `sdpc-pp-cli resources resolve` — Resolve a softwareID to its 'Company / Vendor Name' and 'Resource' name (e.g. 444 -> IXL Learning, Inc. / IXL).

**session** — Premium API session: exchange username/password headers for the SDPHPSESSID cookie used by agreements

- `sdpc-pp-cli session login` — Mint a Premium API session cookie.

**templates** — State standard DPA / NDPA template PDFs linked from public pages

- `sdpc-pp-cli templates for-state` — FAQ page for one state alliance; contains that state's standard DPA template PDF links under /agreements/ (CA
- `sdpc-pp-cli templates hub` — Static page listing ~40 state standard-agreement PDFs under /agreements/ (e.g. UT_NDPA_v2_2_STANDARD.pdf, IL_NDPA_V1a.


### Finding the right command

When you know what you want to do but not which command does it, ask the CLI directly:

```bash
sdpc-pp-cli which "<capability in your own words>"
```

`which` resolves a natural-language capability query to the best matching command from this CLI's curated feature index. Exit code `0` means at least one match; exit code `2` means no confident match — fall back to `--help` or use a narrower query. `--json` (and other machine formats) keep that exit-2 contract and write `{"matches":[]}` on stdout so agents can inspect the envelope without treating a miss as success.

## Submission workflow (TEC)

These commands walk TEC's "Workflow for Building a District's Repository of DPAs" end to end.
Every one of them is a write against a shared registry record, so each previews by default,
needs `--yes` to submit, and refuses under `--agent` or `--no-input` unless `--yes` is present.
All of them need a signed-in session (see Authentication).

| Step | Command |
|---|---|
| 1. Sign in | `auth login --chrome` |
| 2. Find an Exhibit E to piggyback on | `exhibit-e list --query '"Exact Name"'` |
| 2. Create it | `exhibit-e create --data-id N --yes` |
| 2. Check district info and signature | `exhibit-e verify --resource "Exact Name"` |
| 3. File a New Request | `request submit --resource "Exact Name" --requestor NAME --requestor-email EMAIL` |
| 3. Assign TEC so they are notified | `request assign --data-id N --yes` (or `request submit --assign-tec`) |
| 4. Ask SDPC to add a missing product | `request new-resource --company X --name Y --website URL` |

Two more, outside TEC's four steps:

| Task | Command |
|---|---|
| Mark a product as needing no DPA | `request status --data-id N --approved-no-data --yes` |
| Set any other status, with conditions | `request status --data-id N --status "<label>" --restrictions "..."` |

The Exhibit E search is exact-match. Quote the resource name; the CLI adds the quotes if you
leave them off and reports what it searched as `query_used`. Partial names do not work, case
does not matter, and numbers and punctuation must match the registry exactly, as in
`"Soundation4Education"` or `"i-Ready"`.

An Exhibit E signed in another state does not cover your district. `exhibit-e list` is the only
test that matters; when it returns nothing, go to step 3.

`request status` saves the whole Edit Other Info record, so it replays every field it is not
changing exactly as the page rendered it, and its preview lists each change as old to new.

Not supported, still browser work: the later steps of Change Agreement Type (the guardian
release path), and the logo upload on Request to Add New Resource.

## Recipes

### Does this vendor have a DPA I can use in Idaho

```bash
sdpc-pp-cli coverage "Kahoot!" --state ID --agent
```

Per-state rollup with originating district, agreement type, Exhibit E availability, and expiry from the local mirror.

### Compact agreement rows for an agent

```bash
sdpc-pp-cli agreements --page=0 --state=IL --agent --select dataID,district_name,software_name,company_name,status,date_expired,signed_agreement_file
```

Premium API records narrowed to the fields an agent needs instead of 38 columns per row.

### Match an app inventory export

```bash
sdpc-pp-cli check --file apps.csv --state ID --json
```

Every inventory row gets MATCH, GAP, or AMBIGUOUS with the best in-state agreement.

### Weekly what-changed digest

```bash
sdpc-pp-cli changes --since 7d --state ID --json
```

New agreements, status flips, new Exhibit E files, and expiry edits since the previous sync.

### Parent-facing approved list as CSV

```bash
sdpc-pp-cli districts listing --district-id=15032 --csv
```

Public listing rows as CSV for the district website or board packet.

### Notify TEC about a new request

```bash
sdpc-pp-cli request assign --resource "IXL" --dry-run
sdpc-pp-cli request assign --resource "IXL" --yes --json
```

Preview the Manage Agreements Update form for that row, then submit it with Progress Admin = TEC and confirm from a fresh page read. Needs `auth login --chrome`; --district-id/--state default to the logged-in account's district.

## Auth Setup

Public commands (districts listing, dictionary, templates, files, resources) need no credentials.
Logged-in web commands (lookup, national, district, alliance, my-district, reference, exhibit-e,
request) use an SDPC browser session captured by `press-auth`, the same route the
`sdpc-dpa-workflow` skill uses (its `references/cli.md`, Signing in):

1. The agent launches the login window in the background (10 minute timeout); the person signs
   in inside it. Never type credentials for anyone.
   ```bash
   ~/go/bin/press-auth login sdpc.a4l.org --login-url https://sdpc.a4l.org/login.php --force --complete-selector 'a[href*="logout"]'
   ```
2. When it completes, import the cookie into the CLI:
   ```bash
   PATH="$HOME/go/bin:$PATH" sdpc-pp-cli auth login --chrome
   ```
   Success prints "Loaded cookies via press-auth for sdpc.a4l.org."
3. Run gated reads right away. Sessions have gone stale within ten minutes. A gated page that
   answers with "SDPC: Error" or "requires a logged-in session" means the session expired:
   repeat steps 1 and 2. Signing in to sdpc.a4l.org in your normal Chrome does not refresh the
   CLI's session.

The Premium JSON API (agreements, sync --resources agreements) needs Premium membership credentials
exported as SDPC_USERNAME and SDPC_PASSWORD; session login exchanges them for an API session
cookie. Boise has not configured these. The CLI keeps one cookie jar, so session login replaces
the browser session and the reverse.

Run `sdpc-pp-cli doctor` to verify setup.

## Agent Mode

Add `--agent` to any command. Expands to: `--json --compact --no-input --no-color`.

Global format flags share one contract on promoted, novel, sync, and `--deliver` paths:

- `--json` — one JSON document on stdout (sync progress events go to stderr)
- `--compact` — keep identity/status/timestamp fields; does not change the document vs stream shape
- `--csv` / `--plain` — tabular rows (collection envelopes unwrap to the row array)
- `--quiet` — one identity value per row, no envelope

- **Pipeable** — JSON on stdout, errors on stderr
- **Filterable** — `--select` keeps a subset of fields. Dotted paths descend into nested structures; arrays traverse element-wise. Critical for keeping context small on verbose APIs:

  ```bash
  sdpc-pp-cli agreements --agent --select dataID,agreement_typesID,signed_agreement_file
  ```
- **Previewable** — `--dry-run` shows the request without sending
- **Offline-friendly** — sync/search commands can use the local SQLite store when available
- **Non-interactive** — never prompts, every input is a flag
- **Explicit confirmation** — `--agent` does not imply `--yes`; pass `--yes` separately only after the target, arguments, and side effects are clear

### Response envelope

Commands that read from the local store or the API wrap output in a provenance envelope:

```json
{
  "meta": {"source": "live" | "local", "synced_at": "...", "reason": "..."},
  "results": <data>
}
```

Parse `.results` for data and `.meta.source` to know whether it's live or local. A human-readable `N results (live)` summary is printed to stderr only when stdout is a terminal AND no machine-format flag (`--json`, `--csv`, `--compact`, `--quiet`, `--plain`, `--select`) is set — piped/agent consumers and explicit-format runs get pure JSON on stdout.

## Paths and state

Agents should treat the CLI's path resolver as part of the runtime contract:

- Use `--home <dir>` for one invocation, or set `SDPC_HOME=<dir>` to relocate all four path kinds under one root.
- Use per-kind env vars only when a specific kind must diverge: `SDPC_CONFIG_DIR`, `SDPC_DATA_DIR`, `SDPC_STATE_DIR`, `SDPC_CACHE_DIR`.
- Resolution order is per-kind env var, `--home`, `SDPC_HOME`, XDG (`XDG_CONFIG_HOME`, `XDG_DATA_HOME`, `XDG_STATE_HOME`, `XDG_CACHE_HOME`), then platform defaults.
- `config` contains settings like `config.toml` and profiles. `data` contains `credentials.toml`, `data.db`, cookies, and auth sidecars. `state` contains persisted queries, jobs, and `teach.log`. `cache` contains regenerable HTTP/cache files.
- Stored secrets live in `credentials.toml` under the data dir. Existing legacy `config.toml` secrets are read for compatibility and leave `config.toml` on the first auth write.
- Run `sdpc-pp-cli doctor --fail-on warn` to surface path and credential-location warnings. `agent-context` exposes a schema v4 `paths` block for agents that need the resolved dirs.
- For MCP, pass relocation through the MCP host config. The MCP binary does not inherit CLI flags:

  ```json
  {
    "mcpServers": {
      "sdpc": {
        "command": "sdpc-pp-mcp",
        "env": {
          "SDPC_HOME": "/srv/sdpc"
        }
      }
    }
  }
  ```

Fleet precedence: an inherited per-kind env var overrides an explicit `--home` for that kind. Use `SDPC_HOME` or per-kind vars as durable fleet levers, and use `--home` only for a single invocation. Relocation is not reversible by unsetting env vars; move files manually before clearing `SDPC_HOME`, or `doctor` will not find credentials left under the former root.

## Automatic learning

This CLI ships a self-capturing learning loop. The CLI does its own bookkeeping: every invocation is journaled locally, a failed flag followed by a corrected retry auto-derives a `flag_alias` candidate, and a `teach` on a query family without a playbook auto-synthesizes a `playbook_candidate` from the session's journal. Your job is judgment only: `recall` first, act on surfaced candidates, `teach` the final answer, `playbook amend` when you observe a correction. You never record failures by hand.

### Step 1: `recall` before any discovery

Before list/search/drill commands on a new user question, pass the question as an argv or MCP tool argument to `recall --agent`. Do not interpolate user-controlled text into a shell command line.

Quoted `recall "<question>"` breaks on an apostrophe, which is ordinary English. A quoted heredoc breaks when a body line equals the delimiter, and that delimiter is published in these docs. Write the question with a non-shell file-writing tool, then read it back as data:

```bash
# Write the question verbatim with your file-writing tool (no shell involved).
# Command substitution on a file only ever yields data — the shell never
# parses the file's bytes as syntax.
QUERY=$(cat /path/to/question.txt)
sdpc-pp-cli recall "$QUERY" --agent
```

Prefer MCP: pass the question as the tool's query argument. `"$QUERY"` after a file read is argv-safe; putting the question itself in the command text is not.

The response envelope:

```json
{
  "query": "...",
  "normalized": "<normalized form>",
  "query_entities": ["..."],
  "found": true | false,
  "match_score": 0.0,
  "results": [
    { "resource_id": "...", "resource_type": "...", "venue": "...",
      "confidence": 2, "entity_match": "exact|partial|unknown",
      "source": "taught|preseed|pattern", "warnings": ["..."] }
  ],
  "mismatches": [ /* only when --debug-mismatches */ ],
  "warnings": [ /* top-level */ ],
  "candidates": [
    { "id": 12, "class": "flag_alias | playbook_candidate",
      "summary": "...", "sightings": 3, "last_seen": "...",
      "rationale": "...",
      "next_action": ["<trial command>", "sdpc-pp-cli learnings confirm 12"] }
  ],
  "playbook": {
    "query_family": "...",
    "playbook": {
      "steps": [ { "cmd": "<command with {slot} substitution>", "purpose": "..." } ],
      "entity_slots": ["$ENTITY"],
      "expected_tool_calls": 3
    },
    "slots_resolved": { "$ENTITY": { "token": "<live token>", "canonical": "<canonical>" } },
    "notes": "<workarounds + gotchas for this query family>"
  },
  "notes": "<duplicate surface for non-playbook callers>"
}
```

Empty-store short-circuit: if the store has no learnings, playbooks, or candidates yet (recall finds nothing and `learnings list` and `learnings candidates` are both empty), skip recall for the rest of this session instead of taxing every query; resume recall-first once something has been taught.

### Step 2: decision tree

Read `candidates`, `playbook`, `notes`, `results[0]`, and warnings in that order:

```
if Candidates present (warnings include "candidates_present"):
    -> candidates are try-then-confirm, never facts. Follow each candidate's
       two-step next_action verbatim: run the trial command first, then run
       `learnings confirm <id>` only after the trial verified the behavior.
       Reject a wrong candidate with `learnings reject <id>`.
    -> NEVER re-teach something recall surfaced as a candidate; confirm or
       reject that candidate instead of teaching a duplicate.
    -> candidates ride alongside playbooks and resource hits, not instead of
       them; continue with the branches below after acting on them.

if Playbook present:
    -> READ Playbook.notes verbatim FIRST (workarounds + gotchas the CLI surface doesn't expose)
    -> replay Playbook.steps in order, substituting Playbook.slots_resolved entries
       for the entity slot tokens. If a step's slot is unresolved, fall back to
       discovery for that step only.
    -> the Playbook's expected_tool_calls is a budget; if you find yourself running
       materially more, record the divergence via `sdpc-pp-cli playbook amend`
       at end-of-session.

elif Notes present (no Playbook):
    -> read Notes verbatim before any discovery step; they carry known gotchas
       for this query family even when no structured choreography exists yet.

elif Found AND Results[0].EntityMatch == "exact" AND Results[0].Confidence >= 2:
    -> skip discovery; fetch live data for Results[*].ResourceID in parallel

elif Found AND Results[0].EntityMatch == "partial":
    -> candidate hint, NOT a hit; read the resource title to validate before trusting

elif (any row in Mismatches[] when --debug-mismatches was passed):
    -> treat as cold start; the stored learning is for a different entity
       (different canonical resolved from query_entities)

else:  // Found == false, no playbook, no notes
    -> cold start; run discovery normally; teach the answer afterward (Step 4).
       If the family has no playbook yet, that teach auto-synthesizes a
       playbook candidate from this session's journal - you do not need to
       record one by hand.
```

Playbook and Notes are orthogonal to the per-resource path. A recall response can carry both a Playbook AND a `Results[]` hit - use both: the Playbook tells you which choreography to run; the resource hits short-circuit specific steps. Default to skipping `mismatches`; pass `--debug-mismatches` only when investigating cold-start surprises.

Candidate judgment details: `learnings confirm <id>` prints the candidate's full payload before materializing it - check that the printed payload matches the behavior you verified. `learnings reject <id>` tombstones the derivation signature so the same candidate does not resurface. The envelope carries only the few candidates worth acting on now; `sdpc-pp-cli learnings candidates` lists the full open set.

Graceful degradation: if `learnings confirm` is an unknown command, you are driving an older binary - ignore the candidates guidance and follow the rest of the protocol.

### Step 3: always read `warnings`

- `low_confidence`: row exists at `confidence<2`. Treat as a hint, not a skip-discovery hit.
- `resource_not_in_store`: the local store doesn't have the resource the learning points at. The match validator couldn't classify entities — direct-fetch and re-evaluate.
- `cross_alias_match` (per-result): the row was taught under a different alias and matched the live query's canonical via `entity_lookups` (e.g., a "USA" teach satisfying a "United States" recall). Trust the resource_id.
- `similar_shape_different_entity:<canonical>` (top-level): a structurally matching row exists but its canonical entity differs from the live query's. Treated as cold start; the warning carries the conflicting canonical as a hint, but the row is NOT promoted into Results.
- `ambiguous_alias` (top-level): a single query entity resolved to multiple canonicals (e.g., "Cards" → Arizona Cardinals + St. Louis Cardinals). Surface the ambiguity from context before committing to a resource.
- `candidates_present` (top-level): the envelope carries a `candidates` section. Handle it via the candidates branch in Step 2 before anything else.
- `lookup_refresh_available` (top-level): an entity in the query has no lookup row yet, but synced data could provide one. Run `sdpc-pp-cli sync --resources agreements,alliance,alliance-in-negotiations-paging-php,alliance-national-participants-paging-php,alliance-new-requests-state-paging-php,alliance-view-resource-list-paging-php,alliance-view-signed-dpas-all-paging-php,lookup,lookup-district-search-ajax-php,lookup-resource-search-ajax-php,lookup-vendor-search-ajax-php` to refresh entity lookups.
- Top-level `no_learnings_for_query_family`: the table had no rows above the Jaccard floor. Pure cold start.

### Step 4: `teach &` after finalizing your response - always

Teaching is unconditional. After resolving a query the store could not answer, background-teach the final resource mapping - no call-count threshold, no judging whether it was "worth" learning. The teach is the anchor of the loop: it triggers playbook synthesis for a family without a playbook, and same-referent phrasings fold into one family so near-duplicate teaches do not fragment the store. Fire it after assembling your user-facing response but BEFORE emitting it, with a shell `&` so the call returns immediately. Pass the query the same way as recall — argv/MCP, or file-then-`$QUERY`. Do not splice the question into the command text:

```bash
QUERY=$(cat /path/to/question.txt)
sdpc-pp-cli teach --query "$QUERY" --resource-type <type> --resource <id1> --resource <id2>
# (append shell `&` to background it)
```

Silent on success. Errors only land in `teach.log` under the resolved state dir. Teach the **most specific** resource - if the user asked a broad question and you walked through parent records to find the specific answer, teach the leaf id, not the parent. The CLI uses seeded `entity_lookups` for cross-alias resolution at recall time, so a teach under one alias (e.g., "Niners") satisfies future queries under another alias (e.g., "49ers", "San Francisco") automatically.

PII rule: teach the structural question with identifiers stripped - never include names, emails, phone numbers, account ids, or other personal identifiers in taught queries or notes. The CLI scans teach queries for obvious email/phone shapes and warns, but does not block; strip before teaching rather than relying on the warning.

### Step 5: playbooks - optional flags, automatic synthesis

You do not need to decide whether a session "deserves" a playbook: a teach on a family without one auto-synthesizes a `playbook_candidate` from the session's journal, and the next session judges it via confirm/reject. Attach explicit playbook flags only when you already hold choreography worth recording verbatim - workarounds the CLI didn't surface (silently-dropped flags, undocumented params, pagination tricks, payload gotchas). Prefer the **integrated one-call form** - record the resource learning and the playbook in the same `teach` invocation:

```bash
# Common case: record both the resource learning AND the playbook in one call.
QUERY=$(cat /path/to/question.txt)
sdpc-pp-cli teach \
  --query "$QUERY" \
  --resource <id> \
  --playbook-file ~/playbooks/<shape>.json \
  --playbook-notes-file ~/playbooks/<shape>-notes.md
# (append shell `&` to background it)

# Alternate: playbook-only (no resource to record alongside).
QUERY=$(cat /path/to/question.txt)
sdpc-pp-cli teach-playbook \
  --query "$QUERY" \
  --playbook-file ~/playbooks/<shape>.json \
  --notes-file ~/playbooks/<shape>-notes.md
```

Playbook files are JSON with `steps`, `entity_slots`, `expected_tool_calls`. Notes files are markdown carrying the gotchas verbatim. File-free callers (MCP-only agents) pass the same content inline: `--playbook-json` and `--playbook-notes` on the integrated `teach` form, `--playbook-json` and `--notes` on `teach-playbook`. On the integrated `teach` form, the playbook flags are optional - omit them entirely for a resource-only teach. On the standalone `teach-playbook` form, at least one of the playbook and notes flags must be set; both empty is rejected. Playbooks are keyed on the structural query family (entities stripped) so a recipe taught from one entity-shaped query applies to every other query of the same shape, with `slots_resolved` binding the live query's canonical at recall time.

When you DO find a playbook on a future recall, treat it as ground truth: replay the steps with `slots_resolved` substitutions, skip the discovery that the choreography already documents, and read `notes` before any step.

### Step 6: `playbook amend &` when your debug response identifies a correction

If your debug-protocol response identifies a concrete correction the notes or playbook should know — a workaround, an undocumented endpoint shape, a stale field name, observed schema drift, an empty-payload fallback — fire `playbook amend` BEFORE emitting your user-facing response. Same fire-and-forget posture as `teach`. Pass the query and note as argv/MCP arguments, or write each with a non-shell file tool and read them back (`QUERY=$(cat ...)`, `NOTE=$(cat ...)`). Do not interpolate either string into the command text:

```bash
QUERY=$(cat /path/to/question.txt)
NOTE=$(cat /path/to/note.txt)
sdpc-pp-cli playbook amend \
  --query "$QUERY" \
  --add-note "$NOTE"
# (append shell `&` to background it)
```

What counts as worth amending: a behavior you OBSERVED this session that future-you would benefit from knowing. Examples worth amending:

- A workaround for a CLI surface that silently drops or misorders a flag.
- An undocumented endpoint shape (response wrapped in `{meta, results}`, payload nested two levels deeper than the docs claim).
- Observed schema drift (a field renamed, an index that shifted between seasons, a category label that the API now returns lower-cased).

What does NOT belong in notes:

- The year-specific or entity-specific answer to the user's question. That's the response, not a learning.
- Per-team / per-athlete / per-row data the playbook already retrieves at runtime.
- Statements that paraphrase what the existing notes already say.

The amend command appends to the family's existing notes with a timestamped marker (`[amend YYYY-MM-DDTHH:MMZ]: <text>`). Multiple amends accumulate; the audit trail is visible. If no playbook exists yet for the family, amend creates a notes-only one (so cold-start corrections still land).

#### PII discipline for amend notes

`playbook amend` notes are designed to potentially flow upstream as shared knowledge in future versions of the Printing Press. Keep them clean of user-identifying content so the upstream-contribution path stays open without retroactive scrubbing:

- **Do NOT embed** paths to user filesystems, personal API keys or tokens, user email addresses, user GitHub handles, or specific query histories tied to a single user.
- **Acceptable**: endpoint shapes, undocumented field names, API gotchas, observed schema drift, workarounds for CLI surfaces, generalizable pagination or retry tactics.

If a correction is only meaningful with user-specific context, it belongs in a personal note, not in the playbook amend.

### Measuring the loop

`sdpc-pp-cli learnings stats` reports recall hit rate, teach-to-reuse, playbook resolution rate, and candidate confirm/reject counts from the local `learn_events` table. Rates are null until they have a denominator; everything stays on this machine. Use it to check whether the loop is earning its keep for this CLI.

### Disabling learning

- `--no-learn` on a single command short-circuits both `recall` and the `teach` write path. Use for deterministic agent flows or tests that must not be affected by accumulated learnings.
- `SDPC_NO_LEARN=true` in the environment globally disables the pipeline.

## Agent Feedback

When you (or the agent) notice something off about this CLI, record it:

```
sdpc-pp-cli feedback "the --since flag is inclusive but docs say exclusive"
sdpc-pp-cli feedback --stdin < notes.txt
sdpc-pp-cli feedback list --json --limit 10
```

Entries are stored locally as `feedback.jsonl` under the resolved data dir. They are never POSTed unless `SDPC_FEEDBACK_ENDPOINT` is set AND either `--send` is passed or `SDPC_FEEDBACK_AUTO_SEND=true`. Default behavior is local-only.

Write what *surprised* you, not a bug report. Short, specific, one line: that is the part that compounds.

## Output Delivery

Every command accepts `--deliver <sink>`. The output goes to the named sink in addition to (or instead of) stdout, so agents can route command results without hand-piping. Three sinks are supported:

| Sink | Effect |
|------|--------|
| `stdout` | Default; write to stdout only |
| `file:<path>` | Atomically write output to `<path>` (tmp + rename). Binary-response commands write decoded payload bytes (not the base64 JSON envelope) and print a small JSON receipt on stdout; `--json`/`--csv` do not refuse when this sink is set. |
| `webhook:<url>` | POST the output body to the URL (`application/json`) |

Unknown schemes are refused with a structured error naming the supported set. Webhook failures return non-zero and log the URL + HTTP status on stderr.

## Named Profiles

A profile is a saved set of flag values, reused across invocations. Use it when a scheduled or recurring agent reuses the same saved flags while providing different input each run.

```
sdpc-pp-cli profile save briefing --json
sdpc-pp-cli --profile briefing agreements
sdpc-pp-cli profile list --json
sdpc-pp-cli profile show briefing
sdpc-pp-cli profile delete briefing --yes
```

Explicit flags always win over profile values; profile values win over defaults. `agent-context` lists all available profiles under `available_profiles` so introspecting agents discover them at runtime.

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

1. **Empty, `help`, or `--help`** → show `sdpc-pp-cli --help` output
2. **Starts with `install`** → ends with `mcp` → MCP installation; otherwise → see Prerequisites above
3. **Anything else** → Direct Use (execute as CLI command with `--agent`)

## MCP Server Installation

1. Build the MCP server from the CLI source:
   ```bash
   go build -o ~/.local/bin/sdpc-pp-mcp ./cmd/sdpc-pp-mcp
   ```
2. Register with Claude Code:
   ```bash
   claude mcp add sdpc-pp-mcp -- sdpc-pp-mcp
   ```
3. Verify: `claude mcp list`

## Direct Use

1. Check if installed: `which sdpc-pp-cli`
   If not found, build it from source (see Prerequisites).
2. Match the user query to the best command from the Unique Capabilities and Command Reference above.
3. Execute with the `--agent` flag:
   ```bash
   sdpc-pp-cli renewals --within 180d --state ID --agent
   ```
4. If ambiguous, drill into subcommand help: `sdpc-pp-cli national --help`.