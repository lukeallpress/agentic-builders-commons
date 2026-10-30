---
name: sdpc-dpa-workflow
description: >-
  Boise School District playbook for student data privacy agreements (DPAs) in the SDPC Resource
  Registry, with TEC working Boise's requests. Use when asked whether students can use an app or
  website, whether a vendor has a DPA or Exhibit E, how to piggyback (Auto Exhibit E Creator),
  file a New Request or assign TEC, add a missing product, handle "no data collected" apps or
  vendors that will not sign, or check expiring agreements. Also for sdpc.a4l.org, A4L, NDPA,
  Idaho Exhibit G, sdpc-pp-cli.
---

# SDPC DPA Workflow (Boise School District)

*Shared by Ryan Gravette, Boise School District (Idaho), on the Agentic Builders skills shelf. It
is Boise's working playbook, edited for sharing: the district ID (15032), state (ID), and TEC (the
consortium that works Boise's requests and holds the Idaho add-on) are Boise's values; swap in your
own. District-internal references (team roster, counsel's vendor reviews, the parent-release
samples) are not included. The `sdpc-pp-cli` tool is a Printing Press CLI described in the
matching cookbook recipe; it is not yet in the public library.*

This skill takes an app request from "can we use this?" to a recorded decision in the SDPC
Resource Registry. It follows the process TEC trained the Boise team on in September 2026 and
TEC's public quick reference guides, and it uses the `sdpc-pp-cli` tool for the parts the tool
can do.

Boise is district **15032** in the **Idaho (ID)** alliance on https://sdpc.a4l.org. TEC is the
Progress Admin that negotiates Boise's requests with vendors. Boise ranks systems P1 to P4 by data sensitivity (P1: education records, special populations, safety and wellbeing data, student voice, student-facing AI).

## Two things to know before touching the registry

