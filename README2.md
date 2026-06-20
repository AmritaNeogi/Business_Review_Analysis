# Business Review Analysis
### End-to-End Data Pipeline: AWS S3 → Snowflake | Sentiment Analysis | Business Intelligence

---

## Project Overview

This project builds a full data pipeline using the [Yelp Open Dataset](https://business.yelp.com/data/resources/open-dataset/) — a real-world dataset of over 6 million reviews across 150,000+ businesses. The pipeline covers data ingestion from AWS S3 into Snowflake, JSON flattening, sentiment classification using a Python UDF, and business intelligence analysis across 10 analytical questions.

**Key skills demonstrated:** Cloud data ingestion, semi-structured data handling, Snowflake-native UDFs, SQL window functions, data quality validation, and business analysis.

---

## Architecture

```
Yelp Open Dataset (5GB JSON)
        │
        ▼
[Split Data.ipynb]          ← Python: splits large JSON into manageable chunks
        │
        ▼
AWS S3 Bucket               ← yelp-json-ingestion-dev
        │
        ▼
Snowflake External Stage    ← yelp_s3_stage (reusable, credential-managed)
        │
        ▼
Raw Variant Tables          ← yelp_reviews, yelp_businesses
        │
        ▼
Flattened Tables            ← yelp_reviews_table, yelp_businesses_table
        │
        ▼
Sentiment UDF               ← analyze_sentiment() using TextBlob (Python 3.10)
        │
        ▼
Supporting Tables           ← yelp_reviews_enriched, yelp_business_categories
        │
        ▼
Data Quality Table          ← yelp_data_quality_summary
        │
        ▼
Business Analysis           ← 10 analytical questions
```

---

## Tech Stack

| Layer | Tool |
|---|---|
| Cloud Storage | AWS S3 |
| Data Warehouse | Snowflake (BUSINESS_REVIEWS_DB) |
| Warehouse Size | X-Large (LARGEDATA_WH) |
| Data Processing | Python (Jupyter Notebook) |
| Sentiment Analysis | Snowflake Python UDF + TextBlob |
| Query Language | SQL (Snowflake dialect) |

---

## Repository Structure

```
business-review-analysis/
│
├── Split Data.ipynb               ← Splits 5GB JSON into smaller files for S3 upload
├── sentiment analysis function     ← Python UDF: classifies reviews as Positive / Neutral / Negative
├── data ingestion queries          ← Stage creation, COPY INTO, JSON flattening, validation
├── supporting tables               ← yelp_reviews_enriched, yelp_business_categories
├── data quality                    ← yelp_data_quality_summary: null checks, invalid value checks
├── analysis                        ← 10 business questions answered in SQL
├── Overview.png                    ← Project architecture diagram
├── Dataset_User_Agreement.pdf      ← Yelp dataset usage agreement
└── Yelp Dataset Documentation.pdf ← Official dataset documentation
```

---

## How to Run

> **Prerequisites:** Snowflake account, AWS S3 bucket, Python 3.10+

### Step 1 — Split the raw data
Run `Split Data.ipynb` to split the large Yelp review JSON file into smaller chunks and upload to S3.

### Step 2 — Create the Sentiment UDF
Run `sentiment analysis function` in Snowflake **before** the ingestion queries.
This creates the `analyze_sentiment()` UDF used during table creation.

### Step 3 — Ingest data into Snowflake
Run `data ingestion queries` to:
- Create an external stage pointing to your S3 bucket
- Load raw JSON into variant tables (`yelp_reviews`, `yelp_businesses`)
- Flatten JSON into structured tables (`yelp_reviews_table`, `yelp_businesses_table`)

### Step 4 — Create supporting tables
Run `supporting tables` to create:
- `yelp_reviews_enriched` — pre-joined reviews + business data for efficient querying
- `yelp_business_categories` — pre-exploded categories to avoid repeated `LATERAL SPLIT_TO_TABLE` operations

### Step 5 — Validate data quality
Run `data quality` to create `yelp_data_quality_summary` — checks for nulls, invalid star ratings, and date ranges across both core tables.

### Step 6 — Run business analysis
Run `analysis` to execute 10 business questions against the prepared tables.

---

## Data Model

```
yelp_reviews_table          yelp_businesses_table
├── business_id (FK) ──────► business_id (PK)
├── user_id                  ├── name
├── review_date              ├── city
├── review_stars             ├── state
├── review_text              ├── review_count
└── sentiments               ├── stars
                             └── categories
                                      │
                             yelp_business_categories
                             ├── business_id (FK)
                             └── category (exploded)

yelp_reviews_enriched        ← join of reviews + businesses (pre-built for efficiency)
yelp_data_quality_summary    ← quality metrics across both core tables
```

---

## Business Questions Answered

| # | Question | Key Concepts |
|---|---|---|
| 1 | Number of businesses per category | LATERAL SPLIT_TO_TABLE, COUNT DISTINCT |
| 2 | Top 10 users by restaurants reviewed | Category filtering, pre-exploded table |
| 3 | Most popular categories by review count | Category aggregation, JOIN |
| 4 | Top 3 most recent reviews per business | ROW_NUMBER, window functions, tiebreaker |
| 5 | Month with highest review volume | Date formatting, TO_VARCHAR |
| 6 | Percentage of 5-star reviews per business | CASE WHEN, HAVING, percentage calculation |
| 7 | Top 5 most reviewed businesses per city | QUALIFY, ROW_NUMBER, partitioning |
| 8 | Average rating for businesses with 100+ reviews | HAVING, aggregation filtering |
| 9 | Top 10 reviewers and businesses they reviewed | CTE chaining, JOIN back to enriched table |
| 10 | Top 10 businesses by positive sentiment | Sentiment scorecard, full distribution |

---

## Highlights

**Snowflake External Stage** — credentials and S3 path defined once and reused across all `COPY INTO` statements, following the DRY principle and production best practices.

**Python UDF for Sentiment Analysis** — `analyze_sentiment()` runs TextBlob directly inside Snowflake, classifying each review as Positive, Neutral, or Negative without moving data outside the warehouse.

**Pre-built Supporting Tables** — `yelp_business_categories` eliminates repeated `LATERAL SPLIT_TO_TABLE` calls; `yelp_reviews_enriched` eliminates repeated joins, improving query efficiency across all analytical questions.

**Data Quality Layer** — `yelp_data_quality_summary` validates null rates, star rating ranges, and date coverage before analysis begins, reflecting real-world pipeline practices.

**Business Judgment in Queries** — minimum review thresholds applied where relevant (e.g., Q6, Q10) to avoid statistically misleading results from low-volume businesses.

---

## Dataset

- **Source:** [Yelp Open Dataset](https://business.yelp.com/data/resources/open-dataset/)
- **Size:** ~5GB (reviews JSON), split into manageable chunks for S3 upload
- **Usage:** Subject to Yelp's dataset terms — see `Dataset_User_Agreement.pdf`
