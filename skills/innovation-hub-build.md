---
name: innovation-hub-build
description: "Build a district staff hub: one sign-in, pages gated by role, SIS context assembled at conversation speed. The invariants, the access model, the catalog rule, staged build prompts, and the failure modes already paid for."
credit: Luke Allpress, Agua Fria Union High School District
see_also: cookbook recipe `district-staff-hub`; demo at https://innovation.aguafria.org/demo-mode/
---

# Build a district staff hub

This is the build guide for Agua Fria's Innovation Hub, written because the cookbook recipe is one
screen and the thing itself is sixteen pages, two hundred views and about 126,000 lines of Python.
Hand this file to your coding agent. It is organised so you can stop after Stage 4 and still have
something real.

**What the Hub is.** One web app behind one district sign-in. Each page is built around a decision a
staff member actually has to make, with the surrounding context on the same screen — a discipline
referral shown next to the student's current grades, so an administrator can move from the incident
to the cause without leaving the page and without four silent minutes of clicking. Coaches get
eligibility before the bus leaves. Case managers watch a caseload without opening eleven
gradebooks. Teachers see their own students' grades in other classes and log a contact home that
lands back in the student information system.

**The data is not missing; the assembly is missing.** Everything a dean needs is already in the
SIS, spread across screens nobody can reach mid-conversation. The Hub is an assembly layer, and the
assembly has to happen at conversation speed.

---

## 0. What not to copy

Do not try to reproduce two hundred views. The codebase does not transfer — the shape does. What is
worth taking, in order of value:

1. **The catalog rule** (Section 2). Steal this even if you build nothing else.
2. **The five invariants** (Section 1). Get these wrong and you will rebuild.
3. **One page, built around one decision** (Stage 4). This is your pilot.

Everything after that is a menu (Section 6), not a plan.

Also: your SIS is not ours, your role names are not ours, and your regulatory position is not ours.
Where this guide states a rule, the rule transfers. Where it states a value, re-derive it.

---

## 1. The five invariants

These are in order because each one is cheaper now than later, and the first three are the ones
that bite hardest when retrofitted.

### 1.1 Identity in front of the app, not inside it

A small reverse-proxy front door authenticates against the district's shared staff sign-in and
forwards to the application on loopback, passing identity in request headers. Three details make it
safe rather than a hole:

- The front door **strips every inbound identity header** before forwarding, so a client cannot
  assert its own identity.
- It adds a **shared secret header**, and the application refuses any request without it. A request
  that arrives directly at the loopback port gets a 403, not a session.
- The application **refuses to start** when the shared secret is unset, instead of starting wide
  open. Its readiness endpoint reports unhealthy for exactly that condition.

Why a front door rather than auth in the app: the app stays out of the OAuth client registration
entirely, one small component holds the redirect, and every app on the same origin reads the same
identity-only cookie. When the district's sign-in changes, one component changes.

### 1.2 Authorisation resolved per request, server-side, deny by default

Identity goes in the cookie. **Authorisation does not.** On every request, re-resolve the signed-in
person's access names from the database. No roles in the cookie, no roles cached, and never a role
read back from anything the browser sent.

This is the invariant that has actually been violated here. Early pages resolved identity correctly
from the session and then overwrote the username with a value out of a client-side store, and every
permission check downstream trusted it. It was not carelessness in one careless page: a
2,400-line module that validated its inputs properly and was not injectable still scoped campus
data from the browser. Diligence does not close this gap. One server-side choke point does.

A person removed from the active staff list loses access on their next click, because the lookup is
not cached. A Google or Entra account proves someone is in the district; it does not prove they are
staff today.

### 1.3 Read a reporting copy, never the live SIS

Pages read a reporting warehouse fed by nightly and hourly exports. A dean opening a referral must
never put load on the system that registers students.

Two consequences people miss:

