# Tunda Box Junior Data Analyst Assessment

**Candidate:** Dennis Thanji  
**Project:** Tunda Box Junior Data Analyst Assessment  
**Status:** Assessment response portfolio

## Repository contents

- [Assessment responses](assessment-responses.md) — consolidated written responses for Parts 1–5.
- [Solution notebook](tunda_box_assessment_solutions.ipynb) — reproducible Python calculations, PostgreSQL query examples, data-quality checks, statistical analysis and validation assertions.
- [Thought process](THOUGHT-PROCESS.md) — assumptions, analytical approach, validation decisions and limitations.
- [Drive copy](#google-drive-copy) — link to the complete assessment document.

## Google Drive copy

The accompanying assessment document is linked below. Before submitting, test the link in a signed-out or incognito browser to confirm that the intended reviewers can open it.

[Open the complete assessment document in Google Drive](https://drive.google.com/file/d/1kHiROtrTVHS1h-ghcksZ21F2z5KPcErG/view?usp=sharing)

The repository contains the written responses and thought process as accessible Markdown files. Check that the Drive copy contains the same final responses and is not an outdated draft.

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

The notebook uses Python with pandas, NumPy and statsmodels. From the repository folder, install the pinned minimum dependencies with:

```bash
python -m pip install -r requirements.txt
```

Launch Jupyter with:

```bash
jupyter notebook
```

Open `tunda_box_assessment_solutions.ipynb` and choose **Kernel → Restart & Run All**. The final notebook cell runs assertions against the key expected results.

## Final submission checklist

- [ ] Confirm the Google Drive link opens for a reviewer who is not signed in to your account.
- [ ] Confirm the Drive document matches `assessment-responses.md` and `THOUGHT-PROCESS.md`.
- [ ] Run every notebook cell from top to bottom and confirm the final validation message appears.
- [ ] Submit this public repository URL together with the Drive link if the assessment instructions request both.

---
Prepared as a junior data analyst assessment portfolio.
