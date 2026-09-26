# notes.md: Ganga water-quality dataset and dashboard

## Plan (decided before data work)
- Question: where along the Ganga, and when, does the water fail India's bathing standard, and how much does the answer depend on which number is reported?
- Bathing criteria (MoEF&CC notification, 2000): fecal coliform ≤ 2,500 MPN/100 mL (max permissible), pH 6.5–8.5, DO ≥ 5 mg/L, BOD ≤ 3 mg/L.
- Sources to audit: CPCB NWMP yearly river data PDFs (station min/max per year, ~2,265 river locations nationally); CPCB monthly Ganga interstate-boundary reports (monthly, with GPS); UEPPCB water quality data page (incl. Ganga during Magh/Kumbh Mela 2018–19); CPCB Maha Kumbh 2025 reports.
- Key constraint: NWMP gives only yearly min/max → no seasonality from NWMP; seasons/festivals need monthly/event data.
- Dropped from original idea: "before vs after STPs came online" (min/max data can't separate STP effects from flow, rain, upstream changes) → STP dates as context only, no causal claims.
- Honesty rules: yearly max > limit = at least one sample failed, not "unsafe all year"; fecal coliform on log scale; no blanket "Ganga is unsafe" claims; every value traceable to file + page; dashboard labelled not official.
- Thread with Project 1: what does a single summary number hide? (single random draw vs AUC range; median vs worst day, Maha Kumbh 2025 controversy.)

## Step 1: Data audit
