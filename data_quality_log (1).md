# Data quality log: Home Credit Default Risk

*Last updated: 25 October 2026*

This log records what was checked in the 8 Home Credit data tables (plus the column description file), the evidence, and the decision taken. Every item is marked **verified** (backed by printed output and cross-checked), **derived** (calculated from printed output, not printed directly) or **hypothesis** (not yet tested).

**Status**
- Done: table shapes, null profile, placeholder values in two columns, key checks, six analyses (Findings 1 to 6), join and coverage checks.
- Not done: no cleaned tables were produced (the rules in section 3 are proposed, not applied), no predictive model, no PostgreSQL schema, no dashboard. Open work is in section 8.
- No rows were deleted at any point.

## 1. Tables and keys

| Notebook name | Kaggle file | Rows | Columns |
|---|---|---|---|
| app_train | application_train.csv | 307,511 | 122 |
| app_test | application_test.csv | 48,744 | 121 |
| bureau | bureau.csv | 1,716,428 | 17 |
| bureau_balance | bureau_balance.csv | 27,299,925 | 3 |
| previous_app | previous_application.csv | 1,670,214 | 37 |
| pos_cash_balance | POS_CASH_balance.csv | 10,001,358 | 8 |
| credit_card | credit_card_balance.csv | 3,840,312 | 23 |
| installments | installments_payments.csv | 13,605,401 | 8 |
| homecredit_columns_description | HomeCredit_columns_description.csv | 219 | 5 (includes a stray `Unnamed: 0`) |

**Verification (verified):** shapes were printed with `df.shape` and match the counts derived earlier from null count / null fraction for the 7 data tables where derivation was possible and for the description file (219). `bureau_balance` could not be derived (no nulls) and is recorded from the direct print. The 8 data tables hold 58,489,893 rows in total; app_train is 0.53% of them. app_test has one column fewer than app_train, which is `TARGET`.

**Keys (verified from column lists):** `SK_ID_CURR` links application, bureau, previous_app, pos_cash, credit_card and installments. `SK_ID_PREV` links previous_app to pos_cash, credit_card and installments. `SK_ID_BUREAU` links bureau to bureau_balance. All key columns have 0 nulls in every table where they appear. **`SK_ID_BUREAU` is unique in `bureau` (verified):** grouping by it gives 1,716,428 groups, equal to the 1,716,428 rows, so each credit appears once and a join from the loan-level table cannot multiply rows. **`SK_ID_CURR` is unique in `app_train` (verified):** `select distinct` returns 307,511 values, equal to the row count. `bureau` has 305,811 distinct `SK_ID_CURR` values for 1,716,428 credits (about 5.6 credits per applicant, derived). Other duplicate-key and orphan-key checks are still open.

**Target (verified):** `TARGET` exists in app_train only. app_test has no `TARGET`, so any analysis of default must use app_train (307,511 applicants). app_test cannot be used for labelled results.

**Source documentation (verified; table-level lines from the Kaggle Data page, not the column-level lines):**
- `application_train` and `application_test`: one row represents one loan in the data sample. Only the train file has `TARGET`.
- `bureau`: all previous credits from other institutions reported to the credit bureau, for clients who have a loan in the sample, one row per credit.
- `bureau_balance`: one row per month of history of each credit in `bureau`, only for months where some history is observable.
- `POS_CASH_balance` and `credit_card_balance`: monthly snapshots of previous point-of-sale and cash loans, and of credit cards, that the applicant had with Home Credit.
- `previous_application`: all previous Home Credit applications of clients who have a loan in the sample, one row per previous application. The page does not say whether refused applications are included.
- `installments_payments`: one row for every payment made plus one row for each missed payment; one row is one instalment of one previous Home Credit credit.

## 2. Null profile

The first output printed a **fraction**, not a percentage (0.0026 means 0.26%). All percentages below are fraction x 100, with the table's row count as denominator.

| Table | Column(s) | Nulls | Null % |
|---|---|---|---|
| app_train | OWN_CAR_AGE | 202,929 | 65.99 |
| app_train | EXT_SOURCE_1 | 173,378 | 56.38 |
| app_train | EXT_SOURCE_3 | 60,965 | 19.83 |
| app_train | EXT_SOURCE_2 | 660 | 0.21 |
| app_train | OCCUPATION_TYPE | 96,391 | 31.35 |
| app_train | AMT_REQ_CREDIT_BUREAU_* (6 columns, identical count) | 41,519 each | 13.50 |
| app_train | OBS/DEF_30/60_CNT_SOCIAL_CIRCLE (4 columns, identical count) | 1,021 each | 0.33 |
| app_train | 47 housing columns (AVG, MODE, MEDI variants plus 5 categorical) | 145,755 to 214,865 | 47.40 to 69.87 |
| bureau | AMT_ANNUITY | 1,226,791 | 71.47 |
| bureau | AMT_CREDIT_MAX_OVERDUE | 1,124,488 | 65.51 |
| bureau | DAYS_ENDDATE_FACT | 633,653 | 36.92 |
| bureau | AMT_CREDIT_SUM_LIMIT | 591,780 | 34.48 |
| bureau | AMT_CREDIT_SUM_DEBT | 257,669 | 15.01 |
| bureau | DAYS_CREDIT_ENDDATE | 105,553 | 6.15 |
| bureau | AMT_CREDIT_SUM | 13 | 0.0008 |
| previous_app | RATE_INTEREST_PRIMARY, RATE_INTEREST_PRIVILEGED | 1,664,263 each | 99.64 |
| previous_app | AMT_DOWN_PAYMENT, RATE_DOWN_PAYMENT | 895,844 each | 53.64 |
| previous_app | NAME_TYPE_SUITE | 820,405 | 49.12 |
| previous_app | 5 DAYS_* columns + NFLAG_INSURED_ON_APPROVAL | 673,065 each | 40.30 |
| previous_app | AMT_GOODS_PRICE | 385,515 | 23.08 |
| previous_app | AMT_ANNUITY / CNT_PAYMENT | 372,235 / 372,230 | 22.29 |
| credit_card | 6 drawings columns (AMT and CNT, ATM, OTHER, POS) | 749,816 each | 19.52 |
| credit_card | AMT_PAYMENT_CURRENT | 767,988 | 20.00 |
| credit_card | AMT_INST_MIN_REGULARITY, CNT_INSTALMENT_MATURE_CUM | 305,236 each | 7.95 |
| installments | DAYS_ENTRY_PAYMENT, AMT_PAYMENT | 2,905 each | 0.021 |
| pos_cash_balance | CNT_INSTALMENT / CNT_INSTALMENT_FUTURE | 26,071 / 26,087 | 0.261 |
| bureau_balance | all 3 columns | 0 | 0 |

