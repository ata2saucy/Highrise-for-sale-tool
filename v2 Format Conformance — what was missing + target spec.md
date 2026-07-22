# v2 Workbook Format — Conformance Check & Target Spec

> *Adapted for the for-sale tool from the rent-comp tool's conformance audit (which compared
> `10 Lower Spadina Rental Comps _vACTIVE.xlsx` against the canonical `v2 hickory mock comp.xlsx`,
> uploaded 2026-06-17, the regenerated mock). The sale-comp workbook inherits that target format
> with sale semantics throughout: sold prices instead of rents, `RD Resale` / `RD Pre-Con` instead
> of `RD Condos` / `RD Apartments`, and the pre-con premium in place of the apartment premium.
> Below: what that audit found was missing, what was fixed, and the exact target format every
> future run must produce.*

> **⚠ Superseded note (2026-06-18): C2 is no longer LINEST.** After this audit, the parking
> adjustment (C2) was changed from a guarded/bare **LINEST regression** to a **hard-coded
> $-per-parking-spot figure from area parking-spot sale comps** (the with-vs-without-parking
> sold-price delta), entered as a value with a source note and shared by both Raw Data sheets —
> there is **no LINEST and no J:N helper block**. Wherever this doc still says "LINEST" /
> "guarded C2" below, the operative behavior is the hard-coded version. **`CLAUDE.md` and
> `build_workbook.py` win on any conflict.**

---

## A. What was missing / off (and now fixed)

| # | Area | Mock (target) | Audited workbook (before) | Fix applied |
|---|------|---------------|----------------------|-------------|
| 1 | **Sheets** | **6 sheets**, no `Subject & Conclusion` | 7 sheets — still had `Subject & Conclusion` | Deleted the sheet; its summary is folded into the Output tab |
| 2 | **Output → unit-mix grid** | **5 bed-type columns**: Studio · 1-Bed · 1+Den · 2-Bed · 3-Bed (to col **AE**) | 4 columns, no Studio (to col AA) | Rebuilt the right block to 5 groups (Studio added); primary-building-only rows per spec |
| 3 | **Output → recommendation** | Folded-in block `★ RECOMMENDATION` with recommended $/SF, conservative, raw, premium, by-suite table, **custom suite-size input**, **NOTES & CONTEXT** (confidence legend in words) | Partial "mirror" block; no custom suite-size input, no notes; label referenced the deleted sheet | Rebuilt the fold to match; added custom suite-size input + NOTES with the confidence legend in words |
| 4 | **Data_Summary → INPUTS table** | **Removed** (not present) | Still present (A42:D54) | Removed |
| 5 | **Data_Summary → PRE-CON PREMIUM BASIS** | Present (block, N/A when no pre-con comps) | **Missing** | Added |
| 6 | **Data_Summary → C2 parking** | **Guarded** LINEST (`IF(COUNT≥10 AND parking varies, LINEST, 0)`) | Bare LINEST | Replaced with the guarded formula — _later superseded (2026-06-18): C2 is now a hard-coded area-comp figure, no LINEST (see banner)_ |
| 7 | **Data_Summary → C3** | `0` + label "— OFF" when no pre-con comps | `0.1` | Set to 0 + relabelled |
| 8 | **Floor Plans** | **18-column provenance log**: Building·Address·City·Subject/Comp·Source Site·Source URL·Date Read·Suite/Plan·Beds·Baths·Interior SF·Exposure·Floor Band·Stack/Line·Balcony/Terrace·Notes·Used-For-Units·Verification Tier | 8 columns (name·addr·city·suite·beds·baths·sf·exposure) | Rebuilt to the 18-column schema (subject row + comp-building plans + note) |
| 9 | **RD Resale → blank columns** | Populates the descriptive columns (**Outdoor Space, Building Amenities, Maintenance Fee, Property Tax, Locker**) | Left blank | Re-pulled the descriptive fields from condos.ca for every comp and populated |
| 10 | **RD Resale → formats** | MLS Size Range with `" sqft"` (e.g. `600-699 sqft`); Building Age with `"(N yrs)"` (e.g. `Built 2023 (3 yrs)`) | `700-799`; `Built 2025` | Added " sqft" and "(N yrs)" |