- **Know each table's grain before you trust a count.** Discipline rebuilt daily needs collapsing
  to one row per incident before counting. An observation store with one row per question per visit
  will multiply your visit count by the number of questions.
- **A rebuilding source is unreachable, not empty.** Rendering an empty chart during a rebuild tells
  an administrator there are no failing grades. Say the source is unreachable.

### 1.4 One write path

Every mutation goes through a single wrapper function rather than scattered calls. This is what
makes it possible to say honestly that a preview or impersonation session cannot write: the wrapper
refuses, so no individual page has to remember to.

Writes that belong back in the SIS — a logged phone call home, a cell-phone incident — go there, so
the record lives where the district already keeps it. Everything else the hub records (notes,
uploaded evidence, access grants) lands in its own database.

### 1.5 Student data never reaches a hosted model

No student identifier goes to a hosted model, aggregate or not. Anything that reasons over student
records runs against a self-hosted model.

And in the repository: no student names in code, tests, fixtures or commit messages — student
numbers only. CI fails a pull request that adds one. (We have exactly one tracked CSV of student
data that predates the rule. It is a known, unremediated problem, and naming it here is cheaper
than pretending otherwise.)

---

## 2. The catalog rule

**A page, tab, sub-view or report is not done until its entry says what question it answers, in the
words of the person who would ask it, at the grain they would ask it.**

"Gradebook data" is not a question. "Which teachers have not entered a grade in the last two weeks?"
is a question. "How do I log a parent contact?" is also a question — task questions count.

This is the single most valuable thing in the build, and it costs almost nothing to adopt on day
one. At sixteen pages nobody — including the person who wrote them — can remember where anything
is. The catalog is one YAML file per page, and it is the *source*: three artefacts are generated
from it and committed, so they cannot drift.

| Generated | For |
|---|---|
| a flat index (JSON) | a bot or an MCP tool: one entry per place in the app, with click path, link, audience, and the questions it answers |
| a single readable page | people: "where do I find X?" |
| surface records in the district ontology | the organisational map, imported from the index |

A per-page entry looks like this:

```yaml
key: data-analysis              # the page's ACCESS_KEY; file is <key>.yaml
kind: page                      # page | shared | job
name: Data Analysis             # the label staff see in the sidebar
path: /data-analysis
module: pages/data-analysis.py
sidebar: true                   # false: reached by a link, not listed
summary: >
  One or two plain sentences: what the page is for.
audience:
  roles: [Admin, CertNonInst, DOAdmin, DC]   # exactly the page's access list
  grantable: true               # a manager can grant it per person
  scope: >
    Whose data a user sees: their own campus, the site in the selector, only
    their own students.
views:
  - key: gradebook-auditing     # what follows ?t= in the link
    name: Gradebook Auditing
    click_path: [Formative, Gradebook Auditing]
    link: /data-analysis?t=gradebook-auditing
    shows: >
      What is on screen: the rows, the measures, the time window.
    filters: [School, Department, Grading period]
    actions: [Download CSV, Email the teacher]
    questions:
      - Which teachers have not entered a grade in the last two weeks?
    keywords: [gradebook, audit, grade entry]
    source: [gbk/auditing.py:AUDIT_DEFINITIONS]
    views: []                   # sub-views, same shape
findings:
  - Anything that looks wrong, with file and line. Stays in the repo.
```

**Make a test hold it**, or it will not survive contact with a deadline. Ours fails when a
registered page or tab has no entry, when an entry names a tab the code no longer has, when the
roles in an entry differ from the page's actual access list, when a view has no question, and when
the generated files are stale.

**What never goes in the catalog**, because staff and hosted models both read it: no person's name,
no student data or example rows, and no infrastructure outside the one `source` field — no table,
view, procedure, server, database or account names. Write "the gradebook snapshot", not its object
name. `findings` stay in the repository and are never generated out, because some of them describe
gaps in who can see what.

Five to fifteen questions per view is normal. One honest question beats five vague ones.

