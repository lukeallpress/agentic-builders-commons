---
name: cookbook-update
description: Update YOUR practitioner profile and submit YOUR recipes/ideas to the Agentic Builders cookbook, using your own Coda API token. Use when a committee member wants to update their cookbook presence — profile, stack, projects — without waiting on the organizer.
---

# Cookbook self-serve update — skill instructions

*From the Agentic Builders Committee. This skill lets your coding agent update **your own**
entry in the group's shared knowledge base (a Coda doc). The public cookbook at
https://lukeallpress.github.io/agentic-builders-commons/cookbook.html rebuilds from that
doc automatically, so your edits show up on the site without anyone in the loop.*

**Claude Code users:** drop this file at `.claude/skills/cookbook-update/SKILL.md` and say
"update my cookbook profile". **Any other agent:** just tell it to read this file and follow it.

## One-time setup (human does this)

1. You must be a collaborator on the committee's Coda doc (ask Luke if you're not).
2. Create your own Coda API token: **Coda → Account Settings → API → Generate token.**
3. Store it as the environment variable `CODA_API_TOKEN` (e.g. in a local `.env`).
   **Never commit it, never paste it into a chat that gets shared.**

## API facts (for the agent)

- Base URL: `https://coda.io/apis/v1` (note `/apis/`, not `/api/`)
- Doc ID: `6SpujExxEK`
- Auth: `Authorization: Bearer $CODA_API_TOKEN`
- Discover table IDs by name at runtime: `GET /docs/6SpujExxEK/tables` — the tables are
  `People`, `Projects / Recipes`, `Ideas`, `Articles`, `Open Questions`, `Calls`.
  Never hardcode table IDs.
- Column names are `Snake_Case` (e.g. `Core_Stack`, `Recipe_Name`). `Slug` is the key.
- Upsert = `POST /docs/6SpujExxEK/tables/{tableId}/rows` with body
  `{"rows":[{"cells":[{"column":"Slug","value":"..."}, ...]}], "keyColumns":["Slug"]}`
  — column *names* are accepted. Upserts are idempotent on `Slug`.

## Hard rules

- **Only ever write rows that belong to your human.** Find their People row by name and
  work with that `Slug`; for projects/ideas, only rows they tell you are theirs, or new
  rows you create for them. Never modify or delete anyone else's row. Never delete anything.
- New project/idea/article rows: set `Review_Status` to `needs_review` (the committee
  reviews before it publishes). When updating your own existing People row, leave
  `Review_Status` as it is.
- Nothing sensitive in any cell: no credentials, internal hostnames or IPs, student data,
  or other people's contact info.

## Updating your profile

1. `GET` the People table rows, find your human's row, and show them what it currently says.
2. Interview them briefly — one topic at a time, a sentence or two each:
   - **Identity / SSO** — how do users sign in to your tools? Do agents get their own accounts?
   - **Hosting** — where do builds actually run? (cloud, on-prem, home lab, a Mac Mini — the
     group runs all of these.)
   - **Data** — where does district data live when agents use it, and how does it get there?
   - **Daily stack** — models, coding agents, tools they actually reach for.
   - **What's new** — projects or infrastructure changes since the last update.
3. Distill into the row's fields — `Core_Stack` (comma-separated, concrete),
   `Current_Projects`, `Areas_of_Expertise`, `Needs_Help_With`, `Role_Title` — and show
   the human the before/after for each changed field.
4. On their OK, upsert the row (keyed on `Slug`).

## Submitting a recipe or idea

1. If the project has a repo, run the committee's **recipe-prep agent** first
   (`cookbook-recipe-prep.md`, downloadable from the cookbook) — it produces a
   `COOKBOOK_SUBMISSION.md` with everything you need.
2. Map it into the `Projects / Recipes` columns: `Slug` (kebab-case), `Recipe_Name`,
   `Problem_Statement`, `Context_Requirements`, `Tools_Involved`,
   `Data_Access_Requirements`, `Setup_Steps`, `Failure_Modes`, `Example_Code_Prompts`,
   `Review_Status` = `needs_review`. (Ideas: `Slug`, `Title`, `Type`, `Summary`,
   `Maturity`, `Data_Guardrails`, `Human_In_The_Loop`, `Tools_Involved`.)
3. Upsert. The curated presentation extras (tags, prerequisite links, the sample prompt
   shown in the cookbook drawer) are added during review — include your material in the
   row and it gets woven in.

## When you're done

Tell the human: changes are in the group's knowledge base now; the public cookbook
rebuilds from it on the nightly schedule, and brand-new records appear after the
committee's review pass. Anything odd, email Luke: lallpress@aguafria.org.