All other columns in all tables show 0 nulls.

Observations (verified as printed):
- The 14 housing base variables each have one identical null count across their AVG, MODE and MEDI versions (for example APARTMENTS: 156,061 in all three).
- previous_app AMT_ANNUITY and CNT_PAYMENT differ by 5 nulls; pos_cash CNT_INSTALMENT and CNT_INSTALMENT_FUTURE differ by 16.
- credit_card AMT_PAYMENT_CURRENT has 18,172 more nulls than the drawings group (767,988 minus 749,816).
- Source spellings to keep in mind when renaming: `YEARS_BEGINEXPLUATATION_*` (app tables), `AMT_RECIVABLE` (credit_card).

## 3. Issues and decisions

| # | Issue | Status | Decision |
|---|---|---|---|
| 1 | Null counts were shown as fractions with no label | verified | Report as percentages with the table's rows as denominator |
| 2 | Identical null counts across groups of columns suggest nulls arrive together (housing block, bureau-request block, drawings block, previous_app terms block) | hypothesis | Not written as fact. Same count does not prove the same rows. Test with a row-level check (open item 1) |
| 3 | `0 nulls` does not mean `clean`: placeholder values are not detected by a null check | verified for `DAYS_EMPLOYED` and `CODE_GENDER` (issues 7 and 8), other columns still to check | Run sentinel and placeholder checks before any cleaning decision (open item 2) |
| 4 | Description file has a stray `Unnamed: 0` column (an exported index) | verified | Drop it, or read with `index_col=0`. `Special` column is 60.73% null (133 of 219) and not used in analysis |
| 5 | Nulls in credit_card drawings and previous_app terms may mean "not applicable" or "zero", not "unknown" | hypothesis | Decide only after checking against `NAME_CONTRACT_STATUS`, balance and drawing counts |
| 6 | Installments: DAYS_ENTRY_PAYMENT and AMT_PAYMENT both null in 2,905 rows (0.021%), possibly unpaid or unrecorded instalments | hypothesis, supported by the documentation | The Data page says missed payments have their own rows, so these may be the missed payments. Check that both columns are null on the same rows and what the due dates look like before flagging |
| 7 | `DAYS_EMPLOYED` has 0 nulls but holds the placeholder 365243 (about 1,000 years) | verified | 55,374 applicants (18.01% of 307,511). Default rate 5.40% for placeholder rows against 8.66% for all others (weighted result 8.07%, matches the baseline). Proposed rule: keep every row, add flag `days_employed_placeholder` (`yes`/`no`), and in the cleaned table set the value to NULL so averages are not distorted. Not yet applied. Because the two groups default at different rates, the flag is kept as information, not just removed |
| 8 | `CODE_GENDER` has the value `XNA` | verified | 4 rows (202,448 F + 105,059 M + 4 XNA = 307,511). Proposed: flag and keep, no filling |
| 9 | `ORGANIZATION_TYPE` has the value `XNA` on exactly the same rows as the `DAYS_EMPLOYED` placeholder | verified | 55,374 rows with `XNA`, 55,374 with `days_employed = 365243`, and 55,374 with both conditions, so the two groups are identical (one group of 18.01% of applicants with both fields missing, not two problems). Proposed: one flag `employment_info_missing` covers both, keep every row, set both fields to NULL in the cleaned table. Not yet applied |

No cleaning has been applied yet. Nothing above justifies dropping rows.

## 4. Findings

Definitions: **applicant** = one row of `app_train` (307,511). **Longest previous POS/cash term** = `max(cnt_instalment)` per applicant over all their `pos_cash_balance` rows (active or not), reduced to one row per applicant before joining, so no join fan-out. Buckets assume monthly instalments (to confirm in the description file). **Default rate** = share with `TARGET = 1`.

### Finding 1: default rate by longest previous POS/cash term

| Bucket | Applicants | Defaults | Default rate |
|---|---|---|---|
| 1 to 12 instalments | 148,313 | 11,951 | 8.06% |
| 13 to 24 | 78,851 | 5,975 | 7.58% |
| 25 to 36 | 30,858 | 2,631 | 8.53% |
| 37 to 48 | 18,441 | 1,940 | 10.52% |
| over 48 | 12,957 | 1,119 | 8.64% |
| no POS history | 18,091 | 1,209 | 6.68% |
| **Total** | **307,511** | **24,825** | **8.07%** |

