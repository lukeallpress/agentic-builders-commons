---
name: sdpc-weekly-batch
description: File the next batch (default 10) of Boise School District student data privacy agreement requests in the SDPC Resource Registry from the prioritized vendor inventory, with duplicate checks, Exhibit E checks, previews, TEC assignment, and a ledger update. Use every Monday, or when asked to "file the next 10", "run the weekly SDPC batch", "add 10 more from the priority list", or "submit the next batch to TEC". Depends on the sdpc-dpa-workflow skill and the local sdpc-pp-cli.
---

# SDPC weekly batch (Boise)

*Shared by Ryan Gravette, Boise School District (Idaho), on the Agentic Builders skills shelf. It
is Boise's Monday runbook, edited for sharing: the district ID (15032), state (ID), and TEC (the
consortium that works Boise's requests and holds the Idaho add-on) are Boise's values; swap in your
own. District-internal references (team roster, counsel's vendor reviews, the parent-release
samples) are not included. The `sdpc-pp-cli` tool is a Printing Press CLI described in the
matching cookbook recipe; it is not yet in the public library.*

Runs the Monday routine set up on 2026-09-28: take the next ten unfiled systems from Boise's
prioritized inventory (P1 first, then P2, then P3, in inventory order), file each as a New Request
in the SDPC registry, assign TEC, and record it. Everything lives in the `sdpc-dpa-workflow` skill
at `~/.claude/skills/sdpc-dpa-workflow`; this skill is the runbook. Read that skill's SKILL.md
guardrails first if this is your first time in the registry.

`WF=~/.claude/skills/sdpc-dpa-workflow` below. Use the scratchpad directory for batch files.

## 0. Sign in (you launch, the person signs in)

```bash
~/go/bin/press-auth login sdpc.a4l.org --login-url https://sdpc.a4l.org/login.php --force --complete-selector 'a[href*="logout"]'
```

Run it in the background with a 10 minute timeout. Tell the person the window is up. When it completes:

```bash
PATH="$HOME/go/bin:$PATH" $WF/scripts/sdpc auth login --chrome
```

Sessions have expired ten minutes after sign-in, so run steps 1 to 4 promptly and in as few
commands as possible. If anything reports "requires a logged-in session", repeat step 0.

## 1. Plan (read-only)

```bash
python3 $WF/scripts/weekly_batch.py plan --count 10 --out-dir <scratchpad>
```

It refuses to run without the live duplicate check.

**Stop here when the plan is empty.** If it prints `0 New Requests to file` (JSON:
`"nothing_to_file": true`), do not run steps 2 to 5: there is nothing to preview, file, assign, or
record. Report that the inventory has no unfiled P1 to P3 systems this week, give the
skipped count, and list the leftovers that need a person: Path B items (an Exhibit E to create
after reading the originating agreement, with its own preview and yes) and needs-a-name items.
P4 systems are not in the weekly order. Then end the run cleanly. Do not unload or edit the
Monday launcher; whether to pause it is the person's call. The inventory was exhausted as of 2026-10-11:
plan returned 0 to file, with 135 of the 136 P1 to P3 rows already filed or skipped (Garbanzo had
no exact catalog name).

Otherwise it prints four groups:

- **Plan**: the systems to file, with the exact catalog name, softwareID, proposed content area,
  and a purpose built from the inventory row. Writes `batch-<date>.tsv` and `.json`.
- **Path B**: an Exhibit E is available to Boise. Do not file a New Request for these. Read the
  originating agreement PDF, then use `exhibit-e create` per the workflow skill, with its own
  preview and confirmation. Tell the person; these are a signature on the district's behalf.
- **Needs a name**: the inventory name did not match a catalog name exactly. Pick the right catalog
  entry from the hits shown (or `lookup resource --query`), then add a line to the TSV by hand.
- **Skipped**: already in the registry, in the ledger, or on `assets/data/batch-skip.txt`.

Sanity-check the plan: grades default to K-12 and the content area is a keyword guess. Adjust the
TSV if a row is clearly wrong (a high-school-only tool, a staff-only system that belongs in
"Other"). Pick systems only from the plan; do not improvise additions.

## 2. Duplicate check, again

The plan already checked live rows, but re-read them right before writing:

```bash
PATH="$HOME/go/bin:$PATH" $WF/scripts/sdpc my-district agreements --district-id=15032 --state=ID --agent
```

Confirm no row shares a softwareID or resource name with anything in the TSV. The operator's standing rule: a duplicate check for each one, every time.

## 3. Preview, then file