---

## 3. The access model

Keep it this boring:

- **Access names come from the job title** on the person's staff record. A page lists the access
  names that may open it; a person opens it when they hold one of them.
- **A manager can grant one person a whole page, or a single tab**, by name. A tab grant also puts
  the page in that person's sidebar, where they see only the tabs they were granted.
- **Campus staff see their own campus.** District-office staff, and staff assigned to more than one
  school, choose a school from a selector. An administrator can grant one person an extra school.
- **Not on the active staff list means no access at all.**

Write down what each access name means and where it comes from, in plain words, next to the code
that computes it. Never invent an access name that the code does not use — and keep the catalog's
role list and the page's access list identical, with a test to prove it.

---

## 4. Build order, with prompts

Each stage is a prompt you can paste. Run them in order; do not skip to Stage 4.

### Stage 1 — the shell

> Build me a staff web app shell: one sign-in, a left-hand list of pages, and a page registry where
> each page declares the access names that may open it and a stable key. The landing page shows one
> card per page the signed-in person can open, with a line saying what that page is for. Add a
> school selector in the top right that is visible only to people assigned to more than one school.
> Nothing else yet.

The reason to build one hub rather than a dozen apps is organisational, not technical: staff learn
one address, and every project you ship afterwards lands somewhere people already look.

### Stage 2 — the front door

> Put a reverse-proxy front door in front of this app. It authenticates against our existing shared
> staff cookie and forwards to the app on loopback with identity in request headers. Strip every
> inbound identity header before forwarding so a client cannot assert its own. Add a shared secret
> header that the app requires on every request, and make the app refuse to start if that secret is
> unset rather than starting open. Add a readiness endpoint that reports unhealthy for exactly that
> condition. Then show me what a request sent directly to the loopback port receives.

That last sentence matters. A guard whose success path looks identical to its absence has to be
proven to fire.

### Stage 3 — role resolution

> Resolve the signed-in person's access names from the database on every request. Deny by default.
> Do not put roles in the cookie, do not cache the lookup, and never read a role back from anything
> the browser sent. Add per-person grants: a whole page, or a single named tab, which also reveals
> the page in that person's sidebar with only the granted tabs visible. Then write the tests that
> fail if a page renders for someone holding none of its access names, and if a tab renders for
> someone granted a different tab.

### Stage 4 — the first page, built around one decision

> Build one page around a single decision somebody has to make. A discipline referral, shown
> alongside that student's current grades, attendance and intervention tier, with each grade
> linking into the gradebook, and the whole page restricted to the roles entitled to see discipline
> records. Read from our reporting warehouse, not the live SIS. Before you write a query against a
> table, tell me its grain and show me why your count is not double-counting. If a source is
> mid-rebuild, say the source is unreachable — never render an empty chart that reads as "no
> failing grades". Route every write through one wrapper function. Then write the page's catalog
> entry: what it shows, who can open it, and the questions it answers in the words a person would
> use.

Stop here if you need to. One page that assembles context at conversation speed is worth more than
six pages nobody trusts.

### Stage 5 — demo mode, before you show anyone

> Add a demo mode with every student record replaced by generated data, reachable at its own route,
> and make every page work in it. Then tell me which pages you could not verify.

Demo mode is easy to half-build, which is how a demo ends up on blank pages in front of an
audience. Ours is public: <https://innovation.aguafria.org/demo-mode/>

### Stage 6 — the second page, and the usage log

> Add one usage-log call per meaningful page or tab view, with stable labels and nothing sensitive
> in the telemetry. Then build the second page from the menu, and make the build fail if its
> catalog entry is missing or stale.

---

## 5. Failure modes already paid for

Four of these cost us real outages. They are not hypothetical.

**Authorisation read from the browser.** Covered in 1.2. The fix is one server-side choke point, not
more care.

