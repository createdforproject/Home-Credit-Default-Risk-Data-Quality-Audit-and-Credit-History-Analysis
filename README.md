# Home Credit Default Risk: Data Quality Audit and Credit-History Analysis

An exploratory analysis of the Kaggle **Home Credit Default Risk** dataset using SQL (DuckDB) and pandas. The project profiles eight related tables (about 58.5 million rows), documents the data problems found, and tests whether an applicant's credit history at other lenders is associated with default.

It is a data-analysis project, not a prediction project: **no model was built**, and no result here shows cause.

## Status

| Part | Status |
|---|---|
| Profiling of all 8 tables (shapes, nulls, keys) | Done |
| Placeholder and quality checks | Partly done (two columns checked, more listed in the log) |
| Ten findings (Findings 1 to 10 in the log) | Done (Finding 9 is still being refined) |
| Cleaned tables | Not done (cleaning rules are proposed, not applied) |
| Controls for other factors (income, age, and so on) | Not done |
| PostgreSQL star schema | Planned, not started |
| Dashboard | Planned, not started |

## Data

- Source: Kaggle competition **Home Credit Default Risk** (https://www.kaggle.com/c/home-credit-default-risk). The data is **not included** in this repository. Download it from Kaggle after accepting the competition rules, and check the rules before using it.
- Tables used: `application_train`, `application_test`, `bureau`, `bureau_balance`, `previous_application`, `POS_CASH_balance`, `credit_card_balance`, `installments_payments`, plus the column description file.
- Only `application_train` has the `TARGET` column (1 = payment difficulties). Baseline default rate: **8.07%** (24,825 of 307,511 applications).

## Main results

All numbers are in the data quality log with their evidence and checks.

1. **A hidden placeholder.** `DAYS_EMPLOYED` has no nulls but holds the value 365243 (about 1,000 years) for 55,374 applications (18.01%). The same rows have `ORGANIZATION_TYPE = 'XNA'`. They default at 5.40%, against 8.66% for all other rows, so they were flagged and kept, not dropped.
2. **Join row explosion, found and fixed.** Joining `application_train` to `bureau` turned 307,511 rows into 1,509,345, because one application has many bureau credits. Credits were rolled up to one row per application before joining.
3. **Unknown months.** 54.92% of the 817,395 loans in `bureau_balance` have at least one month with unknown status, and about half of the delayed loans contain one (derived).
4. **Limited coverage.** Only 29.99% of applications (92,231) have any monthly credit history in `bureau_balance`. 43,041 loans in that table cannot be linked to any application.
5. **Credit history and default.** Applications grouped by their worst recorded delay across previous credits:

   | Worst delay | Default rate |
   |---|---|
   | none recorded | 7.22% |
   | mild | 9.88% |
   | heavy | 10.80% (small group of 2,611, wide uncertainty) |
   | no monthly history | 8.04% |

   Default rises with the severity of past delay. This is an association only.
6. **Gender and contract type.** Men default more than women on cash loans (10.58% against 7.18%); on revolving loans the gap is small (5.83% against 5.31%) and may be noise. Gender is a sensitive field, and this describes the data only. It is not a suggestion to use gender in credit decisions.
7. **Previous loan term.** Applicants whose longest earlier cash or point-of-sale loan had 37 to 48 instalments default at 10.52%. Loan size may explain part of this and was not controlled.
8. **Payment-to-income ratio.** Only 33 applications (0.01%) have a regular payment above their recorded income. Default rate differs by under 1 point across payment-share bands (7.81% up to 20%, 8.61% over 20% to 40%, 8.27% over 40%), a weak signal. The periods of the two columns are not yet verified.
9. **Past refusals.** 32.62% of applicants had at least one earlier refused application at Home Credit. They default at 10.32%, against 7.08% for applicants with an approved earlier application and no refusal and 5.96% for applicants with no earlier application. Association only; the number of earlier applications is not yet controlled.
10. **Credit card utilization.** Among the 28.26% of applicants with earlier Home Credit card records, those with at least one month above 90% of the limit default at 11.31%, against 5.94% for card holders who never did and 7.84% for applicants without card records. Association only; the measure mixes in the length of card history and is being refined.
11. **Car and house ownership.** Applicants who own a car default less (7.04% to 7.33%) than those without one (8.28% to 8.99%) and have higher average income, so income may explain it (untested). House ownership shows no consistent pattern.

## Method

- Many-to-one tables were reduced to one row per entity (loan, then application) before any join.
- Every join and aggregation was checked against row counts and totals (for example, groups must add up to 307,511 applications and 24,825 defaults).
- Several key results were cross-checked with a second independent query or source (for example the loan categories and the status counts).
- Each log item is marked **verified**, **derived** or **hypothesis**, and mistakes found along the way are recorded in the log.

## Limits

- Association, not cause. Most findings have no controls for other factors.
- "None recorded" means no recorded delay, not paid on time.
- The meaning of the status codes in `bureau_balance` (closed, unknown, delay levels) is from memory and not yet confirmed from the column description.
- Results for credit history describe only the 29.99% of applications that have it.
- An application ID is not a person ID, so unique customers cannot be counted.

See section 5 of the log for the full list.

## Repository contents

- `README.md`: this file.
- `data_quality_log.md`: the full log with evidence, decisions, limits, corrections and open items.
- `notebooks/`: the notebook(s) containing the SQL and pandas code (export from Kaggle).
- `.gitignore`: keeps the raw data out of the repository.

## How to reproduce

1. Download the data from the Kaggle competition page and place the CSV files in a local `data/` folder (git-ignored), or attach the competition data to a Kaggle notebook.
2. Install Python with `pandas` and `duckdb`.
3. Run the notebook. Queries are written in DuckDB SQL on pandas DataFrames.

## Next steps

Apply the cleaning rules in one script with automated checks, control for contract type and income, build a PostgreSQL star schema, and add a dashboard.