- The groups add up to 307,511 applicants and 24,825 defaults. The overall default rate (24,825 / 307,511 = 8.07%) was confirmed directly: `avg(target)` = 0.080729 (verified).
- The relationship is not steady: 8.06, 7.58, 8.53, 10.52, 8.64. The 37 to 48 bucket is the highest (+2.45 points over the overall rate). Applicants with no POS history default least (6.68%).
- Average income and credit amount also rise with the bucket (income 163K to 179K, credit 578K to 622K) and are highest for "no POS history" (income 196,729; credit 725,478). Loan size or product mix may explain the differences.
- This is association, not an effect of the term. No confounder control has been done yet.
- The "no POS history" bucket groups applicants with no rows in `pos_cash_balance` and any with only null terms. The query returned no separate `unknown` bucket, so no applicant had rows with all-null terms that fell outside this label.

### Profile: `bureau_balance` STATUS values (rows are loan-months, not loans)

| STATUS | Rows | Share of rows |
|---|---|---|
| C | 13,646,993 | 49.99% |
| 0 | 7,499,507 | 27.47% |
| X | 5,810,482 | 21.28% |
| 1 | 242,347 | 0.89% |
| 5 | 62,406 | 0.23% |
| 2 | 23,419 | 0.09% |
| 3 | 8,924 | 0.03% |
| 4 | 5,847 | 0.02% |
| **Total** | **27,299,925** | **100.00%** |

- The eight values add up to 27,299,925, the `bureau_balance` row count, so there are no nulls and no other values. `C` and `X` are uppercase.
- Statuses 1 to 5 together are 342,943 rows (1.26%). Status 5 has more rows than statuses 2, 3 and 4 combined (62,406 against 38,190). Hypothesis, not tested: status 5 is a state a loan stays in once it gets there, so it repeats every month. The meaning of each code is from memory and still to be confirmed in the description file.
- At loan level, 448,876 of 817,395 distinct loans (54.92%) have at least one `X` month (verified from a loan-level count; 817,395 is the number of rows after `GROUP BY sk_id_bureau`, and the six test categories add up to it). All 398,776 unclassified loans contain an `X` (count of unclassified with `x_flag > 0` = 398,776), so the missing rule is the `X` rule and nothing else. The other 50,100 loans with an `X` (448,876 minus 398,776) cannot be in the `safe` group, whose rule requires no `X`, so they sit in the four delay groups (103,264 loans in total); that is 48.5% of delayed loans with at least one unknown month (derived from verified counts).
- Counts are loan-months. Loan-level categories are in Finding 2.
- **Independent check (Kaggle Data Explorer panel, screenshot):** C 50%, 0 27%, and "Other" 6,153,425 rows. Our own counts for X, 1, 2, 3, 4 and 5 add up to the same 6,153,425 (5,810,482 + 242,347 + 23,419 + 8,924 + 5,847 + 62,406), so the status distribution is confirmed from two sources. A public third-party notebook prints the same eight counts, a third match. The panel shows `MONTHS_BALANCE` ranging from -96 to 0 (histogram axis, confirm with a min/max query), so month 0 exists and the history spans up to 97 months.
- **Source documentation (verified, table-level line only):** the description file says `bureau_balance` has one row for each month of history of every previous credit reported to the credit bureau, and that rows exist only for months where some history is observable. This confirms one row = one previous credit in one month, and that the number of months differs between credits (it is not a fixed 12). The column-level descriptions of `STATUS` and `MONTHS_BALANCE` have not been read yet, so the code meanings stay unconfirmed.

### Finding 2: credit history categories per loan in `bureau_balance`

Unit: one row per loan (`SK_ID_BUREAU`), 817,395 loans. Three separate columns, each built from month counts per status:
- **delay** = worst status ever among 1 to 5: `none` (no month of 1 to 5), `mild` (a 1, 2 or 3 and no 4 or 5), `heavy` (a 4 or 5 ever).
- **has_x** = any month with status `X` (unknown). **has_c** = any month with status `C` (closed, as of that month).
- `none` means no recorded delay, not proven on time. It includes loans that only have `X` months. Meaning of codes is from memory, still to be confirmed in the description file.

| delay | has_x | has_c | Loans | Share |
|---|---|---|---|---|
| none | no | no | 73,880 | 9.04% |
| none | no | yes | 241,475 | 29.54% |
| none | yes | no | 250,574 | 30.66% |
| none | yes | yes | 148,202 | 18.13% |
| mild | no | no | 13,634 | 1.67% |
| mild | no | yes | 36,581 | 4.48% |
| mild | yes | no | 27,364 | 3.35% |
| mild | yes | yes | 19,394 | 2.37% |
| heavy | no | no | 980 | 0.12% |
| heavy | no | yes | 1,969 | 0.24% |
| heavy | yes | no | 1,359 | 0.17% |
| heavy | yes | yes | 1,983 | 0.24% |
| **Total** | | | **817,395** | **100.00%** |

Totals by delay: none 714,131 (87.37%), mild 96,973 (11.86%), heavy 6,291 (0.77%). Row shares are rounded and can differ from group totals by 0.01.

- **Reconciliation (verified):** the 12 groups add up to 817,395 and no loan has a missing delay value. They also match the first six-category query exactly: none without `X` = 315,355 (old `safe`); none with `X` = 398,776 (old unclassified); mild without `C` = 40,998; mild with `C` = 55,975; heavy without `C` = 2,339; heavy with `C` = 3,952.
- 448,876 loans (54.92%) have at least one `X` month; 449,604 (55.00%) have at least one `C` month.
- 50,100 delayed loans contain an `X` (46,758 mild + 3,342 heavy), 48.5% of the 103,264 delayed loans, so delay groups are not free of unknown months.
- Heavy delay is rare: 6,291 loans (0.77%).
- No link to default has been tested yet: loans are not applicants and `bureau_balance` has no applicant ID.

