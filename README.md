# ReconAI - Transaction Classification & Reconciliation (prototype)

Academic prototype. Uploads a bank statement and a cash book, classifies every bank transaction,
reconciles bank vs books, and produces a bank reconciliation statement and cash-flow summary.
All bundled data is **fictional**.

## Run it

```bash
cd reconai
python -m venv venv
venv\Scripts\activate        # Windows   (Mac/Linux: source venv/bin/activate)
pip install -r requirements.txt
streamlit run app.py
```

The app opens in your browser with `sample_data.xlsx` pre-loaded.

## Files

| File | Purpose |
|---|---|
| `app.py` | Streamlit user interface (6 tabs) |
| `recon_engine.py` | All logic: loading, classification, reconciliation, summaries, evaluation |
| `sample_data.xlsx` | Fictional FY2025 dataset (bank statement, cash book, answer key) |
| `requirements.txt` | Python dependencies |

## How it works

**Classification (hybrid, 3 layers)**
1. Keyword / regex rules for clear narrations (`PAYE`, `POS SETTLEMENT`, `STAMP DUTY` ...).
2. Machine learning for the rest: TF-IDF (word + character n-grams) with Logistic Regression, trained on labelled history.
3. Anything below the confidence threshold goes to a human review queue. Corrections can be fed back to retrain.

**Reconciliation (5 passes)**
1. Exact amount within a date window (closest date first), flagged as *Timing difference* if it crosses a month end.
2. Transposed digits (e.g. 1,245,000 vs 1,254,000) flagged as *Amount difference*.
3. Many bank lines to one book entry (grouped).
4. One bank line to many book entries (split).
5. Leftovers: bank-only items, deposits in transit, outstanding cheques, duplicate entries.

**Outputs**: bank reconciliation statement (difference must be zero), cash flow statement (operating / investing / financing),
monthly summary by category, Excel export.

## Suggested demo flow (5 minutes)

1. **Data** tab: show the raw bank statement and cash book side by side.
2. **Classification** tab: filter by method (Rule vs ML); point out confidence scores.
3. **Review queue**: fix a few unclear items (`MOB TRF-DR-8890121`, `TRF FRM 3PL/PAYMENT 0034`), click Apply.
4. **Reconciliation** tab: show exceptions, then the bank rec statement reaching a zero difference.
5. **Financial summary**: cash flow statement and the suspense warning for unresolved items.
6. **Accuracy** tab: results against the Answer Key; untick the rules layer to show ML-only performance.

## Measured results on the sample data (honest reporting)

ML trained on Jan-Jun labels, tested on Jul-Dec (290 transactions):

| Setup | Accuracy | Flagged for review |
|---|---|---|
| ML only (no rules) | 94.1% | 17 |
| Rules + ML (hybrid) | 99.7% | 32 |

By clarity, ML only: Clear 98.3%, Unclear 87.2%, **Ambiguous 20%**. Hybrid: Ambiguous 90%.
Reconciliation status matches the answer key on 573 of 573 bank lines.

## Limitations (state these in your write-up)

- The rules were written with this dataset in view, so the hybrid figure is optimistic. Test on a different company's statement.
- Data is synthetic and its narrations are consistent, so real bank data will be messier (truncated text, mixed formats).
- Ambiguous items (capital vs loan, drawings vs expense) cannot be resolved from narration alone. They need business context, which is why the review queue exists.
- Amount-difference detection only catches digit transpositions, not other keying errors.
- Single bank account, single currency (NGN), single period.

## Ideas for extension

- Upload any CSV with column mapping; PDF statement parsing.
- Fuzzy matching on descriptions as an extra reconciliation signal.
- Learn from corrections over time and persist them in a database.
- Accrual adjustments (prepaid rent/insurance, loan interest accrual) to move from cash flow to a full income statement and balance sheet.
