---
name: tams-dpa-record
description: Write or update a 1EdTech TrustEd Apps (TAMS) dashboard record for an app whose student data privacy agreement is Active in the SDPC/A4L registry, in a standard approved-record format (agreement link, dates, Exhibit B data, key terms, tags, survey answers). Also covers marking apps "UNDER REVIEW" when a DPA request is filed. Use when asked to update TAMS for new A4L/SDPC approvals, mark an app approved or in progress in 1EdTech/TAMS, or sync TAMS with SDPC.
---

# TAMS records for SDPC agreements

*Shared by Ryan Gravette, Boise School District (Idaho), on the Agentic Builders skills shelf.
It is Boise's working playbook, lightly edited for sharing: district-specific values are called
out in "Adapting this" below. The helper scripts it names are in the appendix at the end.*

The district keeps its 1EdTech TrustEd Apps (TAMS) dashboard in sync with the Student Data
Privacy Consortium (SDPC) registry. Two record shapes exist: **approved** (an Active signed
DPA in the registry) and **in progress** (a New Request has been filed).

## Adapting this for your district

- Replace the SDPC district ID (Boise is 15032, state ID) with yours. The registry link in
  the rendered notes and the `--district-id` flag both use it.
- "TEC" is the consortium that works Boise's registry requests and holds the Idaho add-on.
  Substitute whoever files and signs for you, and your state's NDPA add-on and exhibit names.
- The tag names (`DPA on file`, `SDPC Active`, `ID-NDPA`, `SDPC New Request`, `DPA requested`,
  `TEC assigned`) and the two survey questions are configured on the TAMS dashboard, not in
  the CLI. Create them there first, or rename them in `render_record.py`.
- `tams-pp-cli` and `sdpc-pp-cli` are Go CLIs generated with the Printing Press
  (see the matching cookbook recipe). Both sign in through a browser window the CLI opens;
  no token is ever pasted into a chat.

## Rating rule (operator direction 2026-10-09)

`APPROVED` is reserved for apps with a DPA on file (the `DPA on file` tag: SDPC Active,
Active-with-restrictions, or Not in Use: Part of Agreement). Every other app that would
otherwise be "approved" is rated `REVIEWED`. `UNDER REVIEW` stays for open DPA requests and
`DENIED` is untouched.

`PREFERRED` is a Teaching and Learning decision, not an IT one: it is set when T&L wants the
app used over alternatives, and an app T&L licenses defaults to it. Never set PREFERRED from
the agreement alone, never remove it for lack of a DPA, and never downgrade it. When a
PREFERRED app has no `DPA on file` tag, list it in the report as a fact (it is public on the
dashboard without an agreement); do not change it.

After any approval batch, sweep the dashboard: any `APPROVED` app without the tag goes to
`REVIEWED` (PREFERRED apps are excluded from the sweep). This is a rating-only plan,
`{"product","applicationId","rating":"REVIEWED"}` per app, applied with
`my-apps apply --update --yes`.

## Approved (SDPC status Active)

| Field | Value |
|---|---|
| Rating | `APPROVED` (keep an existing `PREFERRED`; never downgrade) |
| Tags | `DPA on file`, `ID-NDPA` (state agreement or state add-on), `SDPC Active`; remove `SDPC New Request`, `DPA requested`, `TEC assigned` |
| Survey answers | `Grades` = grades covered (KG, 01 … 12); `Is there a data sharing agreement on file between the institution and vendor?` = `Yes` |
| Technical notes | rendered by `render_record.py` from the agreement (format below) |
| Instructional notes | Exhibit A service description, quoted with page, and whether coverage extends to all provider services |

Technical-notes paragraphs, in this order (each a `<p>` with a bold label):
1. **Data privacy agreement (signed):** registry PDF link.
2. **Agreement:** type, e.g. "TEC ID-NDPA V1 with Exhibit E — Idaho Add-On with Idaho Exhibit G. It incorporates the <states> NDPA, Standard Version 1.0 (Standard Clauses v3.0)." or "Idaho NDPA … Signed directly with the district."
3. **District effective:** date · **National DPA:** originating LEA (state), signed date, term · **Expires:** date (+ "provisional: originating state's Exhibit G makes the term open-ended" when applicable).
4. **SDPC status:** status (approved date). Registry link. Include any registry restriction text verbatim.
5. **Student data in Exhibit B (N):** checked elements, in schedule order; quote any "Other" entries.
6. **Subprocessors:** as named in the DPA/Exhibit K, or "None named in the DPA."; vendor data security plan if attached.
7. **Key terms:** breach notice hours; deletion days after written request; data storage locations; "Idaho Exhibit G applies."
8. **Notes:** short facts that matter for use (scope limits, sensitive elements, survey/questionnaire elements that need pre-approval under the state exhibit, vendor location, AI processing, restrictions).