### Finding 3: coverage when linking `bureau_balance` loans to applicants

Link path: loan table (`SK_ID_BUREAU`) to `bureau` (has `SK_ID_BUREAU` and `SK_ID_CURR`) to applicants. `SK_ID_BUREAU` is unique in `bureau`, so this join cannot multiply rows.

| Join | Rows |
|---|---|
| Loan table (one row per loan) | 817,395 |
| Inner join to `bureau` | 774,354 (94.73%) |
| Left join to `bureau` | 817,395 (row count preserved, no fan-out) |
| Loans with `SK_ID_CURR` null after left join | 43,041 (5.27%) |

- The numbers reconcile: 817,395 minus 774,354 = 43,041. `SK_ID_CURR` has no nulls in `bureau` (null profile), so a null after the left join means the loan ID has no row in `bureau` (verified).
- 43,041 loans in `bureau_balance` cannot be linked to any applicant, so they cannot be used in any default comparison. They stay in Finding 2, which is about loans. Proposed rule: keep them in the loan-level table, flag them, exclude them from the applicant roll-up.
- Derived: because loan IDs are unique, 774,354 `bureau` credits have monthly history, which is 45.11% of the 1,716,428 credits in `bureau`. The other 942,074 credits (54.89%) have no rows in `bureau_balance`.
- A left join from `app_train` to `bureau` returns 1,509,345 rows for 307,511 applicants: one row per applicant-credit pair (fan-out), so credits must be rolled up to one row per applicant first (verified).
- Inner join `app_train` to `bureau` = 1,465,325 rows; left join = 1,509,345. The difference, **44,020**, is the number of `app_train` applicants with no credit in `bureau` (derived: `SK_ID_CURR` is unique in `app_train`, so each unmatched applicant adds exactly one row to the left join). So **263,491 applicants (85.68%)** have at least one credit in `bureau` and 44,020 (14.32%) have none.
- Derived: 1,716,428 minus 1,465,325 = 251,103 credits (14.63%) belong to applicants who are not in `app_train`; 305,811 minus 263,491 = 42,320 applicants in `bureau` are not in `app_train`. This is fewer than the 48,744 applicants in `app_test`, consistent with them being test applicants (untested).
- Still to confirm: a direct count of distinct `app_train` applicants with a `bureau` record (expected 263,491). The final join must start from `app_train` and keep all 307,511 rows.
- Applicant-level roll-up (one row per `SK_ID_CURR`, inner join to `app_train` so unlinked loans and non-train applicants drop out, worst delay = maximum of none 0, mild 1, heavy 2): **92,231** `app_train` applicants (29.99% of 307,511) have at least one credit with monthly history in `bureau_balance` (verified from output).
- Derived coverage groups of the 307,511 applicants: no credit in `bureau` 44,020 (14.32%); credits in `bureau` but none with monthly history 171,260 (55.69%); at least one credit with monthly history 92,231 (29.99%). Any result built on `bureau_balance` describes only the last group, see section 5.

### Finding 4: worst delay per applicant (applicants with monthly credit history)

Unit: one row per `SK_ID_CURR` in `app_train` who has at least one credit with monthly history in `bureau_balance` (92,231 applicants, 29.99% of 307,511). Worst delay = the worst loan-level delay across the applicant's linked loans (heavy is worse than mild, mild is worse than none). Loans with no applicant link (43,041) are excluded. Unknown (`X`) months are not yet included at applicant level.

| Worst delay | Applicants | Share of 92,231 |
|---|---|---|
| none | 61,179 | 66.33% |
| mild | 28,441 | 30.84% |
| heavy | 2,611 | 2.83% |
| **Total** | **92,231** | **100.00%** |

- The counts add up to 92,231 and the shares to 100.00% (verified).
- 31,052 applicants (33.67%) have at least one delayed credit (mild or heavy).
- Not comparable with Finding 2: there the unit is loans (12.63% of 817,395 loans delayed), here it is applicants (33.67%). A plausible reason is that an applicant with several loans only needs one delayed loan to land in mild or heavy, but this is not tested, and the two populations also differ.
- `none` means no recorded delay, not proven on time. Only 29.99% of applicants are covered. No link to default has been tested yet.

### Finding 5: default rate by gender and contract type (`app_train`)

Unit: one application (`SK_ID_CURR`). Default rate = share with `TARGET = 1`. The 4 applicants with `CODE_GENDER = 'XNA'` are excluded from the table (placeholder, issue 8); they have 0 defaults (derived: the four groups hold all 24,825 defaults).

| Gender | Contract type | Applicants | Defaults | Default rate | Gap to 8.07% |
|---|---|---|---|---|---|
| F | Cash loans | 182,800 | 13,127 | 7.18% | -0.89 points |
| M | Cash loans | 95,432 | 10,094 | 10.58% | +2.51 points |
| F | Revolving loans | 19,648 | 1,043 | 5.31% | -2.76 points |
| M | Revolving loans | 9,627 | 561 | 5.83% | -2.24 points |
| | **Total (without XNA)** | **307,507** | **24,825** | | |

