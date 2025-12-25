# CoffeeKing Expansion Playbook (Yelp -> SQLite -> SQL Insights)

A consulting-style analytics project that turns the Yelp Open Dataset into a lightweight **SQLite database** and produces an actionable **market-entry playbook** for a hypothetical coffee brand, **CoffeeKing**.

The core deliverable is a simple, explainable recommendation engine that answers:
**Where should CoffeeKing pilot first, and what concept positioning is most likely to "win" in that market?**

---

## Audience

**CoffeeKing Growth / Expansion Team** (market selection + concept strategy)

---

## Business Questions

1. **Where is the market strongest** for highly visible, highly rated coffee-related businesses?
2. **Which concept positioning to win** (e.g., coffee + alcohol vs. market/retail vs. food-forward vs. pure coffee)?
3. For each city, **What is the best "city x concept" bet** to maximize early discoverability?

---

## Data (Not Included)

This repo does **not** redistribute the Yelp dataset.

- Source: **Yelp Open Dataset**
- Input files expected locally:
  - `yelp_academic_dataset_business.json`
  - `yelp_academic_dataset_review.json`

> Note: Raw JSON files are intentionally excluded from version control. The generated SQLite DB is also excluded.

---

## Database Tables (SQLite)

Built from the Yelp JSON files:

- `coffee_business`
  Coffee-related subset from `business.json` (keyword filter on categories)
- `coffee_reviews`
  Reviews filtered to `coffee_business` from `review.json` (chunk-loaded)
- `coffee_business_enriched`
  Adds rule-based **concept tags** derived from Yelp categories
- `coffee_analysis`
  Business-level rollups (review volume, avg rating, visibility score)
- `coffeeking_reco_v2`
  City-level recommendations (recommended concept + winner rate + sample flag)

---

## Definitions (Business Metrics)

### Visibility Score
To make the analysis actionable for a hypothetical **CoffeeKing** brand, I created a Visibility Score that combines popularity and quality:

- **Visibility Score = log(1 + review_volume) × average_review_stars**

This rewards businesses that are both **well-known** (many reviews) and **well-liked** (high stars), while reducing extreme review-count outliers via `log(1 + x)`.

### Winner (Top 10%)
I define “winners” as the **top decile** of Visibility Score:

- **winner = visibility_score ≥ P90(visibility_score)**  
  (equivalently: `NTILE(10) = 1` when sorting by `visibility_score` DESC)

### Winner Rate (by segment)
For any segment (e.g., city, or city × concept):

- **winner_rate = (# winners in segment) / (total businesses in segment)**

### Small-N Reliability Flag
Some city×concept segments are small and should be treated as directional signals:

- `SMALL_N_THRESHOLD = 30`
- If `n_business < 30`, mark as: **"SMALL N (treat as signal)"**

---

## Deliverable Preview (Schema)

The final recommendation output is stored in SQLite as:

- **`coffeeking_reco_v2`**

Schema preview (columns you should expect):

- `state`
- `city`
- `recommended_concept`
- `recommended_winner_rate`
- `recommended_avg_visibility`
- `recommended_n_business`
- `recommended_n_top10pct`
- `sample_flag` *(e.g., SMALL N)*

---

## Key Findings (Interpret with Care)

- **Where the market looks strongest (city-level):**  
  In our sample, higher city-level winner rates appeared in **Santa Barbara (CA)**, **New Orleans (LA)**, and **Saint Louis (MO)** — suggesting these markets may have a higher probability of producing highly visible, highly rated coffee-related businesses.

- **What concept tends to win:**  
  Across most top cities, **coffee+alcohol** is over-represented among winners relative to **market/retail**, implying that an “evening + social” coffee positioning may be a strong go-to-market strategy.  
  One notable exception is **Reno (NV)**, where **market/retail** performs slightly better, suggesting a retail/market destination angle may fit that local market.

- **Recommendation (pilot → scale):**  
  For an initial pilot, CoffeeKing should prioritize **coffee+alcohol** in high-opportunity cities (especially Santa Barbara and New Orleans) and keep **market/retail** as a strategic alternative — particularly in markets like Reno.

---

## Deliverables

### Notebooks
- `notebooks/01_initial_look.ipynb`  
  Quick dataset preview and early exploration
- `notebooks/02_build_sqlite_db.ipynb`  
  Builds the SQLite database from Yelp JSON files (chunk loading for reviews)
- `notebooks/03_CoffeeKing_Consulting_Deliverables.ipynb`  
  Produces the consulting-style analysis:
  - Visibility Score + winners
  - City and city×concept winner rates
  - Final recommendation table saved as `coffeeking_reco_v2`

---

## How to Reproduce (Local)

### 1) Set up environment
Create a clean environment and install dependencies:

```bash
pip install -r requirements.txt
```

### 2) Download Yelp Open Dataset and place files
Place the following files in `data_raw/`:

- `yelp_academic_dataset_business.json`
- `yelp_academic_dataset_review.json`

> Note: This project uses the Yelp Open Dataset for educational/portfolio purposes. The dataset is not included in this repository and is governed by Yelp’s own terms.

### 3) Build the SQLite database
Run:

- `notebooks/02_build_sqlite_db.ipynb`

This creates:

- `db/coffeeking.db` *(not tracked by git)*

### 4) Generate insights + recommendations
Run:

- `notebooks/03_CoffeeKing_Consulting_Deliverables.ipynb`

This writes the final deliverable table into SQLite:

- `coffeeking_reco_v2`

---

### Repo Structure

```text
Coffeeking-Yelp/
├── notebooks/
│   ├── 01_initial_look.ipynb
│   ├── 02_build_sqlite_db.ipynb
│   └── 03_CoffeeKing_Consulting_Deliverables.ipynb
├── data_raw/                 # local only (NOT tracked) - Yelp JSON lives here
├── db/                       # local only (NOT tracked) - SQLite DB lives here
├── .gitignore
├── README.md
└── requirements.txt
```

## Limitations / Next Steps

- **Causality:** This analysis is descriptive and does not establish causal effects.
- **City-level rollups:** Results may be influenced by tourism, category noise, and Yelp usage patterns.
- **Better segmentation:** Future iterations could incorporate price tier, neighborhood density, or review-text themes.

Potential extensions:
- Add a risk-adjusted winner rate (e.g., confidence intervals for small-N segments)
- Cluster competitors by concept profile within each market
- Build a lightweight dashboard to filter and compare recommendations

---

## License
MIT License (code and documentation only).  
The Yelp dataset is governed by its own terms and is **not** included in this repository.