**Sessions expire.** Public pages (Boise's approved-app listing, state templates, signed PDFs)
work any time. Everything under "Your District's Agreements" and "Your Resources" needs a
signed-in session, and that session can lapse quickly: TEC says roughly an hour idle, but on
2026-09-26 it expired about ten minutes after sign-in. Do gated reads promptly after signing in. An expired session comes
back as a page titled **SDPC: Error** saying it is "unable to login to your account." The CLI
reports this inconsistently (exit 5 with `SDPC: Error` from lookups, exit 4 from Manage
Agreements reads and previews), so read the message rather than the exit code. When it happens,
**launch the login window yourself** as described in `references/cli.md` (Signing in): run
`press-auth login` in the background with `--force`, tell the person the window is up, and pick
the task back up when the job finishes. Do not paste commands for the person to type, and do not
type a password or sign in for them. The person's only job is to sign in inside the window.

**press-auth signs in through its own separate Chrome window.** Signing in to sdpc.a4l.org in
your normal Chrome does not refresh the CLI's session, so the CLI can keep failing while the
browser looks fine. `press-auth list` shows when each session was captured, which is how you
spot a stale one.

**Always preview before a write, even when the person already said "submit."** Run the
`--dry-run` first, then the `--yes`. Claude Code's permission classifier blocks a registry write
that was not previewed in the same session (seen 2026-09-27 on `request assign`). A clear "submit"
from the person counts as the yes; the preview is still required.

**Check the vendor's record nationwide.** `scripts/sdpc national by-resource --software-id N
--json --no-input` lists every district's row for a product. It shows whether the vendor has
signed TEC's multi-state NDPA (Musicca had, so TEC only needs Idaho added), signs elsewhere under
state agreements (YouUplift), or refuses everywhere (Marq). Report this in triage; `triage.py`
does not check it yet.

**Registry writes are shared records.** Creating an Exhibit E, filing a New Request, assigning
TEC, changing a status or agreement type, and requesting a new resource all put Boise's name on
a record other districts and TEC can see, and some create work for TEC or SDPC staff.

The CLI now does all of these, and every write command shares one safety contract: it previews by
default, `--yes` is required to submit, and under `--agent` or `--no-input` a write without
`--yes` is refused rather than submitted silently. Use it. Run the command with `--dry-run`, show
the person what it will do in plain words, wait for a clear yes, then re-run with `--yes`. One
write per confirmation. Two registry steps are still browser-only: the later steps of **Change
Agreement Type** (Path E) and the **logo upload** on Request to Add New Resource.

## Step 1: Triage the request

Run the triage script with the name the requester used:

```bash
python3 scripts/triage.py "Blooket"
python3 scripts/triage.py "Blooket" --json      # for chaining
```

Without signing in, it checks Boise's public approved-app listing, the Massachusetts Exhibit E
list and Boise's prioritized app inventory (both copied into `assets/data`), and Google Workspace
sign-in counts (the newest `~/Downloads/owl_apps_export*.xlsx`, else the older CSV copy in
`assets/data`). If a
session is live, it also checks the registry catalog, Idaho's active agreements, and Boise's own
Manage Agreements rows. Each check reports clearly when its source is missing or the session has
expired, so a partial run is still useful.

Then answer the one question no script can: **does the product collect student personally
identifiable information (PII)?** These all count: rostering, any student login (Sign in with
Google included), student work or content stored by the vendor, and student voice, images, or
video. For students under 13, which is every Boise K to 6 student, COPPA also treats persistent
identifiers such as IP addresses and device IDs as personal information. A web game students
join with only a nickname therefore still collects PII from them. An app students open on a
device with no account, where nothing about the student leaves the device, does not.

Check the vendor's privacy policy and, for iPad apps, the App Store privacy label. Say what you
found and where, rather than guessing. TEC's "What is PII?" guide lists the FERPA direct and
indirect identifiers; see `references/registry-navigation.md`.

Choose the path:

| What triage and the PII question show | Path |
|---|---|
| Already approved for Boise and not expired | Report the agreement link, its limits, and which account types it covers. Done. |
| A Boise request for it is already open | Report its Progress stage. Do not file a duplicate. |
| Boise's agreement for it expired | **Renewal**: see "Monthly renewal check"; try Path B, then C |
| Boise's earlier request was Declined, Vendor Unresponsive, Not Approved, or Did Not Pursue | **E. No DPA available**, unless something changed (new terms, a new Exhibit E, a different tier) |
| Collects no student PII | **A. Approved (No Data Collected)** |
| Collects PII and an Exhibit E is available to Boise | **B. Auto Exhibit E Creator** |
| Collects PII, no Exhibit E, product is in the registry | **C. New Request, then assign TEC** |
| Collects PII and the product is not in the registry | **D. Request to Add New Resource**, then C |
| Vendor declines, stalls, or offers consumer terms only | **E. No DPA available** |

Only Path C assigns TEC. Paths A and E add the product with Add New Request but leave Progress
Admin empty; assigning TEC there asks TEC to negotiate something that needs no negotiation.

The Massachusetts list is a lead list. It shows where a signed Exhibit E probably exists and
gives the exact registry name to search for, but it does not prove Boise can use it. The Auto
Exhibit E Creator is the test.

Report the P1 to P4 segment triage returns. It tells the team how much care the request needs:
P1 systems (education records, special populations, safety and wellbeing data, student voice,
student-facing AI) should get counsel review before anyone tells a teacher yes.

Until a path is complete, tell the requester not to start with students. If they need something
now, point them to a tool Boise already supports, or to teacher-only use with nothing about
students entered.

## Path A: Approved (No Data Collected)

TEC does not seek a DPA for products that collect no student PII, for example an iPad app
students simply open and use like stand-alone software. TEC's quick reference guide
"Setting a Resource as Approved: (No Data Collected)" gives these steps:

1. Confirm the no-PII finding from the product itself: no account, no rostering, nothing synced
   to the vendor's cloud. For iPads, the App Store privacy label and, if in doubt, App Privacy
   Report on a test device show what the app sends. Note what you checked.
2. Signed in, use **Add New Request** to add the product to Boise's section. **Do not assign
   anyone as Progress Admin.** The product appears in Manage Agreements as **New Request**.
3. In **Manage Agreements**, click **Edit Other Info** in the first column of that row.
4. About three quarters of the way down, find **Status** and choose
   **Approved (No Data Collected)**.
5. Optionally, add conditions in **Agreement Restrictions** (for example "offline drawing only;
   iCloud and sharing disabled in Mosyle").
6. Click **Save Info**.

The CLI does steps 2 to 6. `request submit` files the request (no `--assign-tec` on this path),
and `request status` is Edit Other Info:

```bash
scripts/sdpc request status --data-id 123456 --approved-no-data \
  --restrictions "offline drawing only; iCloud and sharing disabled in Mosyle" \
  --district-id=15032 --state=ID --dry-run
# show the preview, get a yes, then re-run with --yes
```

Revisit the decision if the vendor later adds accounts, cloud sync, or AI features. If the vendor
offers its own data privacy addendum, do not sign it without counsel; see "Read the agreement,
not just the status" below.

## Path B: Auto Exhibit E Creator (piggyback)

An Exhibit E is a vendor's general offer to extend an existing signed DPA to other districts.
This is the fastest path when it exists.

The CLI runs this whole path: `exhibit-e list` finds the offers, `exhibit-e create` creates one,
and `exhibit-e verify` is TEC's after-check.

```bash
scripts/sdpc exhibit-e list --query "Blooket" --district-id=15032 --state=ID --agent
scripts/sdpc exhibit-e create --data-id 123456 --district-id=15032 --state=ID --dry-run
# show the preview, get a yes, then re-run with --yes
scripts/sdpc exhibit-e verify --resource "Blooket" --district-id=15032 --state=ID --agent
```

The steps, in the browser or through the CLI:

1. Get the exact registry resource name first. The creator search is exact-match only:
   partial names no longer work, case does not matter, and special characters and numbers must
   match (`"i-Ready"`, `"Soundation4Education"`, `"Whooo's Reading"`). Triage lists candidate
   names from the Massachusetts list; the registry's View All Resources page is the authority.
   Some vendors register tiers separately ("Blooket" and "Blooket Plus"), so try each name.
2. Signed in, open **Your District's Agreements > Auto Exhibit E Creator**, or run
   `exhibit-e list`.
3. Search with the name **inside double quotes**. `--query` adds the quotes for you and reports
   what it sent as `query_used`. Rows come back as `available` or `already_signed`, each with its
   `data_id`. If nothing comes back, try the other exact names, and check **Your Resources > View
   Company Successor List** in case the company was renamed or acquired.
4. Before creating, open the originating agreement PDF and read it, including any vendor rider.
   A piggyback inherits those terms.
5. Create the Exhibit E (`exhibit-e create --data-id N`). It is signed by the district's
   designated **Exhibit E Signer**, a registry account role that TEC says each district should
   give to exactly one person, usually the District Main admin. Confirm who holds that role if it
   is not you.
6. Open **View Prepared Exhibit Es**, or run `exhibit-e verify`, and check that Boise's district
   information and the electronic signature both appear. TEC calls this check out specifically,
   and `verify` fails when a row is present without a signature.

If the creator has nothing for the product, go to Path C. An Exhibit E on another state's list is
a lead, not coverage; see `references/vendor-outreach.md`.

## Path C: New Request, then assign TEC

1. Signed in, open **Your District's Agreements > Add New Request**.
2. Pick the product from **Select a Resource**. If it is not in the list, go to Path D.
3. Set **Status** to **New Request**.
4. Under **Internal Data**, enter the name and email of the Boise team member filing it. TEC
   contacts this person. Click **Save Info**.
5. Open **Manage Agreements**, find the New Request row, set **Progress Admin** to **TEC**, and
   click **Update**. TEC is notified electronically only after this step, so a request without
   it sits unseen.

The CLI does all five steps. `request submit` is steps 1 to 4 and reports the new row's
`data_id`; `request assign` is step 5 and verifies it by re-reading the page.

```bash
scripts/sdpc request submit --resource "Blooket" --district-id=15032 --state=ID \
  --requestor "Your Name" --requestor-email you@district.org \
  --grade 4,5,6 --content-area "Educational Technology" \
  --purpose "Classroom review games" --dry-run
# show the preview, get a yes, then re-run with --yes
scripts/sdpc request assign --data-id 123456 --dry-run
# preview again, get a yes, then re-run with --yes
```

`request submit --assign-tec` chains both writes. It is off by default and belongs to Path C
only. Running the two writes separately, with a preview and a yes for each, is the safer habit.

For `assign`, pick the row by its **data ID**. `--resource` and `--software-id` take the first
matching row on the page, which can be an older or expired row for the same product; the preview
shows which data ID it picked, so check it. Previews need a live session. Progress Admin options
are TEC plus the district's registry users.

If the product carries student communication, or the vendor's entity, OAuth scopes, or terms raise
questions, send the question set in `references/vendor-outreach.md` so TEC negotiates with the
answers in hand.

## Path D: Request to Add New Resource

SDPC now requires full product details before it adds anything to the catalog.

1. Rule out a different name first: search **View All Resources** with other spellings and
   check **View Company Successor List** (Quizizz became Wayground, for example).
2. Open **Your Resources > Request to Add New Resource** and complete every field. The public
   request form asks for the requester email, company name, product name, website, terms of
   service URL, privacy policy URL, and a logo image. Use full URLs.
3. Submit only for products Boise actually intends to request. SDPC staff review each one.
4. SDPC emails Boise's Main Admin when it is approved. Then sign back in and follow Path C.

The CLI does step 2 with `request new-resource`. Use `lookup resource --query "<name>"` and
`reference successors` for step 1 first.

```bash
scripts/sdpc request new-resource --company "<legal name>" --name "<product>" \
  --website https://... --terms https://... --privacy https://... \
  --district-id=15032 --state=ID --dry-run
# show the preview, get a yes, then re-run with --yes
```

The logo field is a file upload the CLI leaves empty. If SDPC asks for a logo, add it in the
browser.

## Path E: When no DPA is available

A vendor may decline, go quiet, or offer only consumer terms. In rough order of preference:

1. **Use an approved alternative.** Check Boise's listing and the district's 2026-27 "Same
   Learning Goal, Updated Tool" guidance for a supported tool that does the same job.
2. **Keep it teacher-only** with nothing about students entered.
3. **Guardian release**, only where use is optional and real alternatives exist. TEC cautions
   districts about relying on parent consent for products without DPAs and leaves the decision
   to each district, so this needs a named decision-maker and counsel review. Follow
   `references/parental-consent.md`; the Cambridge samples in `assets/` are the model TEC
   shared. Record it in the registry as TEC's guide describes: Add New Request **without**
   assigning TEC (`request submit`, no `--assign-tec`), then in Manage Agreements click **Change
   Agreement Type**, choose **Other Type of Agreement**, set **Agreement Type** to **Legal Drafted
   Guardian Release**, and click **Save Agreement Info**. **The Change Agreement Type steps are
   browser-only.** Its later pages were never captured, so no CLI command exists for them.
4. **Decline** and tell the requester why in plain terms. Record it with the matching Status
   through `request status --data-id N --status "<label>"` (Declined needs `--declined-reason`;
   also Vendor Unresponsive, Not Approved, Did Not Pursue), or set the matching Progress stage
   with `request assign --data-id N --progress "<stage>"`.

When the sticking point is unanswered questions rather than a flat no, work
`references/vendor-outreach.md` before landing on one of these.

TEC has shared counsel reviews of some large vendors (ChatGPT, Adobe) with member districts; ask TEC for them before advising on those vendors.

## Read the agreement, not just the status

An "Approved" row or an available Exhibit E can still leave gaps. Before telling anyone a product
is covered, check:

- **Which accounts it covers.** Some vendors' DPAs cover only district-paid accounts, not free or
  teacher-paid ones. Match the tier and account type the teacher will actually use.
- **Vendor riders** attached to signed agreement PDFs, such as damages caps or narrower scope.
  These can conflict with Idaho Exhibit G and should go to counsel.
- **Vendor-supplied addenda** offered instead of the NDPA. Some shift parental consent,
  indemnification, or governing law onto the district. Do not recommend signing one without
  counsel.

## Monthly renewal check

TEC asked every team member to set a monthly reminder to check expiring agreements.

- Browser: **Your District's Agreements > View Expiring Agreements** (the site's window is 90
  days). TEC's video "How to Use View Expiring Agreements to Manage DPA Renewals" walks through
  it.
- CLI, signed in: `scripts/sdpc my-district expiring --district-id=15032 --state=ID --agent`
- The CLI's `renewals --within 180d` needs the Premium API sync, which is not set up yet.

If the person wants the reminder automated, offer to create a monthly scheduled task. Ask
first, because it is a standing configuration.

## Reporting back

Lead with the answer. Use this shape so the team can paste it into a ticket or email:

```
<Product>: <one-line status, e.g. "Not yet approved. Exhibit E likely available.">
Path: <A to E and its name>   Sensitivity: <P1 to P4 or "not in inventory">
Students: <grades and ages; whether COPPA applies (under 13); any under-18 consent in vendor terms>
Why: <two or three sentences from the triage facts and the PII finding>
Next steps:
1. <action> (<who>)
2. ...
For the requester: <two or three plain sentences a teacher can act on today>
Open questions: <only if any>
```

Keep the requester note plain and short. Tell them what they can do now, including an approved
alternative, rather than only what they cannot do. When you summarize a legal review or an
agreement, say only what the document says; do not add rankings or conclusions it does not
make. Write without em dashes or stock AI phrasing; the district prefers a plain workplace voice.

## Guardrails

- Say an app is approved only when the registry shows it approved for Boise. A Massachusetts
  signature, a ClassLink tile, a trusted Google OAuth setting, or a vendor's marketing is not
  approval.
- Frame FERPA, COPPA, the National School Lunch Act, and Idaho Code 33-133 points as items for
  counsel to confirm. This skill produces planning guidance, not legal advice.
- Treat parental consent letters and release forms as drafts until counsel reviews them.
- Never file a duplicate request; check Manage Agreements first.
- Never sign in, reset a password, or enter credentials for anyone.
- Keep student names and other identifiers out of registry notes, CLI `teach` queries, and
  anything that could be shared outside the team.

## Scripts and references

Not included here, described so you can rebuild them:

- `scripts/sdpc`: a 30-line wrapper that finds the `sdpc-pp-cli` binary when PATH differs (launchd).
- `scripts/triage.py`: runs the public and session checks listed in Step 1 against the district's own inventory files and prints the P1 to P4 segment, candidate registry names, and sign-in counts. Replace the inventory inputs with your own.
- `references/parental-consent.md` and the Cambridge sample letters, `references/legal-reviews.md`, and `references/boise-context.md` are district-internal.

The two general references the skill cites are appended below: **Registry navigation** and **The CLI in this workflow**, followed by **Vendor outreach**.

## Appendix A: Registry navigation (sdpc.a4l.org, Registry 1.0)

Source: TEC's "Workflow for Building a District's Repository of DPAs" (received 2026-09-15),
TEC's public quick reference guides at https://tec-coop.org/data-privacy/training/ (read
2026-09-17), TEC's training email, and Boise's signed-in pages captured on 2026-09-15
and 2026-09-17. The Auto Exhibit E Creator, Add New Request, Edit Other Info, and Request to Add
New Resource pages were captured on 2026-09-17 and the CLI now drives them; see
`cli.md`. The later steps of **Change Agreement Type** were not captured, so those steps below
come from TEC's written guide. Confirm those field labels the next time someone is signed in.

The landing page says **SDPC Registry V2** is coming. Everything here describes 1.0. Expect menu
names to move when V2 arrives.

### Contents

- Signing in and the error page
- User roles
- Signed-in menu map
- TEC's four-step workflow
- Approved (No Data Collected)
- Approved with Guardian Release
- Search rules
- Manage Agreements: columns, row links, Progress stages, Progress Admins
- What counts as PII
- Quick vetting look-fors
- Other useful pages
- TEC training resources

### Signing in and the error page

Sign in at https://sdpc.a4l.org/login.php with a registry email and password. Each team member
needs their own account, added under **Account Management > Manage District Users**.

A lapsed or missing session shows a page titled **SDPC: Error** with the text "An error has
occurred and we are unable to login to your account." It lists wrong login details and
unregistered users as possible reasons, but after about an hour idle it almost always means the
session timed out. Sign in again.

### User roles

From TEC's "Configuring District User Accounts" guide:

| Role | Can do |
|---|---|
| **District Main** | Everything, including Account Management. Usually the primary District Admin, typically the technology director. Only District Main users can add other users. A district may have a second one as backup. |
| **District (Additional Users)** | Everything except Account Management: add and renew DPAs, but not add users or change district settings. This is the role for most of the Boise team. |
| **Exhibit E Signer** | Signs Exhibit Es for the district. TEC says each district should have exactly one. Often the same person as District Main; a district may designate someone else. |
| **Teacher/Staff Member** | View only. |

TEC offers help configuring accounts.

### Signed-in menu map

**Your District's Agreements**
- Manage Agreements (the working list; Progress Admin and Progress stage are set here)
- Add New Agreement (records an agreement Boise signed outside the registry; not covered in TEC's
  training, so check with TEC before using it)
- Add New Request
- View Expiring Agreements (90-day window)
- View Expired Agreements
- View Active State DPAs
- View All State DPAs
- View Statewide New Requests
- View Contracts in Negotiation
- Auto Exhibit E Creator
- View Prepared Exhibit Es

**Your Resources**
- Manage Resources
- Request to Add New Resource
- View All Resources
- View Company Successor List (renamed and acquired companies)
- View Statewide New Requests

**Tools**
- Bulk Export Exhibit Es
- Export District Data to Excel
- Customized Resource Listings (the public approved-app page)
- Digital Resource Request Form (the public form staff can use to request a product)
- Bulk Email Tool

**Account Management**
- Change Password, Edit User Account Info, Edit District Account Information
- Manage District Users, Manage Progress Workflow, Add Additional District Account

**Help**
- Contact Support, FAQs, Report an Issue, User Trainings, Registry Resources

### TEC's four-step workflow

TEC's instructions assume the person is already trained on the registry and working from the
first menu, Your District's Agreements.

1. **Sign in** at https://sdpc.a4l.org.
2. **Auto Exhibit E Creator.** Piggyback on an existing DPA. After creating it, open the Exhibit E
   and confirm all of Boise's district information appears, including the electronic signature.
   If no Exhibit E is available, go to step 3.
3. **Add a New Request.** Pick the product from the **Select a Resource** drop-down. Next to
   **Status**, choose **New Request**. Under **Internal Data**, enter your name and email as the
   requestor and click **Save Info**. Then, under **Manage Agreements**, find the New Request,
   set **Progress Admin** to **TEC**, and click **Update** so TEC is notified electronically.
   If the product is not in the drop-down, go to step 4.
4. **Request to Add a New Resource.** SDPC requires all the pertinent product details. SDPC
   reviews the request, adds approved products to the catalog, and emails the district's Main
   Admin. Then sign back in and do step 3.

**Product-specific instructions.** TEC keeps a list of products with special handling, taken
from past TEC newsletters. The link is in the same email. Check it before filing for a big-name
vendor.

### Approved (No Data Collected)

TEC's guide "Setting a Resource as Approved: (No Data Collected)" covers resources educators can
use without any transfer of PII.

1. Sign in.
2. Use **Add New Request** to add the resource to Boise's section, but **do not assign anyone as
   Progress Admin**. The resource appears in Manage Agreements with status **New Request**.
3. In **Manage Agreements**, click **Edit Other Info** in the first column of that row.
4. About three quarters of the way down, find **Status**.
5. Change it to **Approved (No Data Collected)**.
6. Click **Save Info**.

Optional: add conditions of use in **Agreement Restrictions** on the same page.

The Progress list also has a **No Data Collected** stage, but TEC's guide sets the **Status**
field, not the Progress stage.

### Approved with Guardian Release

TEC's guide "Setting a Resource as Approved w/Guardian Release" opens with a caution: TEC advises
districts against relying on parent consent for resources without DPAs, but leaves the decision
to each district.

1. Sign in.
2. If the resource is not yet in Boise's section, use **Add New Request** to add it, but
   **do not assign TEC as Progress Admin**. It appears in Manage Agreements as **New Request**.
3. Open **Your District's Agreements > Manage Agreements** and find the resource.
4. To the left of the resource name, click **Change Agreement Type**.
5. Choose **Other Type of Agreement**.
6. In the **Agreement Type** drop-down, select **Legal Drafted Guardian Release**.
7. Click **Save Agreement Info**.

The guide does not say where signed releases are kept.

### Search rules

For the Auto Exhibit E Creator:

- Exact match only. Partial names no longer work in this section.
- Put the name in double quotes: `"i-Ready"`.
- Spelling, special characters, and numbers must match the registry exactly
  (`"Soundation4Education"`, `"Whooo's Reading"`).
- Case does not matter.

The registry name is often not what teachers call the product ("Canva" is registered as
"Canva for Education"; Quizizz is now "Wayground (Formerly Quizizz)"). Get the name from
**View All Resources**, the Massachusetts Exhibit E list, or `triage.py` before searching.

Entity names can also differ between a vendor's privacy policy and the registry. Fanschool's
policy names Kidblog Inc. while the registry lists Fanschool Inc., and the registry carries
`Fanschool` and `Kidblog` as two separate resources. Check **View Company Successor List** when a
search comes up empty, and resolve the entity in writing before signing; see
`vendor-outreach.md`.

### Manage Agreements

**Columns:** Originating Agreement, Company Name, Resource Name, Status, Agreement Type,
Approved On, Expires On, Vendor's Contact Info, Date Added to Registry, Notes. Each row has its
own Update form for Progress Admin and Progress stage, plus two links TEC's guides use:
**Edit Other Info** (first column; holds **Status** and **Agreement Restrictions**) and
**Change Agreement Type** (left of the resource name).

A product can have more than one row, for example an expired agreement and a new request. Work
from the row's data ID, not its name.

**Progress Admin options:** TEC plus the district's registry users. Use TEC unless TEC says otherwise.

**Agreement Type in use for Boise:** "TEC ID-NDPA V1 with Exhibit E". Both signed agreements
(Axiom Learning, Pixlr for Education) carry it. The signed PDF is the Standard Student Data
Privacy Agreement on the **Idaho ID-NDPA, Standard Version 1.0** form, between "Independent
School District of Boise City #1" and the provider, executed through DocuSign.

**Watch the Expires On column.** An agreement recorded with the **01/01/2099** placeholder never
surfaces in View Expiring Agreements. Axiom Learning carries that placeholder; Pixlr expires
04/28/2028.

**Progress stages.** The drop-down is shared across states. The ones that apply to Boise:

| Stage | Use it when |
|---|---|
| Assigned to TEC | TEC has the request. Usually set by the workflow, not by hand. |
| Use Auto EE Creator | An Exhibit E exists; go create it (Path B). |
| Use EE Creator for Specific Resources | Only some of a vendor's products have an Exhibit E. |
| No Exhibit E | The vendor has no general offer. |
| No Data Collected | A progress marker for no-PII products. The approval itself is the **Approved (No Data Collected)** Status set through Edit Other Info. |
| Contract Sent | The DPA went to the vendor. |
| Signer Action Needed | Waiting on a signature, possibly Boise's. |
| With Legal | Counsel is reviewing. |
| Negotiations | Terms are being negotiated. |
| Extended Review | Review is taking longer than usual. |
| Follow Up | Someone owes a follow-up. |
| No Info | Not enough product information to proceed. |
| Vendor Signed | Vendor has signed. |
| Vendor Unresponsive | Vendor stopped responding. |
| Declined | Vendor declined. The page asks for a **Declined Reasoning** note. |
| Not Approved | District decided not to approve. |
| Did Not Pursue | District dropped the request. |
| DPA Renewed | Renewal completed. |
| Retired | Product no longer used. |
| Delete | Remove the row. Confirm with TEC before using. |
| Invalid Resource Name | The resource is recorded under a wrong or duplicate name. |
| Use Expiring Agreements Tool | Handle this row through View Expiring Agreements rather than here. |

State-specific stages (NH Security Issue, NY Country of Storage, NYS Consortium Contract,
NY RIC Check, NY Legal, OH Learn21, OH Awaiting District Response, MO Request Returned) do not
apply to Idaho.

The site spells one status value "Not Appproved" internally. Match the label on screen.

### What counts as PII

TEC's "What is PII?" guide, based on the U.S. Department of Education's FERPA definition,
counts information in education records that identifies a student directly or in combination.

- **Direct identifiers:** student's name, parents' and relatives' names, address, Social
  Security number, an identifying number or code, telephone number, email address, biometric
  record.
- **Indirect identifiers:** date of birth, mother's maiden name, geographic descriptors, place of
  birth, gender, race, religion, weight, activities.

For children under 13, COPPA adds persistent identifiers such as IP addresses and device IDs.
Counsel should confirm how a given product's data maps to these lists.

### Quick vetting look-fors

TEC's "Student Data Privacy Agreement Checklist" (adapted from The Educator's Guide to Student
Data Privacy, 2017) lists what to look for before a DPA exists. A product clears the basics when:

- Its privacy policy and terms are easy to find and read, say what is collected, why, and how it
  is shared, and are limited to the educational service.
- The vendor will not change the policy without notice, and a sale or merger carries the same
  protections.
- It describes how data is secured in transit and at rest and who can access it; encryption is a
  good sign.
- It will notify the district of a breach and help notify affected students.
- It shows no advertising to students. The "AdChoices" triangle marks behaviorally targeted ads,
  which are never acceptable for school use.
- It lets parents, or the school on their behalf, access the student's data.
- It shares student data only as needed to provide the service, never sells it or uses it for
  research without consent, and builds no student profiles beyond the educational purpose.

For a full review, Boise uses the 1EdTech TrustEd Apps rubric.

### Other useful pages

- **Public approved-app listing** (no sign-in):
  `https://sdpc.a4l.org/district_listing.php?districtID=15032`. Suitable to link from district
  web pages. The CLI reads it with `districts listing --district-id=15032`.
- **Signed PDFs** follow predictable public paths:
  `/agreements/{YYYY-MM-DD}_{districtID}_{softwareID}_signed_agreement_file.pdf` and
  `/addendums/{districtID}_{softwareID}_{YYYY-MM-DD}.pdf` for Exhibit Es.
- **State templates hub**: the standard NDPA and state agreement PDFs. The CLI lists them with
  `templates hub`.

### TEC training resources

**Public**, at https://tec-coop.org/data-privacy/training/:

- Videos: TEC SDPC Overview; How to Add A New Request; How to Use View Expiring Agreements to
  Manage DPA Renewals; How to Manually Change a Resource's Status.
- Quick reference guides: Understanding User Roles; TEC SDPC Workflow Overview; Setting a
  Resource as Approved w/Guardian Release; Setting a Resource as Approved (No Data Collected);
  What is PII?; Student Data Privacy Agreement Checklist.

TEC also shares with member districts, by email: a registry cheat sheet, Exhibit E and renewal demo videos, a list of products with special instructions, a current Massachusetts Exhibit E list, a sample parent cover letter and release (with Cambridge Public Schools' permission), and counsel's reviews of a few large vendors. Ask TEC for the current set.

## Appendix B: The CLI in this workflow

`sdpc-pp-cli` is a Go CLI for SDPC Registry 1.0, generated with the Printing Press on 2026-09-15/16
from the registry's pages and extended by hand with the registry write commands on 2026-09-17. It is
not yet in the public Printing Press library; the cookbook recipe describes how it was built. Its
own `SKILL.md` (shared on this shelf as `pp-sdpc`) and `README.md` carry the full command reference. This file covers what the DPA workflow needs and what
those generated docs get wrong.

### Contents

- What it can and cannot do today
- Signing in
- The write safety contract
- Commands by workflow step
- Output and exit codes
- Gotchas

### What it can and cannot do today (2026-09-17)

**Reads.** No sign-in: `districts listing` (Boise's public approved-app page), `files`,
`templates`, `resources resolve`. Needs a browser session: `lookup`, `alliance`, `national`,
`reference`, `pipeline`, and the `my-district` pages (Manage Agreements, expiring, expired,
prepared Exhibit Es).

**The TEC workflow.** All four steps now run from the CLI. Every one needs a browser session.

| Step | Command | Write? |
|---|---|---|
| Path B: find what Boise can piggyback on | `exhibit-e list` | no |
| Path B: create the Exhibit E | `exhibit-e create` | **yes** |
| Path B: TEC's after-check | `exhibit-e verify` | no |
| Path C steps 1 to 4: file the New Request | `request submit` | **yes** |
| Path C step 5: assign TEC | `request assign` | **yes** |
| Paths A and E: set Status, restrictions, declined reason | `request status` | **yes** |
| Path D: ask SDPC to add a missing product | `request new-resource` | **yes** |

**Still browser-only.** **Change Agreement Type** (Path E guardian release) is a multi-step
wizard whose first page is a search form. The later steps were never captured, so no
`request agreement-type` command was built. Do that one in the browser using
`registry-navigation.md`. The **logo upload** on Request to Add New Resource is a file field the
CLI leaves empty; add a logo in the browser if SDPC asks for one.

**Not usable yet:** `coverage`, `renewals`, `check`, `changes`. They read a local mirror filled by
the Premium API sync, Premium credentials were never configured, and the mirror is empty.

The working copy's own `SKILL.md` (the `pp-sdpc` skill) was corrected on 2026-10-11: it lists
these write commands and says each needs the person's explicit OK after a preview.

### Signing in

The CLI borrows a browser session. **You launch the login window; the person only signs in.**
Never paste `press-auth` commands for the person to type, and never enter credentials yourself.

Run this yourself, in the background (`run_in_background: true`, timeout 10 minutes):

```bash
~/go/bin/press-auth login sdpc.a4l.org --login-url https://sdpc.a4l.org/login.php --force
```

`--force` overwrites the stale state without a prompt, so no `forget` step is needed. press-auth
opens its own Chrome window on the SDPC login page, waits for the person to sign in, captures the
cookies, closes the window, and exits. Tell the person the window is up. Adding
`--complete-selector 'a[href*="logout"]'` makes it detect completion from the signed-in page
rather than waiting for the window to close.

When the background job completes, **import the new cookie into the CLI**. The CLI caches its
session in `~/.local/share/sdpc-pp-cli/credentials.toml` and keeps using the stale one until you
do this. `press-auth` must be on PATH for the import to find it:

```bash
PATH="$HOME/go/bin:$PATH" scripts/sdpc auth login --chrome
```

Success prints "Loaded cookies via press-auth for sdpc.a4l.org." Then continue the task. If you
skip this step, every gated command keeps failing with "requires a logged-in session" even though
`press-auth list` shows a fresh capture (confirmed 2026-09-26).

**press-auth captures the session from that window only.** Signing in to sdpc.a4l.org in your
normal Chrome does nothing for the CLI, so the CLI keeps failing with the same sign-in error while
the browser looks fine. `press-auth list` shows each domain and when it was captured, which is how
you spot a stale one. Sessions lapse after roughly an hour idle at most; on 2026-09-26 one expired
about ten minutes after sign-in, so batch gated reads right after the import.

`scripts/sdpc auth login --chrome` is an alternative import path, but it needs a cookie-extraction
tool on PATH (pycookiecheat, cookies, or cookie-scoop-cli); `doctor` reported none installed on
2026-09-17, so the press-auth route is the one that has worked.

**Premium API** commands (`agreements`, `sync --resources agreements`) use a separate login:
`SDPC_USERNAME` and `SDPC_PASSWORD` in the environment, then `scripts/sdpc session login`. This
has never been tested live. The CLI keeps **one cookie jar**, so a Premium login replaces the
browser session and the reverse. Re-run whichever one the next command needs.

Check for a live session with any cheap gated read, for example
`scripts/sdpc lookup resource --query IXL --agent --no-learn`. Do not trust `doctor` for this;
see Gotchas.

### The write safety contract

Every write command behaves the same way, so learn it once. **Preview by default:** `--dry-run`
prints the exact form that would be posted and sends nothing, and without `--yes` the command asks
before it submits. **`--yes` is required to submit.** **Under `--agent` or `--no-input`, a write
without `--yes` is refused** with exit 2 rather than submitted silently.

That covers the rule in SKILL.md: run `--dry-run`, show the person what it will do, wait for a
clear yes, then re-run with `--yes`. One write per confirmation. `--dry-run` still needs a live
session, because it reads the real form before showing it.

### Commands by workflow step

Add `--agent` for JSON and non-interactive behavior. Add `--no-learn` in scripted runs so the
CLI's learning store does not change later answers. `--district-id` and `--state` default to the
signed-in account; passing `--district-id=15032 --state=ID` avoids surprises.

**Triage.** Public: `districts listing --district-id=15032`, `resources resolve --software-id N`.
Session: `lookup resource --query "Blooket"`, `lookup company --query "Blooket"`,
`alliance active-dpas --state=ID --search "Blooket"`, `alliance resources --search "Blooket"`,
`reference successors --state=ID`. `national by-resource` walks every district in the country and
is expensive for popular products (IXL returns over 1,300 rows); prefer the state and lookup
commands.

**Boise's own rows (session).** `my-district agreements`, `my-district expiring`,
`my-district expired`, `my-district prepared-exhibit-es`, each with
`--district-id=15032 --state=ID --agent`. Also `pipeline --state ID --older-than 30d --agent`.

`pipeline` is a **state** view, not a district view. It returns every Idaho alliance row, West Ada's
included, and the `--district-id` flag does not narrow it. On 2026-10-07 it returned 147 rows of
which 5 were Joint School District No. 2. Filter its rows against the data IDs from
`my-district agreements` before counting anything, or the totals will be wrong. `pipeline` also
covers only in-progress rows: Active and Declined agreements are in `my-district agreements` and
the public listing, not here.

**Path B: Auto Exhibit E Creator**
```bash
scripts/sdpc exhibit-e list --district-id=15032 --state=ID --agent            # everything available
scripts/sdpc exhibit-e list --query "Fanschool" --district-id=15032 --state=ID --agent
scripts/sdpc exhibit-e create --data-id 123456 --district-id=15032 --state=ID --dry-run
scripts/sdpc exhibit-e create --data-id 123456 --district-id=15032 --state=ID --yes
scripts/sdpc exhibit-e verify --resource "Pixlr for Education" --district-id=15032 --state=ID --agent
```
`list` reads the Auto Exhibit E Creator grid. Each row reports `available` or `already_signed`
plus `data_id`, `software_id`, the originating district, the dates, and a link to the originating
signed agreement PDF. Read that PDF before creating: a piggyback inherits its terms, riders
included. Search is exact match only; `--query` adds the double quotes when you leave them off and
reports what it sent as `query_used`. `--all` pages through everything; `--limit` defaults to 25.

`create` replays that row's own Create Exhibit E form; `--software-id` is optional and confirms
the row, and an already-signed row is refused with exit 2. `verify` is TEC's after-check on View
Prepared Exhibit Es: district information and an electronic signature both present. A row present
with no signature fails; a resource not on the page yet is exit 0 with a note.

**Path C: New Request, then assign TEC**
```bash
scripts/sdpc request submit --resource "Pixlr for Education" --district-id=15032 --state=ID --dry-run
scripts/sdpc request submit --software-id 200 --district-id=15032 --state=ID \
  --status "New Request" --requestor "Your Name" \
  --requestor-email you@district.org --grade K,1,2 \
  --content-area "Educational Technology" --purpose "Classroom image editing" --yes
scripts/sdpc request assign --data-id 123456 --dry-run
scripts/sdpc request assign --data-id 123456 --yes --agent
```
`submit` fills Add New Request: `--software-id` or exact `--resource`, `--status` (New Request,
Renewal Pending (Public), Renewal Pending (Not Public); default New Request), `--requestor` and
`--requestor-email` for TEC's Internal Data contact, `--grade` (JK,PK,K,1 to 12),
`--content-area` (20 values, listed in `request submit --help`), `--purpose`, `--notes`, and
optional vendor and cost fields. It re-reads Manage Agreements afterwards and reports the new
row's `data_id`. `--assign-tec` chains the Progress Admin step; **it is off by default and for
Path C only.** Paths A and E file the request without assigning TEC.

`assign` sets Progress Admin (`--admin`, default TEC) and optionally the Progress stage
(`--progress "Assigned to TEC"`), submits that row's Update form, then re-reads the page and
reports ok only when the row shows the new admin.

**Use `--data-id`.** With `--software-id` or `--resource`, `assign` takes the *first* row on the
page that matches, with no check of its status. If Boise has an older or expired row for the same
product, that row can be the one changed. The preview shows which data ID it picked; check it.

**Path A and conditions of use**
```bash
scripts/sdpc request status --data-id 123456 --approved-no-data \
  --restrictions "offline drawing only; iCloud and sharing disabled" \
  --district-id=15032 --state=ID --dry-run
```
`status` is Edit Other Info. `--approved-no-data` is shorthand for
`--status "Approved (No Data Collected)"`. `--status` takes the label as the page shows it, read
from the fetched page rather than hardcoded, so a wrong label fails with the labels the site
actually offers. `--restrictions` writes Agreement Restrictions, `--notes` the notes textarea, and
`--declined-reason` is required for the Declined statuses. Edit Other Info saves the whole record,
so the command replays every control the page rendered and rewrites only the fields you named. The
Exhibit A and B file uploads are never touched.

**Path D: Request to Add New Resource**
```bash
scripts/sdpc request new-resource --company "Inmagine Lab PTE. LTD." \
  --name "Pixlr for Education" --website https://pixlr.com/education/ \
  --terms https://pixlr.com/terms/ --privacy https://pixlr.com/privacy-policy/ \
  --district-id=15032 --state=ID --dry-run
```
`--company`, `--name`, and `--website` are required; `--terms` and `--privacy` are the URLs SDPC
now expects. The logo is a file upload the CLI leaves empty. Confirm the product is genuinely
missing first with `lookup resource` under other spellings and `reference successors`.

**Documents (public):** `files agreement`, `files addendum`, `templates hub`.

### Output and exit codes

Reads return `{"meta": {"source": "live" | "local"}, "results": ...}`. Read `.results`.

| Exit | Meaning |
|---|---|
| 0 | Success |
| 2 | Usage error, or a write refused for lack of `--yes` |
| 3 | Not found |
| 4 | Authentication required |
| 5 | API error |
| 7 | Rate limited |
| 10 | Config error |

### Gotchas

- **Progress stage and Progress Admin are not in CLI output.** `my-district agreements` returns
  status but not the Progress Workflow stage (for example "Contract Sent"). Since 2026-10-07 the
  registry shows the stage and the per-row Update form only on the **Manage New Requests and
  Renewal Agreements** page (`manage_agreements_new_requests.php`), not on Manage Agreements.
  Fetch that page (and a row's log at `view_progress_log.php?state=ID&dataID=<id>`) with the
  CLI's signed-in session.

  In the page each row is a `<form>` with hidden `dataID`, `before` (admin id) and `before2`
  (current stage label); the `progress_admin` select's selected option is the admin. Note the
  `selected="selected"` attribute has no space before it. Active and Declined rows are not on that
  page; take their status from `my-district agreements`. A 302 with an empty body means the
  session expired. `request assign` was updated the same day to read the form from the new page
  (falling back to the old one).
- **Stage changes do not write progress log entries.** An empty log does not mean nothing
  happened; check the stage.
- **New Request rows carry no dates** in Manage Agreements, and a Declined row on the public
  listing folds its reasoning into the status text ("Declined Declined Reasoning: ...").
- **The CLI stamps dates in UTC.** A request filed on a Boise evening shows the next day's date.
- **Expired sessions print plain text even under `--agent`**, which breaks JSON parsing in loops.
  Check for "requires a logged-in session" before parsing.
- **zsh reserves `status`.** Do not use it as a loop variable in batch loops.
- **`districts listing`** takes only `--district-id`; `--state` is an error.
- **`lookup resource --agent` dropped names** before the 2026-10-11 build (`--compact` stripped
  `text`). The current build keeps `text`; `triage.py` uses `--json --no-input` either way.

- **Expired sessions report inconsistent exit codes.** Lookups get the HTML "SDPC: Error" page
  and exit 5 ("expected JSON, API returned HTML instead of JSON: ... SDPC: Error").
  `my-district` reads and the `request` and `exhibit-e` commands, including `--dry-run`, exit 4
  with a sign-in message. Look for either. `triage.py` does.
- **Signing in to your normal Chrome does not refresh the CLI's session.** See Signing in above.
- **`doctor` checks reachability only.** Before the 2026-10-11 build it reported API FAIL because
  its probe expected JSON from an HTML page; it now reports "reachable". It does not test the
  session and reports the store as empty, which is accurate. Use a real gated read for the session.
- **Many subcommands need flags `--help` does not flag as required.** `my-district agreements`
  with no flags fails with "requires input." Always pass `--district-id=15032 --state=ID`.
- **`lookup resource` takes `--query`, not a positional name.**
- **`request --help` calls the filing command `request new`.** It is `request submit`.
- **The learning loop** (`recall`, `teach`) is on by default. Strip student and staff names from
  anything taught. Use `--no-learn` for deterministic runs.
- **A spec change plus `generate --force` drops the hand edits.** Keep a reapply script with the manuscripts.
- **Registry 2.0** (registry.a4l.org) is out of scope for this CLI.

## Appendix C: Vendor outreach before a DPA

Some products need answers from the vendor before Boise can decide whether to sign anything. This
file is the reusable question set and the lessons from running it. The worked example is
Fanschool, which Boise reviewed in September 2026; the questions are not Fanschool-specific.

### Contents

- When to send a question set
- The question set
- Sequencing: when the vendor asks for a signed DPA first
- Lessons that carry
- The Fanschool example in brief

### When to send a question set

Send one when the registry cannot answer the question on its own. Typical triggers:

- The product carries **student communication**: posts, comments, replies, messaging, or anything
  an audience other than the teacher can see. Idaho's digital communication rule applies to these, which raises their review priority.
- The product is **P1 or P2** on Boise's sensitivity scale.
- The vendor's **privacy policy and the registry disagree** about who the company is.
- The product asks for a **broad OAuth scope** into Google Workspace.
- The terms of service look **stale or consumer-grade** next to what a district needs.

Send it before the request goes to TEC when you can, so TEC negotiates with the answers in hand.
If the request is urgent, file it and send the questions in parallel, and say in the request notes
that answers are outstanding.

### The question set

Written to a vendor, these are four groups. Adapt the wording; keep the substance.

### 1. Administrative visibility and oversight

- Can an organization administrator see **all content in all spaces**, including posts restricted
  to a limited audience and responses that are pending, hidden, or unapproved? Or is visibility
  limited to the moderator of each space?
- Is there an **audit log** of user activity, moderation actions, edits, deletions, and sign-ins?
  Who can read it and how far back does it go?
- Separately from any product feature: on a **written district request** for all content in a
  named space, by a named user, or within a date range, can the vendor produce it? In what format,
  and within what committed response time?
- Can the district **export in bulk** itself, without the vendor running the export?
- When a user deletes something and the interface calls it permanent, is the record **gone from
  the database** or retained in backups? For how long, and can the vendor recover it?
- Can **deletion be disabled** at the organization level, or a **preservation hold** applied?
- Can **private or unapproved reply traffic** be viewed by any account, including an
  administrator?

### 2. Identity and access

- Can sign-in be **restricted to a single domain** so that only district accounts can join?
- Why does the product request the OAuth scopes it requests? Name the scope and say what it
  grants. Fanschool requests `classroom.coursework.students`, a read and write scope over student
  coursework.
- **Will the product work with a read-only scope** instead?

### 3. Contracting

- Will the vendor execute the **SDPC National Data Privacy Agreement Standard Clauses v1.0 with
  Idaho Exhibit G**?
- **Which legal entity is correct** when the privacy policy and the SDPC registry name different
  companies? Get the answer in writing.
- Will the vendor accept an **Exhibit H** carrying the records-production response-time commitment
  from group 1?
- If the terms of service are stale, or contain a **perpetual sublicensable license over user
  submissions** or a **low liability cap**, ask the vendor to confirm in writing that an executed
  DPA controls for district data and student work.

### 4. Pricing

- Ask what a **school or district tier actually adds**. Get the feature list, not the marketing
  page.
- Do **not** assume a paid tier grants administrative visibility into content. Ask directly; it is
  a group 1 question, not a pricing question.

### Sequencing: when the vendor asks for a signed DPA first

A vendor may ask Boise to send a partially executed DPA before it answers the substantive
questions. Fanschool did: its reply asked for a partially executed DPA with exhibits, by DocuSign
or as a filled PDF, plus school and grade-level counts for a quote, and promised answers later.

That forces a sequencing choice:

| Sign first, ask after | Ask first, sign after |
|---|---|
| Keeps the vendor engaged and moves the quote along | Keeps Boise's commitment tied to answers it has actually seen |
| Boise's signature is on a document before the oversight and records answers are known | May stall a vendor that treats the DPA as its intake step |

Either way the agreement itself goes to counsel, and the decision belongs to the district, not to
this skill. Say what each option costs and let the named decision-maker choose. If Boise does send
a partially executed DPA, note in the request that the substantive answers are still outstanding
so nobody later reads the signature as a completed review.

### Lessons that carry

**Public records obligations drive several of the questions.** The operative ones are whether
deletion is truly permanent, whether a preservation hold is possible, and whether the district can
export in bulk on its own. If a vendor cannot preserve or produce, say so plainly in the
recommendation rather than burying it. Counsel confirms what the obligation requires.

**Entity mismatches are common and must be resolved before signing.** Fanschool's privacy policy
names Kidblog Inc., a Delaware corporation, while the SDPC registry and the Massachusetts Exhibit
E list name Fanschool Inc. The registry also carries `Fanschool` (softwareID 15551) and `Kidblog`
(softwareID 123) as two separate resources. Confirm the signing entity in writing before anything
is executed, and check **View Company Successor List** for renames and acquisitions.

**An Exhibit E in another state is a lead, not coverage.** Fanschool Inc. appears on the
Massachusetts Exhibit E list, and
`scripts/sdpc exhibit-e list --query "Fanschool" --district-id=15032 --state=ID` still returned
zero available rows for Boise, because an Exhibit E applies within its own state. The Auto Exhibit
E Creator is the only test that matters. When it comes back empty, the product goes to Add New
Request with TEC assigned (Path C).

**A promised answer is not an answer.** Record which questions are still open, and keep them open
in the request notes until the vendor answers them in writing.

### The Fanschool example in brief

| Item | What Boise found |
|---|---|
| Registry names | `Fanschool` softwareID 15551 and `Kidblog` softwareID 123, separate resources |
| Privacy policy entity | Kidblog Inc., a Delaware corporation |
| Registry and MA Exhibit E entity | Fanschool Inc. |
| Massachusetts Exhibit E list | Fanschool Inc. is on it |
| Auto Exhibit E Creator for Boise | Zero available rows |
| OAuth scope requested | `classroom.coursework.students` (read and write) |
| Vendor's first reply | Asked for a partially executed DPA with exhibits (DocuSign or filled PDF) plus school and grade-level counts for a quote; substantive answers promised later |
| Path | C, New Request with TEC assigned |

Nothing above is a legal conclusion. The entity question, the records
obligations, and the terms of service all go to counsel.