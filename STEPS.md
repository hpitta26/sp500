# Updating the S&P 500 Dataset

Run these steps in order whenever you want to refresh the data.

## Pipeline

```
sp500.ipynb → sp500.csv
sp500_changes_since_2019.csv + base historical CSV → sp500_historical.ipynb
  → S&P 500 Historical Components & Changes(MM-DD-YYYY).csv
PIT to Ticker Delta.ipynb → sp500_ticker_start_end.csv
```

## Step 1 — Fetch current index (Wikipedia)

**Notebook:** `sp500.ipynb`

- Scrapes the [Wikipedia S&P 500 list](https://en.wikipedia.org/wiki/List_of_S%26P_500_companies)
- **Output:** `sp500.csv`

## Step 2 — Add any new index changes

**File:** `sp500_changes_since_2019.csv`

1. Check the last row in this file (current last change: `2026-01-14`).
2. Compare against [Wikipedia selected changes](https://en.wikipedia.org/wiki/List_of_S%26P_500_companies#Selected_changes_to_the_list_of_S%26P_500_components).
3. Google/search for exact effective dates — Wikipedia is incomplete.
4. Append new rows:

```csv
date,add,remove
2026-03-15,"NEWTICK","OLDTICK"
```

- `add` / `remove`: comma-separated tickers, or leave empty if none.
- One row per change event.

## Step 3 — Rebuild historical point-in-time file

**Notebook:** `sp500_historical.ipynb`

- Merges the base `S&P 500 Historical Components & Changes.csv` with `sp500_changes_since_2019.csv`
- Compares the last row against `sp500.csv`
- **If `diff` is not empty** → go back to Step 2 and add missing changes
- **If `diff` is `[]`** → export is valid
- **Output:** `S&P 500 Historical Components & Changes(MM-DD-YYYY).csv` (today's date)

## Step 4 — Regenerate ticker start/end file

**Notebook:** `PIT to Ticker Delta.ipynb`

1. Update the input filename in cell 1 to the new file from Step 3:

```python
df = pd.read_csv("S&P 500 Historical Components & Changes(MM-DD-YYYY).csv")
```

2. Run all cells (~7–8 min).
3. **Output:** `sp500_ticker_start_end.csv`

| Column | Meaning |
|--------|---------|
| `start_date` | First day ticker was in the index |
| `end_date` | First day ticker was removed (blank = still active) |

Multiple rows per ticker = left and re-entered the index.

## Sanity checks

- [ ] Last historical date matches your newest row in `sp500_changes_since_2019.csv`
- [ ] `sp500_historical.ipynb` diff is `[]`
- [ ] Tickers with blank `end_date` match `sp500.csv`
- [ ] Recent removals have correct `end_date` values

## Dependencies

```bash
pip install pandas wikipedia tqdm lxml html5lib
```
