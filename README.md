# Prediction-json-data-
Prediction json data (extracted from Jon Becker Prediction Market Analysis Dataset). It contains Polymarket and Kalshi market and trade data. Link to Source: https://github.com/Jon-Becker/prediction-market-analysis


# Prediction Markets → Dune via LiveFetch

Uses Dune's `http_get()` LiveFetch function to query pre-aggregated
JSON files hosted on GitHub — **no paid Dune plan required**.

---

## How It Works

```
GitHub Actions (daily, free)
  ↓
Downloads Jon's Parquet dataset from Cloudflare R2
  ↓
build_json.py aggregates it into small JSON files (< 4 MB each)
  ↓
Commits json_data/*.json to this repo

Dune (free plan, any time)
  ↓
http_get('https://raw.githubusercontent.com/YOU/REPO/main/json_data/kalshi_monthly_volume.json')
  ↓
Parses JSON inline with json_parse() + UNNEST()
  ↓
Your query results and dashboard — always up to date
```

---

## Setup: 4 Steps

### Step 1 — Create a GitHub repo and add these files

```
your-repo/
├── .github/workflows/build_json.yml
├── build_json.py
├── requirements.txt
├── json_data/              ← empty folder, Actions will fill it
│   └── .gitkeep
└── .gitignore
```

Create the empty folder placeholder:
```bash
mkdir json_data && touch json_data/.gitkeep
git add json_data/.gitkeep
```

### Step 2 — Enable Actions write permissions

Your repo → **Settings → Actions → General → Workflow permissions**
→ **"Read and write permissions"** → Save

### Step 3 — Trigger the first build

**Actions → "Rebuild JSON for Dune LiveFetch" → Run workflow**

This takes 30–90 minutes (downloading 36 GB). Once done, you'll see
`json_data/*.json` files committed to your repo.

### Step 4 — Use the queries in Dune

Open `dune_queries.sql` — paste any query into a new Dune query editor.

**Replace `only1angelnath` and `prediction-json-data`** with your actual GitHub
username and repo name in every URL.

Example:
```sql
-- Before:
http_get('https://raw.githubusercontent.com/only1angelnath/prediction-json-data/main/json_data/kalshi_monthly_volume.json')

-- After (your actual values):
http_get('https://raw.githubusercontent.com/alice/pred-markets/main/json_data/kalshi_monthly_volume.json')
```

---

## Available JSON Files

| File | Contents | Rows |
|------|----------|------|
| `kalshi_monthly_volume.json` | Contracts traded per month | ~60 |
| `kalshi_calibration.json` | Price bucket → actual resolution rate | ~18 |
| `kalshi_top_markets.json` | Top 100 Kalshi markets by volume | 100 |
| `kalshi_price_distribution.json` | Volume at each YES price (1–99¢) | 99 |
| `kalshi_resolution_by_month.json` | YES/NO resolution counts per month | ~120 |
| `poly_monthly_volume.json` | Polymarket USDC volume per month | ~60 |
| `poly_top_markets.json` | Top 100 Polymarket markets by USDC | 100 |
| `poly_price_distribution.json` | Volume at each price bucket (0–1) | ~20 |
| `index.json` | List of all files + last updated time | — |

All files are fetched live at query time — always reflects the latest
GitHub Actions run.

---

## Limits to Know

- Each `http_get()` call has a **4 MB response limit** and **5 second timeout**
- That's why data is pre-aggregated, not raw rows
- You can add more JSON files to `build_json.py` for custom aggregations
- Multiple `http_get()` calls in one query are fine (see Query 8 for an example)

---

## Adding a Custom Aggregation

Edit `build_json.py` and add a new block following the same pattern:

```python
print("\n[X] My custom analysis")
if not kt.empty:
    my_data = kt.groupby("something").agg(...).reset_index()
    save("my_custom_file", my_data.to_dict(orient="records"))
```

Then add the matching Dune query following the same `http_get()` + `UNNEST()` pattern
shown in `dune_queries.sql`.