Facts only, taken from the signed PDF and the registry. No recommendations.

## In progress (DPA requested)

| Field | Value |
|---|---|
| Rating | `UNDER REVIEW` (do not lower `APPROVED`/`PREFERRED` an app already has) |
| Tags | `SDPC New Request`, `DPA requested`, `TEC assigned` (only once the consortium is Progress Admin) |
| Technical notes | `<p><strong>SDPC request:</strong> New Request filed YYYY-MM-DD (data ID N), Progress Admin TEC. Reason: …</p>` |

## Workflow

1. Get current SDPC rows (`sdpc-pp-cli my-district agreements --state ID --district-id <id> --agent`, saved as a dated JSON snapshot). Targets = Active rows whose TAMS record is missing, not APPROVED/PREFERRED, missing the `SDPC Active` tag, or has no agreement notes.
2. Download each signed PDF (`agreement_pdf_url`, public) and extract the fields into `fields/<data_id>.json` (schema in the `render_record.py` docstring). Read the PDF itself; a subagent per agreement works well.
3. Build the `tams-pp-cli my-apps apply` plan (rating, tags, technicalNotes, instructionalNotes, surveys) with `assemble_plan.py`. It reads `fields/*.json`, takes TAMS application IDs from `catalog_map.json` (product to applicationId; `null` = not in the 1EdTech catalog; add new products there), and adds the part-of-agreement products:
   ```bash
   python3 scripts/assemble_plan.py --work ~/Documents/tams \
     --sdpc ~/Documents/tams/daily/$(date +%F)/sdpc.json --out ~/Documents/tams/plans/$(date +%F)-plan.json
   ```
   With `--sdpc` (a `my-district agreements` snapshot) it derives the part-of-agreement list from the live `Not in Use: Part of Agreement` rows and picks each parent as the one `fields/*.json` agreement from the same company. When a company has more than one active agreement, it skips those products until you read the signed PDF and pass `--parent "Child=<parent product>"`. It prints each entry and every skipped product with the reason. `render_record.py fields/x.json > plan.json` still renders single agreements but does no catalog mapping or part-of-agreement clones.
4. Auth: `tams-pp-cli auth login --timeout 9m` (run in the background; it opens its own browser window, the person signs in, and the CLI saves the session itself). Never copy a token out of the browser by hand.
5. Preview `my-apps apply plan.json --update`, show it, then add `--yes`. New apps are added; existing apps are updated.
6. Verify with `my-apps list --limit 500 --data-source live` (rating, tags, notes present).
7. If `--update` fails, set the rating on the website dashboard dropdown and tags/notes/survey answers in Edit Additional Info.

## Gotchas

