# Tunda Box Junior Data Analyst Assessment

**Candidate:** Dennis Thanji  
**Project:** Tunda Box Junior Data Analyst Assessment  
**Status:** Preparation portfolio / worked assessment responses

## Repository contents

- [Assessment responses](assessment-responses.md) — consolidated written responses for Parts 1–5.
- [Solution notebook](tunda_box_assessment_solutions.ipynb) — reproducible Python calculations, SQL examples, data-quality checks and statistical analysis.
- [Thought process](THOUGHT-PROCESS.md) — assumptions, analytical approach, validation decisions and limitations.
- [Drive copy](#google-drive-copy) — link to the complete assessment document.

## Google Drive copy

**TODO:** Upload the complete assessment document to Google Drive, set access so the intended reviewers can open it, then replace the placeholder below with its share link.

[Open the complete assessment document in Google Drive](https://drive.google.com/file/d/1kHiROtrTVHS1h-ghcksZ21F2z5KPcErG/view?usp=sharing)

The Drive copy should include all assessment responses in `assessment-responses.md`, together with the thought-process explanation. Do not submit this placeholder link.

## How to review

1. Read the assessment responses for the written explanations and recommendations.
2. Open the notebook and run the analysis cells from top to bottom.
3. Read the thought-process document for assumptions, data-quality treatment and interpretation.

## Important notes

- Trading Bot V6 figures are historical **simulated research/backtest results**, not live trading income or proof of future profitability.
- Tunda Box is the assessment case dataset. Suspicious records are flagged for investigation rather than silently changed.
- The duplicate order 1006 is an exact duplicate in the supplied sample and is counted once in deduplicated analyses.
- Order 1013 (KES 185,000) is treated as an unverified outlier, not automatically deleted.
- Order 1009 has a missing amount; it is not assumed to be zero.
- Order 1020 references customer 999, who is absent from the customer table.
- SQL examples use PostgreSQL syntax. Spreadsheet examples use modern Excel syntax where indicated.

## Reproducibility

The notebook uses Python with pandas, numpy and scipy. Install dependencies with:

```bash
python -m pip install pandas numpy scipy jupyter
```

Launch it with:

```bash
jupyter notebook
```

---
Prepared as a junior data analyst assessment portfolio.