**Already matching** (no change needed): the 41-column A:AO RD schema and per-row live formulas; Building Summary columns + `ALL RESALE` / `ALL PRE-CON` bands; the Data_Summary lever/rollup/by-bedroom/SUBJECT RECOMMENDATION/PREMIUM BASIS/MIX BASIS block order; H2:H4 derived unit-mix; the Output left comp block (group → buildings → Average → Pre-Con Premium → Implied Untrended → Subject Site → Other Excluded → Weighted Average); confidence fills on Raw Data; subject premium C4 as a live formula off PREMIUM BASIS (note the mock *types* C4 only because Hickory — the rent-comp tool's worked example — is a single-comp set with a 0% vintage spread; a multi-comp subject derives it).

Result: a conforming workbook has the **same 6 sheets**, the **5-bed-type Output grid**, the **18-column Floor Plans**, the populated RD columns, C2 (since superseded — now a hard-coded area-comp figure; see the banner above), the PRE-CON PREMIUM BASIS block, and **0 formula errors** (LibreOffice recalc, tied to an independent numpy recomputation).

---

## B. Target format — the spec every future run must hit

**Six sheets, in this order:** `Output` · `Building Summary` · `Data_Summary` · `RD Resale` · `RD Pre-Con` · `Floor Plans`. **No `Subject & Conclusion` sheet.** No INPUTS table. No /10 comp-quality scores. No colour fills anywhere except the Raw Data confidence columns.

**1. `RD Resale` / `RD Pre-Con`** — identical 41-column schema **A:AO**: Include · Sq Ft. · Price · Adj. Price · $/sq ft · Adj. $/sq ft · Adj BD · Beds Number · Den · Date · Building Name · Building Address · Building City · Developer · Unit # · Unit Address · Beds · Baths · Sqft (Condos.ca) · MLS Size Range · # Parking · Parking Included · Locker · Outdoor Space · Exposure · Building Age · Building Amenities · Sold Price ($) · List Price ($) · PSF (Calculated) · PSF (Listing) · Days on Market · Maintenance Fee ($/mo) · Property Tax ($/yr) · Sold Date · MLS# · Listing URL · Description · Product Type · $/SF (Product-Adj) · Date Scraped. **Populate every descriptive column** (Outdoor Space, Building Amenities, Maintenance Fee, Property Tax, Locker), not just the analytical ones. Formats: **MLS Size Range = `"<lo>-<hi> sqft"`**, **Building Age = `"Built YYYY (N yrs)"`**, SF & prices `#,##0`, $/SF keeps decimals. Live per-row formulas in A,C,D,E,F,G,H,I,AD,AE,AN. Confidence fills (green/cream/orange) only on cols B & E.

**2. `Data_Summary`** — levers + rollups, in this block order: C1 date filter (yellow) · **C2 = hard-coded parking adjustment** ($ per parking spot from area parking-spot sale comps, blue value + yellow lever, with a source note — **NO LINEST, no J:N helper block**) · **C3 = pre-con premium, 0 + "— OFF" when no pre-con comps** (yellow) · **C4 = subject premium, live formula off PREMIUM BASIS** (`=(C..+C..)/2`, black) · H2:H4 derived unit-mix · S/T subject mix + suite count · comp-building rollup (grows one row per comp) · By-Bedroom (all comps) · By-Bedroom (primary only, with Subj Price +prem) · SUBJECT RECOMMENDATION · **PREMIUM BASIS** · **MIX BASIS** · **PRE-CON PREMIUM BASIS**. **No INPUTS table.**

**3. `Building Summary`** — one row per comp building (Building · Address · Area · Developer · First Occupancy · Yr Built · Product · Storeys · Units · Transaction Count · Avg SF · Avg Price · Avg $/SF, live off the matching RD sheet) + **`ALL RESALE`** and **`ALL PRE-CON`** band rows.

**4. `Output`** — two regions:
- *Left:* comp blocks → group label → buildings → Average → Pre-Con Premium → Implied Untrended Price → Subject Site (Untrended Price) → Other Excluded (prose) → Weighted Average block; then the folded **`★ RECOMMENDATION`**: recommended $/SF (=`Data_Summary!C…25`-equiv) · conservative (all-comp+prem) · raw comp $/SF · plus premium · recommended price by suite type · **custom suite-size input** + implied prices · **NOTES & CONTEXT** with the **confidence legend in words**.
- *Right:* unit-mix grid with **5 bed-type column groups — Studio · 1-Bedroom · 1-Bedroom + Den · 2-Bedroom · 3-Bedroom** (each = Transaction Count · Avg Suite Size · Adj. Avg. Sold Price · Avg. Net Sold PSF), rows: **primary building → Weighted Avg. → Low → High → Pre-Con Premium → Weighted Avg.(×(1+PC)) → Weighted Avg. Building Total (Comps) → Subject Property (Comp $ Price) + Building Total → Subject Property (Comp PSF Price) + Building Total → Subject Property Price + Building Total → Regression/Coefficients/Total Building coefficient → Subject Property Price (Increase) → TRREB Data table** (district + City of Toronto + GTA YoY, 1BR/2BR/3BR).

**5. `Floor Plans`** — **18-column provenance log**: Building Name · Building Address · Building City · Subject/Comp · Source Site · Source URL · Date Read · Suite / Plan Name · Beds · Baths · Interior SF · Exposure · Floor Band · Stack / Line · Balcony/Terrace · Notes · Used For Unit(s) · Verification Tier. One row per distinct VIPcondos plan opened that session (subject plans tagged `(SUBJECT)`); a note row if only condos.ca registered areas were used.

**Conventions:** blue font = hard-coded input · black = formula/label · yellow fill = tunable lever (C1, C2, C3; **not C4**). Confidence fills (green `E2EFDA` / cream `FFF2CC` / orange `FCE4D6`) appear **only** on the Raw Data SF/$/SF columns. Gridlines hidden on all sheets. `fullCalcOnLoad` set. Verify: recalc to **zero formula errors** and tie averages / weighted blocks to an independent recomputation, and confirm C2 carries its area-comp source note, before delivering.