- Reconciliation (verified): F 182,800 + 19,648 = 202,448 and M 95,432 + 9,627 = 105,059, the gender totals from the earlier value count; 307,507 + 4 XNA = 307,511. Defaults add up to 24,825.
- Derived totals: F 14,170 of 202,448 = 7.00%; M 10,655 of 105,059 = 10.14%; cash loans 23,221 of 278,232 = 8.35%; revolving loans 1,604 of 29,275 = 5.48%.
- The male minus female gap is 3.40 points in cash loans and 0.52 points in revolving loans. The revolving groups are small (9,627 and 19,648), and a rough margin of error of about 0.3 to 0.5 points each (own estimate, not computed) means the revolving gap is not clearly different from zero.
- Contract mix is similar: 9.16% of men and 9.71% of women have revolving loans (derived), so contract type does not explain the overall gender gap. The gap sits in cash loans.
- Limits: association only. Income, age, job type, loan size and other factors are not controlled. This describes the data. It is not a suggestion to use gender in lending decisions, and gender is a sensitive field.

### Finding 6: default rate by worst credit-history delay (all 307,511 applicants)

Unit: one `app_train` application (`SK_ID_CURR`). Built in three layers: loan-level labels (Finding 2), one row per applicant with the worst delay (Finding 4), then a left join starting from `app_train` so that applicants without monthly history stay in as their own group. Default rate = defaults divided by the size of the group.

| Worst delay | Applicants | Defaults | Default rate | Gap to 8.07% |
|---|---|---|---|---|
| none | 61,179 | 4,418 | 7.22% | -0.85 points |
| mild | 28,441 | 2,810 | 9.88% | +1.81 points |
| heavy | 2,611 | 282 | 10.80% | +2.73 points |
| no history | 215,280 | 17,315 | 8.04% | -0.03 points |
| **Total** | **307,511** | **24,825** | **8.07%** | |

- Reconciliation: defaults add up to 24,825 (verified from output). The query did not print group sizes; they come from Finding 4 (none, mild, heavy) and from 307,511 minus 92,231 (no history, derived), and every rate recomputes exactly from them (for example 4,418 / 61,179 = 7.22%). The four sizes add up to 307,511.
- Derived: applicants with any recorded delay (mild or heavy) default at 3,092 of 31,052 = 9.96%, against 7.22% for none (about 1.38 times). Applicants with monthly history overall: 7,510 of 92,231 = 8.14%, close to the baseline.
- Default rate rises with severity (none 7.22%, mild 9.88%, heavy 10.80%). Applicants with no monthly history sit at the baseline.
- Rough margins of error (own estimate, not computed): about 0.2 points for none, 0.35 for mild and 1.2 for heavy. The gap between none and mild is clearly larger than that; the gap between mild and heavy is not clearly different from zero.
- Limits: association only, no controls (income, age, contract type and others). `none` still includes applicants whose loans have unknown (`X`) months. The worst delay depends on the number of loans, since more loans give more chances of a delay, which is not controlled. Only 29.99% of applicants have monthly history. The meaning of the status codes is still unverified.

### Finding 7: annuity-to-income ratio and default rate (`app_train`)

Unit: one application. Ratio = `AMT_ANNUITY` / `AMT_INCOME_TOTAL` x 100, rounded to 2 decimals. `AMT_ANNUITY` is the regular payment (from memory). The periods of the two columns (for example monthly payment against yearly income) are not yet verified from the column descriptions, so the ratio is not read as a share of monthly income.

| Ratio band | Applications | Share |
|---|---|---|
| up to 20% | 202,463 | 65.84% |
| over 20% to 40% | 97,129 | 31.59% |
| over 40% to 60% | 6,988 | 2.27% |
| over 60% to 80% | 745 | 0.24% |
| over 80% to 100% | 141 | 0.05% |
| over 100% | 33 | 0.01% |
| annuity missing | 12 | 0.00% |
| **Total** | **307,511** | **100.00%** |

- Reconciliation (verified): the bands add up to 307,511. The 12 missing rows match the 12 null `AMT_ANNUITY` values in the null profile. The coarser bands from the first run also match (over 40% to 80%: 6,988 + 745 = 7,733; over 80%: 141 + 33 = 174).
- 33 applications (0.01%) have an annuity above their recorded income. Rows above 100% could be data errors, unusual loans, or a period mismatch between the two columns (not tested).
- Derived: 7,907 applications (2.57%) are above 40%.
- The bands above 40% are small, so they were merged for the default rate. The over-100% group (33 applications) is too small for a rate and is not reported separately.

