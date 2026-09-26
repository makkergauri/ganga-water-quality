# notes.md: Ganga water-quality dataset and dashboard

## Plan (decided before data work)
- Question: where along the Ganga, and when, does the water fail India's bathing standard, and how much does the answer depend on which number is reported?
- Bathing criteria (MoEF&CC notification, 2000): fecal coliform ≤ 2,500 MPN/100 mL (max permissible), pH 6.5–8.5, DO ≥ 5 mg/L, BOD ≤ 3 mg/L.
- Sources to audit: CPCB NWMP yearly river data PDFs (station min/max per year, ~2,265 river locations nationally); CPCB monthly Ganga interstate-boundary reports (monthly, with GPS); UEPPCB water quality data page (incl. Ganga during Magh/Kumbh Mela 2018–19); CPCB Maha Kumbh 2025 reports.
- Key constraint: NWMP gives only yearly min/max → no seasonality from NWMP; seasons/festivals need monthly/event data.
- Dropped from original idea: "before vs after STPs came online" (min/max data can't separate STP effects from flow, rain, upstream changes) → STP dates as context only, no causal claims.
- Honesty rules: yearly max > limit = at least one sample failed, not "unsafe all year"; fecal coliform on log scale; no blanket "Ganga is unsafe" claims; every value traceable to file + page; dashboard labelled not official.
- Thread with Project 1: what does a single summary number hide? (single random draw vs AUC range; median vs worst day, Maha Kumbh 2025 controversy.)
- Timeline: application deadline November 2026 → Project 2 scoped to ~2 weeks; Project 3 reduced to a mini version (dataset + seasonal baseline) or deferred; SOP started in parallel (30–45 min/day); no new analysis in November.

## Step 1: Data audit and download (notebook 01_download_and_extract)

### Finding the files
- Guessed URLs failed for 2019/2022/2023; only 2021 opened. File names differ by year; CPCB site may also be intermittently slow → retry before concluding a file is gone.
- Correct route: yearly index pages on CPCB's site (e.g. https://cpcb.nic.in/nwmp-data-2014/) list yearly data 2012–2023; river files found at cpcb.gov.in/wqm/<year>/….
- Yearly river files found: 2016–2025 as PDFs; 2014 only as HTML pages in several parts (e.g. RIVERWATER DATA 2014_5.htm, _7.htm) → left out for now (different format).
- Monthly Ganga data: https://cpcb.nic.in/wq-interstate2/ lists 6 files, March 2019 – Sept 2021; plus the Jul–Aug 2023 file (https://cpcb.nic.in/NGTMC/ganga_Interstate10.pdf). Period covers the 2020 lockdown and the April 2021 Haridwar Kumbh (flow confounds any comparison → no causal claims). Six 2019–2021 links still to collect.
- UEPPCB page: not checked yet.
- Prior work check (Kaggle/GitHub "NWMP river"): not done yet (a Kaggle NWMP lakes dataset 2017–2022 exists).

### Downloading
- Links collected by hand in the browser; one Colab cell downloads into Drive ganga_project/raw/{nwmp, interstate, ueppcb} with standard names and logs results in sources.csv.
- 2016–2022 downloaded by Colab. The first download cell then hung for > 28 min: pdfplumber hangs when opening the 2023 PDF (12 MB, about twice the size of other years). 2023, 2024, 2025 and the interstate Jul–Aug 2023 file were uploaded by hand from the browser; Colab only saw them after a Drive remount (drive.mount(..., force_remount=True)).
- Notebook cleaned to 4 cells: (1) setup with PyMuPDF; (2) file list + record of hand-uploaded files; (3) streamed download with a 10-minute cap per file, skipping files already in Drive (safe to re-run; PDF signature check); (4) local copies in /content/raw_local + PyMuPDF checks (pages, text vs scanned) → sources.csv, plus first-page titles. No pdfplumber.
- Lesson: after a restart or disconnect, use Runtime → "Run before" on the cell you want to run.

### Audit results
- Text PDFs: 2016 (70 p), 2017 (81), 2018 (103), 2019 (66), 2020 (56), 2021 (127), 2022 (161), 2024 (84); interstate Jul–Aug 2023 (9 p).
- "2025" file: 2 pages, Yamuna only ("Water quality data of river Yamuna – 2025") → not the yearly river data; dropped.
- 2023 file (12 MB, 81 p): PyMuPDF errors "syntax error: expected object number", "non-page object in page tree"; pages render all black even in the browser → broken at source → 2023 skipped, recorded as a known gap (could report to CPCB). A repair attempt hung inside PyMuPDF → cell deleted.
- Working set: 2016–2022 + 2024 (8 years).

## Step 2: Extraction (notebook 01_download_and_extract)

### Finding the Ganga tables (Cell 5)
- Each yearly file = one table per river ("Water Quality of River <name>"); 2020–2022 start with an index page; title wording differs between years.
- Ganga pages by table titles: 2016 p. 8–10; 2017 11–16 (actually 12–17); 2018 11–17; 2019 12–16; 2020 13–16; 2021 20–27; 2022 22–30; 2024 9–13. "Ganga at" mentions: 37, 58, 51, 64, 73, 80, 71, 72 → network grew over time.
- Boundary risk (2017 p. 11 starts with column headers) → extract ±1 page.

### Extraction tests (Cells 7–10)
- PyMuPDF find_tables detects 1 table per page. 2022: 63-column grid (mostly empty spacers); all 94 GANGA rows complete (21 values at every 3rd column).
- 2022 column map: 0 code, 3 location, 6 state, 9/12 temp min/max, 15/18 DO, 21/24 pH, 27/30 conductivity, 33/36 BOD, 39/42 nitrate, 45/48 fecal coliform, 51/54 total coliform, 57/60 fecal streptococci.
- All years: GANGA rows 2016 57, 2017 71, 2018 68, 2019 85, 2020 93, 2021 92, 2022 94, 2024 105. 2016/2017: fixed columns 0–18, missing values = blank cells. 2019, 2021, 2022, 2024: always 21 values. 2018: irregular layouts. 2018/2020 single-cell rows = wrapped station names (no hidden data).
- DO, pH, BOD, fecal coliform present in all 8 years. Fecal streptococci only from 2019.
- Nitrate definition differs: "Nitrate + Nitrite" in 2016 and 2018, "Nitrate" in other years → no cross-year nitrate comparison.
- "GANGA" keyword also catches canals (e.g. Upper Ganga Canal) → registry needs a type column.
- Flaw found: when "GANGA" sits on a wrapped name line, the data row lacks the keyword → keyword filtering silently drops stations.

### Final extraction (Cell 11, v3)
- No keyword filter: every row starting with a 3–5 digit station code on Ganga pages ±1.
- Complete rows read in order (nothing missing → nothing can shift); incomplete rows mapped by column position, trying every complete-row layout in the same table; fixed-column years read directly by column.
- Wrapped name pieces attached to the vertically nearest data row ([before]/[after]); only accepted in the location column and without header words.
- Each row keeps its full raw text (row_text) for Ganga detection and hand checks.
- History: v1 attached header words as name fragments and read incomplete rows by order (a blank shifts all later values); v2 learned one layout per table, which broke 2018 (several layouts per table: 32 flagged rows, Ganga rows 65 → 56). v3 fixed both.
- Results: 938 rows; 2018 Ganga rows 65; 9 flagged rows, none Ganga; 0 flagged Ganga rows in any year. 2020 Ganga rows 91 (vs 71 complete keyword rows before). 2018 duplicate codes 2745/2746 = Tawi river stations (J&K) on an edge page → dropped later.
- Saved interim/nwmp_extracted_raw.csv.
- Extraction QA (50 random values vs PDFs): to do after cleaning.

## Step 3: Station registry and cleaning (notebook 02_station_registry)

### Registry
- Candidates: rows mentioning GANGA (raw text or name fragments) OR on core Ganga-table pages. Names rebuilt from fragments.
- Draft: 136 candidates → suggested main stem 104, check 24, headstream 6, canal 2. Years present: 8 years 57 stations; 7: 21; 6: 14; 5: 3; 4: 3; 3: 3; 2: 3; 1: 32.
- State names truncated in some years ("UTTARAKHAN") → standardised.
- Decisions (by hand):
  - 2727 Upper Ganga Canal D/S Roorkee = canal; 1061 = river station D/S Haridwar (canal only a landmark) → main stem.
  - Headstream: Bhagirathi (Gangotri, B/C Devprayag), Alaknanda (Badrinath, Rudraprayag B/C and A/C Mandakini, B/C Devprayag), Mandakini (Kedarnath, B/C Rudraprayag). 1489 (after the confluence at Devprayag) → main stem = first main-stem station.
  - Dropped: other rivers from table edges (Himachal streams, Yamuna, Srinagar, Punjab); 4432 Dhauli Khad (Himachal); 20048/20049/20050 (Swarg Ashram-1, Lakkar Ghat oxidation ponds, Jagjitpur, 2018–2019) = probably wastewater sampling points (to verify in 2018 PDF).
  - 3000x codes (30005 Sultanpur, 30006 Bijnor, 30075 Tarighat Ghazipur, 30076 Manjhighat Chhapra) = CPCB inter-state boundary stations → GPS in monthly reports; 30076 state = Bihar.
  - 2024-only stations 5708–5716 (UP), 5776–5778 (Uttarakhand) = network expansion in 2024; kept.
  - 2555 "Ganga at Punpun, Patna": kept, location to verify.
- Name clean-up: removed encoding junk ("Gangaâ") and cut-off endings.
- Final: main stem 105, headstream 8, canal 1, drop 22 → 114 stations kept. Saved interim/station_registry_v1.csv.
- Code-change check (shared place word, non-overlapping years): none → gaps are missing reports, not renamed stations (check only catches renames sharing a place word).
- Suspicious-word check: 3 hits, all fine (1061 canal landmark; 1070 Varanasi U/S B/C drains = useful "before" point; 1335 Patikali = "KALI" inside a place name, false positive).

### Value cleaning
- 740 station-year rows (114 stations), 0 duplicates. DO: 0 non-numeric values; pH: 2.
- Non-numeric values: "-", "--", "_", "__" (not reported); "BDL" (below detection limit: nitrate 113, BOD 36, fs 30, fc 14, tc 14); "160000 0" (tc, 2×); "0.." (nitrate, 1×).
- Rules: not-reported markers → missing; BDL → no number + qualifier "below detection limit" (counts as meeting bathing limits for BOD/FC; no substituted value); unreadable → check PDF. Fecal streptococci before 2019 = not measured (rows omitted).
- Tidy long table: year, station_code, station_name, parameter, stat, unit, value_raw, value, qualifier, flag, source_file, page → interim/ganga_values_long.csv.
- Flags (nothing auto-fixed): outside plausible range; min > max; jump ≥3× (cond) or ≥5× (BOD) vs the station's median over years.
- Results: 12,848 values → 11,755 numbers, 883 not reported, 207 below detection limit, 3 unreadable. 29 values flagged.
- Lesson: my freshwater "plausible" conductivity range (≤ 5,000) was wrong for the tidal estuary. West Bengal stations (Diamond Harbour 1469, Patikali 1335, Uluberia 1052, Serampore 1472, Howrah 1471) reach 5,000–33,000 µmhos/cm = brackish water from tidal seawater intrusion (seawater ~50,000) → real values; mark stations as estuarine in the registry.
- Physically implausible (likely source typos): pH 2.2/3.3 Chunar (2019/2020), 3.5 Khalgaon (2021); water temp 44 °C Janta Ghat (2019), 41 °C Balughat (2024).
- Suspicious inland conductivity: 21,601 Madhya Ganga Barrage Bijnor (2021), 1,961 Har-ki-Pauri (2022), 2,200 Anoopshahar (2020), 789/1,960 Kachhla Ghat (2019/2020); low minima 41 Howrah (2018), 16 Aami (2019).
- Unreadable: "160000 0" TC max at Sultanpur and Bijnor (2021, same page, same value); "0.." nitrate Mokama (2021).
- Principle: only correct OUR extraction errors (logged); never "correct" CPCB values → implausible source values kept, marked "suspect source value", excluded from analysis with the reason.