Preview every row first (the permission classifier blocks a write that was not previewed):

```bash
while IFS=$'\t' read -r sid name grades area purpose system pri; do
  $WF/scripts/sdpc request submit --software-id "$sid" --district-id=15032 --state=ID \
    --requestor "Your Name" --requestor-email you@district.org \
    --grade "$grades" --content-area "$area" --purpose "$purpose" --dry-run --no-input
done < batch-<date>.tsv
```

Then the same loop with `--yes` in place of `--dry-run`. Capture each result's `data_id`,
`resource_name`, `company_name`, and `row_status` into `batch-<date>_filed.tsv` as
`data_id<TAB>software_id<TAB>resource_name<TAB>company<TAB>status<TAB>system`.
Do not use `--assign-tec`; assign in a separate step so each write has its own preview.

Shell note: do not name a loop variable `status`; zsh treats it as read-only.

## 4. Assign TEC

Preview all, then apply all, by data ID:

```bash
$WF/scripts/sdpc request assign --data-id <id> --district-id=15032 --state=ID --dry-run --no-input
$WF/scripts/sdpc request assign --data-id <id> --district-id=15032 --state=ID --yes --no-input
```

Each result must show `after.progress_admin` = TEC and `verified` true. TEC is not notified until
this step runs.

## 5. Record

```bash
python3 $WF/scripts/weekly_batch.py record batch-<date>_filed.tsv
```

Then add the rows to the district's agreements-on-record table (product, company, "New Request, TEC assigned, data ID N, softwareID M", filed date).

## 6. Report

Lead with the count filed and the data IDs. List each system on one line with its priority,
company, and anything TEC or counsel should know (P1 systems need counsel review before any
teacher is told yes). List Path B and needs-a-name items as follow-ups. Say plainly if a write
failed or the session expired partway, and which rows were left unfiled.

## Where things live

- Inventory: `$WF/assets/data/BSD vendor privacy priorities.csv` (144 Tier 1 and 2 systems,
  P1 25, P2 61, P3 50, P4 8; built 2026-09-09). The Massachusetts Exhibit E list sits beside it;
  refresh it by copying a newer list into the repo copy and syncing (see the workflow skill's
  `references/boise-context.md`). Triage reads Google sign-in counts from the newest
  `~/Downloads/owl_apps_export*.xlsx`.
- Ledger: `$WF/assets/data/batch-ledger.json`. Skip list: `$WF/assets/data/batch-skip.txt`.
- The registry itself is the source of truth for duplicates; the ledger is a backup.
- Monday launcher: `$WF/assets/org.boiseschools.sdpc-weekly-batch.plist` opens a Terminal window
  running `claude "/sdpc-weekly-batch"` at 7:30 AM local time on Mondays. The person installs it themselves
  (Claude Code's permission rules block agents from installing launch agents):
  `cp $WF/assets/org.boiseschools.sdpc-weekly-batch.plist ~/Library/LaunchAgents/ && launchctl load ~/Library/LaunchAgents/org.boiseschools.sdpc-weekly-batch.plist`.
  `launchctl unload` the installed file to pause it (the person's decision; agents do not load, unload,
  or edit it). If it is not installed, the batch still runs whenever the person types
  `/sdpc-weekly-batch`.
- Launcher failure seen in `/tmp/sdpc-weekly-batch.log`: `execution error: Terminal got an error:
  AppleEvent timed out. (-1712)`. osascript waits about two minutes for Terminal to answer
  `do script`; at 7:30 AM Terminal can be slow to launch (Mac just woke, screen locked, or a
  pending Automation permission prompt), so no window opened and no batch ran. When that line is
  in the log, say so in the report and run the batch by hand. Mitigation for the person to apply: wrap the AppleScript in `with timeout of 600 seconds ... end timeout` with a
  `delay 5` after `activate`, then reload the agent; and confirm Terminal is allowed under
  System Settings > Privacy & Security > Automation for osascript.

## Scripts (not included)

`weekly_batch.py` (about 280 lines) does two things: `plan` reads the prioritized inventory CSV,
the ledger, the skip list, and a live `my-district agreements` read, resolves each inventory name
to an exact catalog name with `lookup resource`, checks `exhibit-e list` for Path B, and writes the
TSV; `record` appends the filed rows to the ledger. It is tied to Boise's inventory columns, so
rebuild it against yours. The launchd plist is a standard `ProgramArguments` entry running
`osascript` to open Terminal with `claude "/sdpc-weekly-batch"` on a weekly `StartCalendarInterval`.