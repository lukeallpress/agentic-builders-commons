# Cookbook Recipe Prep — agent instructions

*From the Agentic Builders Committee. Drop this file into your project repo and tell your
coding agent (Claude Code, Codex, Cursor — whatever you build with):*

> Read agents/cookbook-recipe-prep.md and follow it.

*The agent will study your project and produce a submission packet for the committee's
Agentic Cookbook — so your build becomes a recipe another district can actually reproduce.*

---

## Your task (instructions for the coding agent)

You are preparing this repository's project for the **Agentic Cookbook**, a shared recipe
book of agentic builds by K-12 practitioners. Your job is to study the project as it
actually exists — code, configs, prompts, README, deploy scripts — and write a single file,
`COOKBOOK_SUBMISSION.md`, in the repo root.

Ground every claim in the repository. Where something critical isn't discoverable from the
code (who it's for, what data it touches in production, what broke), **ask the builder
directly** rather than guessing. A short interview beats a wrong recipe.

### 1. Study the project

- Read the README, entry points, configs, CI/deploy files, and any prompt files.
- Identify: what the tool does, who uses it, what it's built with, where it runs, how users
  sign in, and what data it reads or writes.
- Note anything a rebuilder would trip over: undocumented environment variables, manual
  setup steps, vendor quirks.

### 2. Write `COOKBOOK_SUBMISSION.md` with exactly these sections

```markdown
# <Project name>

**Builder:** <name, district/org>
**Status:** <idea | prototype | pilot | in production>
**Tags:** <3-5 lowercase topic tags, e.g. mcp, data warehouse, communications>

## What it is
<2-4 sentences a colleague would enjoy reading. Concrete and vivid, no hype — the
"digital student ID where students log in to prove their identity anywhere on campus"
register. What does it do, for whom, and what's the payoff?>

## Prerequisites
<Which of these foundations must be in place before someone attempts this build?
Use ONLY these keys, one bullet each, with one line on how THIS project uses it:>
- sso — <e.g. staff sign in with district Google OAuth>
- hosting — <e.g. runs on a district subdomain behind IIS>
- data-pipeline — <e.g. reads the nightly PowerSchool export in the warehouse>
- guardrails — <e.g. read-only; principals scoped to their building>
- reproducibility — <e.g. prompts and configs versioned in this repo>
- accessibility — <only if student/public-facing; what standard it meets>
<Omit any that genuinely don't apply.>

## Data needs
| Data | Where it comes from |
|------|---------------------|
| <e.g. Student photo> | <e.g. SIS export table STUDENT_PHOTO> |
| <e.g. Class schedule> | <e.g. PowerSchool API /sections> |
<Every dataset the build touches, or "None — no district data involved.">

## The recipe
<Numbered steps to rebuild this from zero, honest about effort. Include: environment
setup, auth registration, data plumbing, the build itself, guardrail configuration,
and how to verify it works. 10-25 steps is typical. Name real tools and versions.>

## Watch out for
<The failure modes: what broke, what you'd do differently, where the sharp edges are.>

## Repo
<If this project's code is PUBLIC, the repo URL — often more useful to a peer than any
prompt. Ask the builder before assuming: never list a private or internal-only repo, and
never a URL that exposes district infrastructure. Write "None — not public" if so.>

## Sample prompt
<A single long prompt (150-400 words) that a peer could paste into their own coding
agent to reproduce the core of this build. Imperative voice. Include the architecture,
the stack, the guardrails, and the definition of done. Write it as if briefing a
capable engineer who has the prerequisites in place but has never seen this project.
If there's a public repo, this is still worth writing, but it can be shorter — the two
are complementary, and either alone is enough to publish.>

## Screenshot
<See step 3.>

## Practitioner profile update
<See step 2b — the builder's current infrastructure, for their cookbook profile.>
```

### 2b. Interview the builder about their infrastructure

The cookbook shows a profile for every builder — their stack and how they run things —
so peers can learn from the person, not just the project. This repo only shows part of
that picture, so **ask the builder directly**, one topic at a time, and keep it
conversational (they can answer in a sentence or two each):

1. **Identity / SSO** — how do users sign in to your tools? (Google OAuth, Entra/AD,
   Firebase, something else?) Do agents get their own accounts?
2. **Hosting** — where do your builds actually run? (cloud provider, on-prem, home lab,
   a Mac Mini on a desk — no judgment, the group runs all of these.)
3. **Data** — where does your district data live when agents use it, and how does it
   get there? (warehouse + exports, MCP layer, vendor APIs?)
4. **Daily stack** — the models, coding agents, and tools you actually reach for.
5. **What changed lately** — anything new since you last updated your profile.

Distill the answers into a short `## Practitioner profile update` section: a
comma-separated stack list plus 1-3 sentences on their infrastructure patterns.
Skip anything the builder declines to share; never include credentials, IPs,
internal hostnames, or anything they call sensitive.

### 3. Capture a screenshot

Everyone likes pretty things. Capture the tool's most representative screen:

- If you (the agent) can run the app and screenshot it, do that — save as
  `cookbook-screenshot.png` next to the submission and reference it.
- Otherwise, leave a placeholder line telling the builder exactly which screen to
  capture (e.g. "the dashboard right after login") and at what size (1280×800 or wider).
- **Redact real student/staff data first** — use a demo account or synthetic data. If the
  visible screen contains any real names, photos, or records, say so and stop; the builder
  must swap in demo data before capturing.

### 4. Quality bar before you finish

- [ ] Every prerequisite listed uses only the six keys above.
- [ ] The recipe was checked against the actual repo — no invented steps or tools.
- [ ] The sample prompt would genuinely produce this architecture, not a generic app.
- [ ] No secrets, tokens, internal URLs, student data, or staff emails anywhere in the file.
- [ ] You asked the builder about anything you couldn't verify, and marked anything still
      uncertain with `TODO(builder):`.
- [ ] You interviewed the builder for the profile update (step 2b) rather than guessing
      their infrastructure from this one repo.

### 5. Hand off

Tell the builder: review `COOKBOOK_SUBMISSION.md` and fill any `TODO(builder):` lines.
Then submit it one of three ways:

1. **Self-serve (best):** run the committee's **cookbook-update skill**
   (https://lukeallpress.github.io/agentic-builders-commons/cookbook-update-SKILL.md) —
   the sections of this packet map 1:1 onto the Coda columns it writes
   (`Description_Rich`, `Tags`, `Prereqs`, `Prereq_Details`, `Data_Needs`, `Setup_Steps`,
   `Sample_Prompt`, `Repo_Link`, `Credit` = the builder), with
   `Review_Status` = `needs_review`.
2. The committee's project form.
3. Email it to the organizer.

The committee reviews every new submission before it appears in the cookbook.
