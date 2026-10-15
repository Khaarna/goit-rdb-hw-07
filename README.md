# Relational Databases — Homework 7

## Dataset

**Name:** NYC TLC Yellow Taxi Trip Records

**Source:** [NYC Taxi & Limousine Commission](https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page)

**Dataset file:** `yellow_tripdata_2024-03.parquet`

The dataset contains information about taxi trips in New York City, including pickup and dropoff times, locations, trip distances, fares, and passenger counts.

For this homework, a reproducible sample of 500,000 trips from March 2024 was used (`random_state=42`).

After cleaning and aggregation, the daily time-series dataset contains:

| Metric | Result |
|---|---:|
| Sampled trips | 500,000 |
| Pickup zones | 250 |
| Daily records | 5,445 |
| Calendar-spine records | 7,750 |
| Date range | March 1–31, 2024 |

A separate synthetic event-log dataset containing 50,000 records was also generated for the JSONB exercises.

## Tools

- Google Colab
- PostgreSQL 16 (`pgserver`)
- Python
- SQLAlchemy
- pandas
- PyArrow

## Homework Tasks

### 1. Data Loading and Preparation

Downloaded the NYC Taxi dataset from the official source.

Cleaned invalid fare and distance values, selected a reproducible sample, and loaded it into PostgreSQL.

Created daily aggregates by pickup zone and date.

### 2. Time-Series Feature Engineering

Created a calendar spine to fill missing dates and calculated:

- **Lag features:** `trips_lag_1d`, `trips_lag_7d`, `trips_lag_30d`.
- **Rolling features:** 7-day average fare and 30-day fare volatility.
- **Expanding features:** cumulative trips and expanding average trip count.

All window functions use `PARTITION BY pickup_zone_id`.

The final features are available through `v_t7_hw_taxi_features`.

### 3. JSONB Event Log

Generated 50,000 synthetic events with five event types:

- `page_view`
- `click`
- `add_to_cart`
- `checkout`
- `recommendation_shown`

Each event type has its own JSONB payload structure.

Practiced JSONB containment (`@>`), key existence (`?`, `?|`), and JSONPath queries.

### 4. JSONB Index Performance

Created a GIN index for containment queries and an expression B-tree index for device lookups.

Used `EXPLAIN (ANALYZE, BUFFERS)` to compare query performance before and after indexing.

| Query | Before index | After index |
|---|---:|---:|
| JSONB containment (GIN) | 57.149 ms | 6.410 ms |
| Device lookup (B-tree) | 17.073 ms | 19.131 ms |

The GIN index improved containment query performance by approximately 8.9 times and occupied 1,144 kB of storage.

The B-tree expression index did not improve performance in this particular execution.

### 5. JSONB vs Normalized Tables

Created a normalized table containing 10,000 checkout events.

Compared the same aggregation query using JSONB and typed PostgreSQL columns.

| Method | Execution time |
|---|---:|
| JSONB | 15.159 ms |
| Normalized | 5.501 ms |

The normalized approach was approximately 2.8 times faster in this execution.

All performance measurements are based on individual runs and may vary depending on the environment.

### 6. Reflection

Summarized the main findings about time-series feature engineering, JSONB indexing, query performance, data modeling, and architectural trade-offs.

## How to Run

1. Open `HW07.ipynb` in Google Colab.
2. Select **Runtime → Restart session and run all**.
3. Wait for all cells to complete.

The notebook automatically installs dependencies, downloads the NYC Taxi dataset, starts PostgreSQL, creates tables, generates synthetic events, and executes the SQL queries.

No manual database setup is required.

## Repository Structure

```text
goit-rdb-hw-07/
├── README.md
└── HW07.ipynb
```

The source Parquet file is downloaded automatically, so it does not need to be included in the repository.