**A web garden multiplies every query.** Three worker processes each running a query at degree of
parallelism 8 put 24 workers on 12 schedulers: 91–96% CPU, 0% idle, 48 rows a second, and the
parallelism-wait signature that means thrash rather than work. The cause was somebody loading a
slow page and refreshing twice. **Set the application pool to a single worker process**, and cap
query parallelism below the cores per NUMA node. A web garden also runs every background worker
once per process, which is its own surprise.

**An analytics page over your own write-heavy log table will deadlock the whole app.** A six-month
scan with JSON extraction per row, filtering on the computed result so nothing pushed down, ran 66
minutes, escalated to a table-level shared lock, and every page-load insert queued behind it
forever, because the lock timeout waits forever. Every environment stalled — including developer
laptops, whose inserts sat in the same queue. Fixed with read-committed snapshot isolation, a
covering index, and a nightly rollup.

**Shipping the fix is not landing the fix.** Verify in the environment, not in the diff.

**A rebuilding source rendered as empty.** See 1.3. This one is a data-integrity bug that looks
like a UI nicety.

---

## 6. The menu: what sixteen pages turned out to be

Pick the two or three your district would actually use. Each line is the decision the page serves.

| Page | What it answers |
|---|---|
| **Home** | What can I open, and what is each of these for? |
| **Intervention** | The campus workbench: open discipline referrals student by student, campus-wide attendance, and who needs follow-up. |
| **Teacher Dash** | A teacher's own rosters, one screen per period: each student with support badges and their current grade in every class they take. |
| **Teacher Dash – Admin** | The same roster for any teacher on campus, seen the way that teacher sees it. |
| **Team Dash** | The PLC view: teachers of the same course side by side — grade distributions, how recently grades were entered. |
| **Data Analysis** | The reading room: current and posted grades, gradebook health, discipline, attendance intervention, school-improvement progress. |
| **Assessment Center** | Class-by-class results on the district's own assessments. |
| **Academy Dash** | An academy team's intervention board: grades, attendance, referrals, risk level. |
| **Case Manager Dash** | One caseload in one place, plus a one-page summary for any student on it. |
| **Coaches Corner** | Grades and attendance for a team, club or program, so eligibility is known before the bus leaves. |
| **Pop In** | Log a short classroom observation against the district rubric; optionally email the teacher. |
| **Directory** | Find a colleague; update your own room, phone and photo. |
| **Toolbox** | Shared links to outside tools, scoped to you, your school, or your role. |
| **Data Agent** | Ask a question about district data in plain words, answered only from approved, scoped data. |
| **User Management** | Grant and revoke a page, a tab or an extra school, per person, with a record of who did it. |
| **Usage Audit** | Is anyone actually using this, and which parts? |

Alongside the pages sit shared tools that open from several places (a one-page student summary, an
attendance case workspace, a contact log, the referral workflow) and scheduled jobs that send mail
or rebuild stored summaries overnight. Catalogue those too; a tool that arrives by email is still a
place a person can open.

**The one-page student summary is the highest-leverage single screen.** Everything about one student
on one screen, opened from anywhere. If you build one shared tool, build that.

---

## 7. Definition of done

A page is done when:

- it renders for everyone holding one of its access names and for nobody else, proven by a test;
- its catalog entry names what it shows, who can open it, and the questions it answers;
- every count it displays has a stated grain;
- an unreachable source says so instead of rendering empty;
- every write goes through the wrapper;
- it works in demo mode;
- and the generated catalog artefacts are regenerated and committed in the same change.

---

## A note on scope and age

The Hub's oldest modules carry file headers dated November 2021; it was assembled out of separate
pieces before it was a hub, and it predates any AI assistance by years. It is now roughly 126,000
lines of Python across 291 modules with 95 test modules, and it ships from the main branch with no
release tags.

That history is the honest argument for this document. You are not going to reproduce it, and you
should not try. Take the invariants, take the catalog rule, build one page around one decision, and
let the rest arrive when somebody asks for it.