Default rate by merged band (default rate = defaults divided by the group's own size):

| Ratio band | Applications | Defaults | Default rate | Gap to 8.07% |
|---|---|---|---|---|
| up to 20% | 202,463 | 15,807 | 7.81% | -0.26 points |
| over 20% to 40% | 97,129 | 8,364 | 8.61% | +0.54 points |
| over 40% | 7,907 | 654 | 8.27% | +0.20 points |
| annuity missing | 12 | 0 | 0.00% | too small to read |
| **Total** | **307,511** | **24,825** | **8.07%** | |

- Reconciliation (verified): applications add up to 307,511 and defaults to 24,825.
- The separation is weak. The three bands lie within 0.8 points of each other, and the rate does not rise step by step (7.81%, 8.61%, 8.27%). For comparison, the worst-delay groups in Finding 6 span 3.6 points (7.22% to 10.80%).
- Rough margins of error (own estimate, not computed): about 0.12 points for up to 20%, 0.18 for over 20% to 40% and 0.6 for over 40%. The gap between the first two bands (0.80 points) is larger than these margins; the over-40% band is not clearly different from either neighbour or from the baseline.
- Limits: association only; the periods of the two columns are unverified; income level and contract type are not controlled.

### Finding 8: default rate by past refusal (`previous_app` and `app_train`)

Unit: one application (`SK_ID_CURR`). Previous applications were reduced to one row per `SK_ID_CURR` (counts of rows with `NAME_CONTRACT_STATUS` = Approved, Refused, Unused offer and Canceled), then left joined from `app_train`, so applicants without any previous application stay in as their own group. Each applicant gets one group, checked in this order: no previous application; at least one refused (even if also approved); at least one approved and no refusal; only unused offers; only cancelled. Default rate = defaults divided by the group's own size. Assumption: every `app_train` row is a granted loan (the Data page calls a row one loan in the sample); supported by the documentation, not proven. The meaning of the status values is from common usage, not documentation.

| Group | Applicants | Share | Defaults | Default rate | Gap to 8.07% |
|---|---|---|---|---|---|
| No previous application | 16,454 | 5.35% | 980 | 5.96% | -2.11 points |
| Approved, never refused | 190,370 | 61.91% | 13,472 | 7.08% | -0.99 points |
| At least one refused | 100,294 | 32.62% | 10,352 | 10.32% | +2.25 points |
| Only cancelled | 227 | 0.07% | 8 | 3.52% | too small to read |
| Only unused offer | 166 | 0.05% | 13 | 7.83% | too small to read |
| **Total** | **307,511** | **100.00%** | **24,825** | **8.07%** | |

- Reconciliation (verified): groups add up to 307,511 applicants and 24,825 defaults, and each rate recomputes from the counts. The first version of this analysis had one "none refused" group (190,763 applicants, 13,493 defaults, 7.07%); the split matches it exactly (190,370 + 227 + 166 = 190,763 applicants; 13,472 + 8 + 13 = 13,493 defaults).
- No applicant fell into a leftover "mixed history" group, so every applicant with a previous application has at least one of the four counted statuses (derived; value counts of the status column are still to be checked against 1,670,214).
- Applicants with at least one earlier refusal default at 10.32%, against 7.08% for applicants with an approval and no refusal: 3.24 points higher, about 1.46 times (derived). Rough margins of error (own estimate): about 0.2 points for each of the two large groups, so the gap is much larger than the margin.
- Derived: the refused group is 32.62% of applicants but 41.70% of all defaults (10,352 of 24,825).
- The two small groups (227 and 166 applicants, together 0.21% of the earlier "none refused" group) have margins of roughly 2 and 4 points (own estimate) and are not interpreted. An applicant with both an unused offer and a cancelled application is placed in the "only unused offer" group because of the order of the rules.
- Limits: association only. The number of earlier applications (exposure) is not controlled; an applicant with many earlier applications has more chances of a refusal. Refusal reasons and timing are not examined. Applicants who were refused and never received a loan are not in `app_train` (selection).

### Profile: credit currency in `bureau` (rows are credits, not applicants)

| CREDIT_CURRENCY | Credits | Share |
|---|---|---|
| currency 1 | 1,715,020 | 99.92% |
| currency 2 | 1,224 | 0.07% |
| currency 3 | 174 | 0.01% |
| currency 4 | 10 | 0.00% |
| **Total** | **1,716,428** | **100.00%** |

- The counts add up to the 1,716,428 rows of `bureau` (verified).
- Only 1,408 credits (0.08%) are not in currency 1, so at most 1,408 applicants (at most 0.46% of 307,511, derived) can have such a credit, and fewer of them are in `app_train`. At roughly an 8% default rate a group of that size has a margin of error of at least 1.4 points (own estimate), so a default-rate comparison by currency was judged too small to be useful and was not run. Currencies 3 and 4 (174 and 10 credits) cannot support any rate.
- The values are stored as codes ("currency 1" to "currency 4"). Which one is the home currency is not documented in the data; the idea that currency 1 is the home currency is from outside sources and unverified.

### Finding 9: credit card utilization and default (`credit_card` and `app_train`)

Unit: one application (`SK_ID_CURR`). For each card-month, utilization = `AMT_RECEIVABLE_PRINCIPAL` / `AMT_CREDIT_LIMIT_ACTUAL` x 100, and the month is flagged if utilization is above 90 and both the balance and the principal are non-zero. Flagged months are summed per applicant (one row per applicant), then left joined from `app_train`, so applicants without any card rows are their own group. The meanings of the three amount columns are inferred from column names and a few printed rows (limit = card limit, principal = amount still owed, balance a little higher than principal), not read in the description file. The 90% cut-off is a choice.

Group = number of flagged card-months per applicant:

| Group | Applicants | Share | Defaults | Default rate | Gap to 8.07% |
|---|---|---|---|---|---|
| No card rows | 220,606 | 71.74% | 17,291 | 7.84% | -0.23 points |
| 0 flagged months | 42,756 | 13.90% | 2,539 | 5.94% | -2.13 points |
| 1 to 10 | 28,500 | 9.27% | 3,465 | 12.16% | +4.09 points |
| 11 to 20 | 7,928 | 2.58% | 854 | 10.77% | +2.70 points |
| 21 to 30 | 3,620 | 1.18% | 338 | 9.34% | +1.27 points |
| 31 to 40 | 1,880 | 0.61% | 149 | 7.93% | -0.14 points |
| over 40 | 2,221 | 0.72% | 189 | 8.51% | +0.44 points |
| **Total** | **307,511** | **100.00%** | **24,825** | **8.07%** | |

- Reconciliation (verified): groups add up to 307,511 applicants and 24,825 defaults, and each rate recomputes from the counts. The empty group label in the output means no rows in `credit_card` (derived: every applicant with card rows appears in the roll-up). 86,905 applicants (28.26%) have card rows (derived).
- Derived summary (counts added, not rates averaged):

| Group | Applicants | Defaults | Default rate |
|---|---|---|---|
| Card rows, never above 90% | 42,756 | 2,539 | 5.94% |
| Card rows, at least one month above 90% | 44,149 | 4,995 | 11.31% |
| No card rows | 220,606 | 17,291 | 7.84% |

  Applicants with at least one month above 90% default about 1.9 times as often as card holders who never did (5.37 points apart; rough margins of error about 0.3 and 0.2 points, own estimate).
- The finer groups are not a clean measure of "maxing out": the group is a count of card-months (one row per card per month, so an applicant with several cards has more rows), not a share. The average number of card-month rows per applicant rises with each group: 25.22 (0 flagged), 38.62 (1 to 10), 56.93 (11 to 20), 70.63 (21 to 30), 78.85 (31 to 40), 87.09 (over 40) (verified from output). So the groups track the amount of card history. Default rate falls from 12.16% to 7.93% as the group rises from 1 to 10 up to 31 to 40, then rises slightly to 8.51%; whether the amount of card history explains this decline is not tested. The group with the shortest history (0 flagged months, 25.22) also has the lowest default rate (5.94%), so a short history is not simply riskier. A share of months (flagged months divided by card-month rows) would separate maxing out from the amount of history.
- "Never above 90%" also includes cards that were never used (zero balance).
- Limits: association only; the timing of the flagged months (recent or long ago) is not used; a limit of zero or less was not guarded; the cut-off is a choice; applicants with a card record are a minority (28.26%).

### Finding 10: default rate by car and house ownership (`app_train`)

Unit: one application. Groups from `FLAG_OWN_CAR` and `FLAG_OWN_REALTY` (only the values Y and N occurred, since the four groups cover all rows). Written and checked by the author without help.

| Group | Applicants | Share | Defaults | Default rate | Gap to 8.07% | Average income |
|---|---|---|---|---|---|---|
| No car, no house | 61,972 | 20.15% | 5,572 | 8.99% | +0.92 points | 152,473 |
| No car, house | 140,952 | 45.84% | 11,677 | 8.28% | +0.21 points | 155,540 |
| Car and house | 72,360 | 23.53% | 5,306 | 7.33% | -0.74 points | 195,986 |
| Car, no house | 32,227 | 10.48% | 2,270 | 7.04% | -1.03 points | 197,131 |
| **Total** | **307,511** | **100.00%** | **24,825** | **8.07%** | | |

- Reconciliation (verified): groups add up to 307,511 applicants and 24,825 defaults, and each rate recomputes from the counts.
- Car owners default less (7.04% and 7.33%) than applicants without a car (8.28% and 8.99%), and their average income is higher by about 26% to 29% between groups (derived). A possible explanation is income, which is not tested. Averages are sensitive to extreme incomes; medians were not computed.
- House ownership is not consistent: among applicants without a car, house owners default 0.71 points less (8.28% against 8.99%, larger than the rough margin of about 0.3 points, own estimate); among car owners the difference is 0.29 points in the other direction (7.33% against 7.04%), within the margin.
- Derived: applicants without a car number 202,924 (61,972 + 140,952), while `OWN_CAR_AGE` has 202,929 nulls, 5 more. So the nulls are almost, but not exactly, the applicants without a car.
- Limits: association only; income, age, region and other factors are not controlled.

## 5. Limits

- **Association only.** None of the findings show cause. Only Finding 5 looks inside a second variable (contract type); income, age, job type, loan size and similar factors are not controlled anywhere.
- **Coverage.** Only 29.99% of applicants (92,231 of 307,511) have monthly credit history in `bureau_balance`. 14.32% (44,020) have no credit in `bureau` at all, and 43,041 loans (5.27%) in `bureau_balance` cannot be linked to any applicant. Findings 2 to 4 and 6 describe the covered group.
- **"None" means no recorded delay, not paid on time.** It includes loans and applicants with unknown (`X`) months: 54.92% of loans have at least one `X` month.
- **Worst delay depends on the number of loans.** An applicant with more loans has more chances of a delay. This is not controlled.
- **Small groups.** The heavy-delay group (2,611 applicants) and the revolving-loan groups are small. Margins of error quoted in the findings are rough estimates, not computed.
- **Applicant means one row of the application table.** The Data page describes one row as one loan in the sample. `app_train` has no separate person ID column, so unique customers cannot be counted.
- **Gender is a sensitive attribute.** Finding 5 describes this dataset only. It is not a recommendation to use gender in credit decisions.
- **Scope.** Only `app_train` has `TARGET` (`app_test` is not used for results). The findings describe this Kaggle dataset, not lenders or borrowers in general.
- **Not built.** No cleaned tables, no predictive model, no PostgreSQL schema, no dashboard.

## 6. Assumptions not yet verified

- Status codes in `bureau_balance`: `C` = closed, `X` = unknown, `0` = no delay, `1` to `5` = increasing delay. These meanings are from memory. A public third-party notebook (not official documentation) uses the same meanings (`1` to `4` as days past due, `5` labelled sold by its author) and prints the same eight status counts as this log, which supports but does not confirm them. The column-level lines of the description file have not been read; only the table-level lines from the Data page were.
- `MONTHS_BALANCE` counts months backwards from the application date. The range of -96 to 0 comes from the Kaggle data panel and is not yet confirmed with a min/max query.
- Instalment buckets in Finding 1 assume monthly instalments.
- `bureau` probably contains applicants from `app_test` (the arithmetic is consistent with it, not tested).
- The reason why 43,041 loans in `bureau_balance` have no row in `bureau` is unknown.
- Identical null counts across column groups are assumed to mean the same rows are null (not tested).

## 7. Corrections made along the way

- The first null profile printed an unlabelled fraction. Relabelled as percent.
- The first output computed `isna().sum()` twice per column and printed all columns, including about 300 zero-null lines. Replaced with a DataFrame of table, column, dtype, rows, nulls and null percent.
- First POS/cash query put NULL terms into the `4+ years` bucket (a NULL fails every `WHEN`), added an explicit `unknown` branch and a `> 48` branch.
- A typo (`a,amt_income_total`) selected the whole table alias into the CTE; fixed to `a.amt_income_total`.
- `count(...) / 12` was rejected: it does not measure loans or applicants. Replaced by one row per applicant.
- First version averaged a previous loan's term with the current application's amounts and counted monthly snapshots, which over-weights long loans. Replaced by `max(cnt_instalment)` per applicant joined to `app_train`.
- The inner join dropped 18,091 applicants with no POS/cash history (289,420 of 307,511 covered). A left join with a `no pos history` bucket restored all 307,511.
- First loan-category version left 398,776 of 817,395 loans unclassified (48.79%): no rule for loans with an `X`. Found by testing that every unclassified loan had `x_flag > 0`, then replaced by three separate columns (delay, has_x, has_c).
- Three-column draft had a typo (`greatest(flag_4,flag_4)` instead of `flag_5`), tested "ever" with `> 1` instead of `> 0`, tied "none" to `flag_0 > 0` and had no `else`, so some loans got no delay value. Fixed and checked against the old totals.
- First default-by-delay query was rejected. The defaults of its three groups added up to 43,311 against 24,825 in total, which is impossible: `sum(target)` was taken over loan rows (one row per loan, so a customer's default was counted once per loan). It also divided each group's defaults by the grand total of customers instead of the group's own size, and it joined `app_train` after the roll-up with a left join from the loan side, so customers outside `app_train` stayed in and the no-history group never appeared. Fix: roll up to one row per customer first, then start from `app_train` and group by label with count(*) and sum(target).
- Row counts were first quoted from memory, then derived from null output, then confirmed with a direct `df.shape` print. All matched.

## 8. Open items

1. Row-level check that grouped null columns are null on the same rows (housing block, bureau-request block, social-circle block, drawings block, previous_app terms block).
2. More placeholder and quality checks: 365243 in the `DAYS_*` columns of `previous_app`; confirm that 55,374 equals all rows with `days_employed > 0`; implausible values in `DAYS_*`; extreme `AMT_INCOME_TOTAL`; whitespace and empty strings in text columns.
3. Who are the 55,374 placeholder applicants (`name_income_type` breakdown)? Hypothesis only so far.
4. Key integrity: orphan keys between child tables and parents; a direct count of the 263,491 `app_train` applicants with a `bureau` record.
5. Installments: confirm that `DAYS_ENTRY_PAYMENT` and `AMT_PAYMENT` are null on the same rows, check whether those rows are the missed payments the documentation mentions (look at their due dates), and check `AMT_PAYMENT` against `AMT_INSTALMENT`.
6. Finding 1: check whether the 37 to 48 bucket gap survives within bands of `amt_credit`, look at the most frequent term values at the bucket edges (12, 24, 36, 48), add medians for income, and count how many of the 18,091 applicants have no rows at all in `pos_cash_balance`.
7. `bureau_balance`: read the column-level description of `STATUS` and `MONTHS_BALANCE`; check where `X` months sit in time; months per loan (minimum, median, maximum); min and max of `MONTHS_BALANCE`.
8. Finding 6: print the group sizes in the query output, add the unknown-month flag at applicant level, check default rate by number of linked loans, and control for contract type and income.
9. Apply the proposed cleaning rules (flags, placeholders set to NULL) in one script with automated row-count and flag-count checks, and save cleaned tables.
10. Planned, not started: PostgreSQL star schema and a dashboard.
11. Finding 7 follow-up: read the `AMT_ANNUITY` and `AMT_INCOME_TOTAL` lines in the description CSV (periods), check zero and extreme incomes, and check whether the pattern holds inside cash and revolving loans and inside income bands.
12. Finding 8 follow-up: value counts of `NAME_CONTRACT_STATUS` in `previous_app` (must add to 1,670,214); average number of previous applications per group and a comparison within bands of that number (exposure); refusal reasons (`CODE_REJECT_REASON`) and timing (`DAYS_DECISION`); confirm the definition of `TARGET` in the description CSV.
13. `bureau` credit type vs default (next candidate): credits per `CREDIT_TYPE` must add to 1,716,428; roll up to one row per applicant with yes/no flags per type, left join from `app_train`; groups overlap because an applicant can hold several types, so they will not add up to 307,511.
14. Finding 9 follow-up: read the description lines of `AMT_BALANCE`, `AMT_CREDIT_LIMIT_ACTUAL` and `AMT_RECEIVABLE_PRINCIPAL`; use the share of flagged months instead of the count (card-months per group already printed); check the ratio distribution (rows above 100%, limit of zero or less); use timing (`MONTHS_BALANCE`) to see whether recent maxing out matters more.
15. Finding 10 follow-up: compare car and no-car applicants within income bands (does the gap remain?); use median income; check the 5 extra `OWN_CAR_AGE` nulls (count rows with `FLAG_OWN_CAR` = Y and a null car age).
