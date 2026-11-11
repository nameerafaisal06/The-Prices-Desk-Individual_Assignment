Log and reflections

Part 3: CHECK
1. Before-and-After Audit Table

| Metric / Stat | Raw Dataset (`raw`) | AI Code (Uncorrected) | Final Clean Dataset (`clean`) |
| :--- | :--- | :--- | :--- |
| **Total Rows** | 97 | 95 | **95** |
| **Missing Prices (`NaN`)** | 1 (Row 5) | 1 (Row 5) | **2** (Row 5 + Row 30) |
| **Valid Price Count (`count`)** | 96 | 94 | **93.00** |
| **Mean Price (PKR)** | N/A (String) | 317.73 PKR | **213.63 PKR** |
| **Std Dev (PKR)** | N/A | 1018.09 PKR | **134.33 PKR** |
| **Min Price (PKR)** | N/A | 99.50 PKR | **99.50 PKR** |
| **25% Percentile** | N/A | 150.00 PKR | **149.00 PKR** |
| **Median / 50% (PKR)** | N/A | 200.00 PKR | **200.00 PKR** |
| **75% Percentile** | N/A | 302.50 PKR | **298.00 PKR** |
| **Max Price (PKR)** | `"9999"` | 9999.00 PKR | **1250.00 PKR** (Row 17) |
| **Exact Duplicates** | 1 (Row 96) | 0 | **0** |
| **Subset Duplicates** | 1 (Row 8) | 0 | **0** |

-------------------------------------------------------------------------

2. Line-by-Line Code Verdicts

Line 1: `clean = clean.drop_duplicates()`
Verdict: KEEP
Reason: Dropped row 96 because it was an exact copy of row 8 (`2026-01-08, Staple A, City 1, 99.5`). Brings row count down from 97 to 96.
Check: No actual unique rows were deleted.

Line 2: `clean["item"] = clean["item"].str.strip().str.title()`
Verdict: KEEP
Reason: Fixed capitalization and trailing spaces like turning `"oil b "` into `"Oil B"`.
Check: Must do this before checking subset duplicates, otherwise python won't catch `"oil b"` and `"Oil B"` as the same thing.

Line 3: `clean = clean.drop_duplicates(subset=["date", "item", "city"])`
Verdict: KEEP
Reason: Removed 1 duplicate record logged twice for the exact same date, item, and city. Drops total rows from 96 to 95.
Check: Kept all valid single entries.

Line 4: `clean["price"] = pd.to_numeric(clean["price"].astype(str).str.replace(",", ""), errors="coerce")`
Verdict:KEEP
Reason:Fixed row 17 (`"1,250"`) into float `1250.0`, left row 5 missing as `NaN`, and turned the price column into numbers so `describe()` actually works.
Check: Didn't drop any actual numbers.

Line 5: `clean["price"] = clean["price"].replace(9999.0, np.nan)` (Added Human Step)
Verdict: MODIFY (added human step)
Reason:The AI missed that `9999.0` in row 30 is just a placeholder code for missing data. Leaving it shot the average price up to 317.73 PKR. Replacing it with `NaN` dropped the real average to 213.63 PKR.
Check: `9999.0` is clearly wrong for Sweetener D since its normal price range is 149–184.5 PKR.

Line 6: `clean["date"] = pd.to_datetime(clean["date"])`
Verdict: KEEP
Reason: Converted text dates to pandas datetime format.
Check: All date values stayed the same.

---

3. Item Check

Oil B: 24
Pulse C: 24
Sweetener D: 24
Staple A:23 *(25 raw entries minus 1 exact duplicate and 1 subset duplicate)
Total Rows:95

Audit Result: Everything lines up with the expected counts (12 weeks × 2 cities = 24 entries per item; Staple A has 23 because 1 entry was missing from the raw sheet). No labels or numbers were secretly changed.

--------------------------------------------------------

Part 4: REFLECT 

1. Decisions Differing from AI
The 9999.0 Outlier: AI only looks at syntax, so it thought `9999.0` was a real price. I caught it manually because it completely wrecked the mean and max in `describe()`. Changing it to `NaN` makes way more sense statistically and logically.


2. What I Would Have Missed Alone
Spotting things like `"1,250"` hidden in a string column across 97 rows by eye is easy to miss. Using `str.replace(",", "")` in pandas cleaned it up instantly.
