# Ganga Water Quality: an open, cleaned dataset and dashboard 🌊

> 🚧 **In progress.** Built in the open.

Where along the Ganga, and when, does the water fail India's bathing standard,
and how much does the answer depend on which number you report?

India's Central Pollution Control Board (CPCB) monitors the Ganga at more than 100 stations,
but publishes the results mainly as long PDF tables, with formats that change from year to year.
This project extracts those tables, cleans them into one consistent, documented dataset,
and builds a dashboard on top of it.

**Not official data.** All values come from published CPCB reports; every value is traceable
to its source file and page.

## The bathing standard

Primary water quality criteria for outdoor bathing (MoEF&CC notification, 2000):

| Parameter | Criterion |
|---|---|
| Fecal coliform | ≤ 2,500 MPN/100 mL (maximum permissible) |
| pH | 6.5 – 8.5 |
| Dissolved oxygen (DO) | ≥ 5 mg/L |
| Biochemical oxygen demand (BOD) | ≤ 3 mg/L |

## Data so far

- **Yearly river data (CPCB National Water Quality Monitoring Programme, NWMP), 2016–2022 and 2024.**
  One PDF per year; for each station, the **minimum and maximum** of each parameter over the year.
  - **2023 is missing:** CPCB's 2023 file is damaged (all pages render black, even in a browser).
  - **The "2025" file covers only the Yamuna** (2 pages), so it isn't used.
  - 2014 exists only as web pages in several parts, in a different format, so it's left out for now.
- **Monthly data at inter-state boundary stations** (CPCB): being added.

Current extraction: **938 table rows** from the 8 yearly files → **114 Ganga stations** kept
(105 main stem, 8 headstream on the Bhagirathi, Alaknanda and Mandakini, 1 canal)
→ **12,848 values**, of which 11,755 are numbers, 207 are "below detection limit" and 883 not reported.

## Method (so far)

1. **Download and audit** every source file; log size, pages and whether it's text or scanned (`sources.csv`).
2. **Find the Ganga table** in each yearly file, and **extract every row** with PyMuPDF.
   Column layouts differ between years (and sometimes within one table), so each table's layout
   is learned from its complete rows; missing values stay blank instead of shifting into the wrong column.
3. **Station registry:** one row per station code, **classified by hand** (main stem / headstream / canal / dropped),
   with a clean name and the reason for every decision. The word "Ganga" alone isn't enough:
   some canal and non-river sampling points mention it, and some Ganga stations don't.
4. **Value cleaning:** every value keeps its original text. "BDL" (below detection limit) is kept as a qualifier,
   not replaced by a made-up number. Suspicious values are **flagged, not changed**: only extraction mistakes
   are corrected (and logged); values that look wrong in the source are marked and left out of analysis.

## Honesty rules

- A yearly **maximum** above the limit means **at least one sample** failed, not that the water was unsafe all year.
- Fecal coliform is shown on a **log scale**.
- No blanket claims like "the Ganga is unsafe": always a station, a parameter and a time.
- No cause-and-effect claims (sewage treatment plants, festivals) that min/max data can't support.

## Roadmap

- [x] Step 1: Data audit and download (`01_download_and_extract.ipynb`)
- [x] Step 2: Table extraction, all 8 years (`01_download_and_extract.ipynb`)
- [x] Step 3a: Station registry, reviewed by hand (`02_station_registry.ipynb`)
- [x] Step 3b: Value cleaning with qualifiers and flags (`02_station_registry.ipynb`)
- [ ] Step 3c: Hand check of flagged values against the PDFs; extraction QA sample ← **in progress**
- [ ] Step 3d: Station coordinates and order along the river
- [ ] Step 4: Publish the cleaned dataset with a data dictionary
- [ ] Step 5: Findings (river profile, home stretch, change across years, festival case study)
- [ ] Step 6: Interactive dashboard on GitHub Pages
- [ ] Step 7: Short write-up

## Repository structure

| File | What it does |
|---|---|
| `01_download_and_extract.ipynb` | Downloads the CPCB PDFs, checks them, finds the Ganga tables and extracts every row |
| `02_station_registry.ipynb` | Builds and reviews the station registry, cleans and flags the values |
| `notes.md` | Running log of every decision, problem and fix |

## Data sources

- Central Pollution Control Board (CPCB), National Water Quality Monitoring Programme (NWMP):
  yearly water quality data of rivers, 2016–2024, https://cpcb.gov.in/
- CPCB, water quality of River Ganga at inter-state boundaries (monthly reports).
- Ministry of Environment, Forest and Climate Change (2000): primary water quality criteria for bathing waters.