- **Working files.** Keep `fields/<data_id>.json`, `catalog_map.json`, `plans/` and `agreements/` in one work directory (Boise uses `~/Documents/tams/`). The CLI binary lives on PATH.
- **SDPC sign-in goes stale within minutes.** Run `sdpc-pp-cli auth login --chrome` and then the `my-district agreements` read straight away; a login alone just re-imports the stale session.
- **SDPC statuses.** Besides `Active`, the table carries `Active With the Following DPA Restrictions: …` (treat as Active, record the restriction verbatim) and `Not in Use: Part of Agreement` (a product covered by another row's signed DPA; same record as the parent, prepend a "Covered by the <parent> agreement (SDPC data ID N)" note, and tag `DPA on file`). Every product with a DPA on file gets the tag, so these go on the dashboard too.
- **Reading the PDFs.** `extract_pdf_text.py` (pypdf; run with `python3 -E` from a directory that is not the download folder) gives clean text, but **Exhibit B checkmarks lose their row positions in text**. Render pages 16-19 (or 11-13 on shorter forms) with `render_pdf.swift <pdf> <outdir> <tag> 16,17,18,19` and read the images, or verify an agent's list against them. One agent per PDF works; give it the SDPC row, the file paths and the JSON schema from `render_record.py`.
- **Fields the agents usually cannot fill:** the add-on revision date (rarely printed; write "revision date not stated on the add-on"), grades (often absent from Exhibit A; use KG-12 and note it), and a deletion day-count on some vendor-modified NDPAs (`render_record.py` accepts a string or empty value).
- **Approval date vs district signature.** The agreement PDF filename date is the registry approval date; the add-on signature page is the district effective date; they differ by a day when the district signed late in the evening.
- **Catalog gaps.** `check-list --names …` against the local mirror finds exact names; `tams-pp-cli sync --resources apps` refreshes the 8,400-app mirror in a few minutes. Products absent from the catalog go on as private custom apps: dashboard > **Add Custom App** (name, supplier search-or-add, URL, rating, description). The CLI has no command for it, so use the browser. Open the form from the dashboard button, not by URL (a direct load re-runs sign-in and wipes typed fields); pressing Enter in the supplier box selects the highlighted supplier AND submits, so fill every field first and leave rating Unrated. Then set rating/tags/notes with `my-apps apply --update` and add the new custom IDs to `catalog_map.json`. Read `my-apps list` with `--no-cache` right after adding or new apps are missing. An `apps request` for 1EdTech vetting is still optional.
- **Update replaces technicalNotes and tags wholesale.** Diff against the live record before applying.
- PREFERRED and APPROVED are public on the district's embedded dashboard.
- One catalog product can have duplicate catalog entries; match supplier and website.
- The SDPC default Manage Agreements page hides Progress Admin above ~150 rows; read it from `manage_agreements_new_requests.php`.

## Appendix: helper scripts

Save these under `scripts/` next to this file.

### scripts/render_record.py

```python
#!/usr/bin/env python3
"""Render TAMS approved-record plan entries from extracted agreement fields.

Fields JSON (one per agreement):
{
 "product": "Math Playground", "catalog_id": 1234,
 "sdpc_data_id": 960436, "pdf_url": "https://sdpc.a4l.org/agreements/...pdf",
 "agreement": "TEC ID-NDPA V1 with Exhibit E — Idaho Add-On (...) with Idaho Exhibit G. It incorporates ...",
 "boise_effective": "2026-10-08",
 "national_dpa": "Francis Howell R-III SD (MO), signed 2025-10-10, 3-year term",
 "expires": "2028-10-10", "expires_note": "provisional: ...",
 "sdpc_status": "Active (approved 10/08/2026)", "registry_restriction": "",
 "exhibit_b": ["IP addresses/cookies", "Student name", ...],
 "subprocessors": "None named in the DPA.", "security_plan": "",
 "breach_hours": 72, "deletion_days": 60, "storage": "US, Canada, EU or UK",
 "notes": ["..."], "exhibit_a": "Exhibit A (p10): '...'", "coverage_all_services": false,
 "grades": ["KG","01",...,"12"], "idaho": true
}
Usage: render_record.py fields1.json [fields2.json ...] > plan.json
"""
import html, json, sys

DISTRICT_ID = 15032  # your SDPC district ID
DISTRICT = "Boise"   # label used in the "effective" line

def esc(s): return html.escape(str(s or ''))

def render(f):
    p = []
    p.append(f"<p><strong>Data privacy agreement (signed):</strong> <a href=\"{esc(f['pdf_url'])}\">{esc(f['pdf_url'])}</a></p>")
    p.append(f"<p><strong>Agreement:</strong> {esc(f['agreement'])}</p>")
    exp = esc(f.get('expires') or 'not stated')
    if f.get('expires_note'): exp += f" ({esc(f['expires_note'])})"
    p.append(f"<p><strong>{DISTRICT} effective:</strong> {esc(f.get('boise_effective'))} · <strong>National DPA:</strong> {esc(f.get('national_dpa'))} · <strong>Expires:</strong> {exp}</p>")
    st = f"<p><strong>SDPC status:</strong> {esc(f.get('sdpc_status'))}. Registry: https://sdpc.a4l.org (district {DISTRICT_ID})."
    if f.get('registry_restriction'): st += f" Registry restriction: {esc(f['registry_restriction'])}"
    p.append(st + "</p>")
    eb = f.get('exhibit_b') or []
    p.append(f"<p><strong>Student data in Exhibit B ({len(eb)}):</strong> {esc(', '.join(eb))}</p>")
    sub = esc(f.get('subprocessors') or 'None named in the DPA.')
    if f.get('security_plan'): sub += f" Vendor data security plan: {esc(f['security_plan'])}."
    p.append(f"<p><strong>Subprocessors:</strong> {sub}</p>")
    dd = f.get('deletion_days', 60)
    deletion = (f"deletion within {esc(dd)} days of written request" if isinstance(dd, int) else
                f"deletion: {esc(dd)}" if dd else "deletion: no fixed day count stated in the DPA (see notes)")
    kt = (f"breach notice within {esc(f.get('breach_hours', 72))} hours; {deletion}; "
          f"student data stored only in {esc(f.get('storage', 'US, Canada, EU or UK'))}" + ("; Idaho Exhibit G applies." if f.get('idaho', True) else "."))
    p.append(f"<p><strong>Key terms:</strong> {kt}</p>")
    if f.get('notes'):
        p.append(f"<p><strong>Notes:</strong> {esc('; '.join(f['notes']))}</p>")
    instr = esc(f.get('exhibit_a') or '')
    if instr:
        instr += ' Covers all Provider services.' if f.get('coverage_all_services') else ' There is no clause extending coverage to all services.'
    tags = ['DPA on file', 'SDPC Active'] + (['ID-NDPA'] if f.get('idaho', True) else [])
    surveys = {'Is there a data sharing agreement on file between the institution and vendor?': {'answers': ['Yes']}}
    if f.get('grades'): surveys['Grades'] = {'answers': f['grades']}
    return {'product': f['product'], 'applicationId': int(f['catalog_id']), 'rating': f.get('rating', 'APPROVED'),
            'tags': tags, 'technicalNotes': ''.join(p), 'instructionalNotes': f'<p>{instr}</p>' if instr else '',
            'surveys': surveys}

if __name__ == '__main__':
    plan = [render(json.load(open(a))) for a in sys.argv[1:]]
    json.dump(plan, sys.stdout, indent=1)
```

### scripts/assemble_plan.py

```python
#!/usr/bin/env python3
"""Build a tams-pp-cli `my-apps apply` plan from fields/*.json plus the part-of-agreement products.

    assemble_plan.py [--work ~/Documents/tams] [--catalog-map ~/Documents/tams/catalog_map.json]
                     [--sdpc <my-district agreements JSON>] [--parent "Child=Parent" ...]
                     [--out plan.json]

Inputs:
  <work>/fields/*.json   one per signed agreement (schema in render_record.py)
  --catalog-map          {"<product>": <TAMS applicationId> | null}; null = not in the 1EdTech catalog
  --sdpc                 `sdpc-pp-cli my-district agreements ... --agent` output. When given, the
                         part-of-agreement list is derived from its "Not in Use: Part of Agreement"
                         rows: the parent is the one fields/*.json agreement from the same company.
  --parent               override or supply a parent, e.g. --parent "Keeble=Proloquo2go"

Read-only: it only writes the plan file. Preview with `my-apps apply <plan> --update`.
"""
import argparse, copy, glob, json, os, re, sys

sys.path.insert(0, os.path.dirname(os.path.abspath(__file__)))
from render_record import render  # noqa: E402

# Active rows that carry no agreement of their own in fields/ and ride on a sibling's DPA.
# Fill in for your district, e.g. {'Pear Deck': 'GoGuardian Beacon'}.
ACTIVE_RIDERS = {}


def base_name(n):
    return re.sub(r'\s*\(.*?\)\s*', ' ', n or '').strip()


def rows_of(x):
    if isinstance(x, dict):
        x = x.get('results', x.get('rows', []))
    return [r for r in (x or []) if isinstance(r, dict)]


def main():
    ap = argparse.ArgumentParser(description=__doc__, formatter_class=argparse.RawDescriptionHelpFormatter)
    ap.add_argument('--work', default=os.path.expanduser('~/Documents/tams'))
    ap.add_argument('--catalog-map', default=None, help='default <work>/catalog_map.json')
    ap.add_argument('--sdpc', required=True, help='my-district agreements JSON; derives the part-of-agreement list')
    ap.add_argument('--parent', action='append', default=[], help='"Child=Parent" override (repeatable)')
    ap.add_argument('--out', default='plan.json')
    a = ap.parse_args()

    cat_path = a.catalog_map or os.path.join(a.work, 'catalog_map.json')
    CAT = json.load(open(cat_path))
    fields = {}
    for f in sorted(glob.glob(os.path.join(a.work, 'fields', '*.json'))):
        d = json.load(open(f))
        fields[d['product']] = d
    overrides = dict(p.split('=', 1) for p in a.parent)

    plan, skipped = [], []

    def catalog_id(name):
        for k in (name, base_name(name)):
            if k in CAT:
                return CAT[k], True
        return None, False

    def add(d, product=None, extra_note=None):
        d = copy.deepcopy(d)
        if product:
            d['product'] = product
        if extra_note:
            d['notes'] = [extra_note] + list(d.get('notes') or [])
        cid, known = catalog_id(d['product'])
        if not cid:
            skipped.append((d['product'], 'not in the 1EdTech catalog' if known else f'no entry in {cat_path}'))
            return
        d['catalog_id'] = cid
        plan.append(render(d))

    for d in fields.values():
        add(d)

    # Part-of-agreement products: (product, parent, own data ID, status, reason skipped)
    parts = []
    rows = rows_of(json.load(open(a.sdpc)))
    status = lambda r: (r.get('status') or r.get('agreement_status') or '').strip()
    name = lambda r: r.get('resource_name') or r.get('software_name') or ''
    by_id = {str(r.get('data_id')): r for r in rows}
    parent_by_company = {}
    for p, d in fields.items():
        r = by_id.get(str(d.get('sdpc_data_id')))
        if r:
            parent_by_company.setdefault(r.get('company_name'), []).append(p)
    for r in rows:
        n = name(r)
        if status(r).lower().startswith('not in use: part of agreement'):
            cands = parent_by_company.get(r.get('company_name'), [])
            parent = overrides.get(n) or (cands[0] if len(cands) == 1 else None)
            why = None if parent else (f"parent ambiguous ({', '.join(cands)}); pass --parent" if cands
                                       else f"no fields/*.json agreement for {r.get('company_name')}")
            parts.append((n, parent, r.get('data_id'), status(r), why))
        elif n in ACTIVE_RIDERS and n not in fields:
            parts.append((n, overrides.get(n) or ACTIVE_RIDERS[n], r.get('data_id'), status(r), None))

    for n, parent, did, st, why in parts:
        if why:
            skipped.append((n, why))
            continue
        if parent not in fields:
            skipped.append((n, f'parent {parent} fields missing'))
            continue
        note = (f"Covered by the {parent} agreement (SDPC data ID {did}, registry status '{st}'); same signed DPA "
                "and Exhibit E; the Exhibit B list below is the parent product's schedule, see notes for "
                "product-specific schedules")
        add(fields[parent], product=n, extra_note=note)

    json.dump(plan, open(a.out, 'w'), indent=1)
    print('plan entries', len(plan), '->', a.out)
    for e in plan:
        g = e['surveys'].get('Grades', {}).get('answers') or ['-']
        print(f"  {e['applicationId']:>6} {e['product']:36} {e['rating']} tags={e['tags']} grades={g[0]}..{g[-1]} notes={len(e['technicalNotes'])}ch")
    print('skipped:')
    for s in skipped:
        print('  ', *s)


if __name__ == '__main__':
    main()
```

### scripts/extract_pdf_text.py

```python
import sys, os
from pypdf import PdfReader
for p in sys.argv[1:]:
    try:
        r = PdfReader(p); txt = []
        for i, pg in enumerate(r.pages, 1): txt.append(f"\n=== PAGE {i} ===\n" + (pg.extract_text() or ''))
        out = p[:-4] + '.txt'; open(out, 'w').write(''.join(txt)); print(os.path.basename(p), len(r.pages), 'pages', len(''.join(txt).split()), 'words')
    except Exception as e: print(os.path.basename(p), 'ERR', e)
```

### scripts/render_pdf.swift (macOS)

```swift
import Foundation
import PDFKit
import AppKit
let args = CommandLine.arguments
let pdfPath = args[1]; let outDir = args[2]; let tag = args[3]
let pages = args[4].split(separator: ",").compactMap { Int($0) }
guard let doc = PDFDocument(url: URL(fileURLWithPath: pdfPath)) else { print("cannot open"); exit(1) }
for n in pages {
    guard let page = doc.page(at: n-1) else { print("no page \(n)"); continue }
    let bounds = page.bounds(for: .mediaBox)
    let scale: CGFloat = 2.0
    let size = NSSize(width: bounds.width*scale, height: bounds.height*scale)
    let img = NSImage(size: size)
    img.lockFocus()
    NSColor.white.setFill(); NSRect(origin: .zero, size: size).fill()
    let ctx = NSGraphicsContext.current!.cgContext
    ctx.scaleBy(x: scale, y: scale)
    page.draw(with: .mediaBox, to: ctx)
    img.unlockFocus()
    let rep = NSBitmapImageRep(data: img.tiffRepresentation!)!
    let png = rep.representation(using: .png, properties: [:])!
    let out = "\(outDir)/\(tag)-p\(n).png"
    try! png.write(to: URL(fileURLWithPath: out))
    print("wrote \(out)")
}
```