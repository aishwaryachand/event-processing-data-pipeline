# Phillies Pitch Tracking Data Pipeline
Names: Aishwarya Chand
Email: aishwarya7811@gmail.com

## Project Overview
This project implements a Python-based data pipeline to ingest, clean, validate, and enrich high-frequency pitch tracking data from a baseball game. The raw input is a nested JSON file (`batch_raw.json`) containing anonymized tracking events.

The pipeline flattens the raw data into a pitch-level tabular dataset (one row per pitch), engineers baseball-relevant features, applies basic domain validation, and produces batter-level performance metrics using SQL. The final outputs are intended to support downstream analytics and research workflows.

---

## Architectural Design
The pipeline is organized into a real-world analytics workflow:

- **Ingestion Layer**  
 Loads raw JSON data from disk and processes pitch-level records without assuming the data is complete or perfectly clean

- **Transformation & Enrichment Layer**  
  Flattens nested structures, extracts key attributes (batter ID, swing indicator, exit velocity), and computes derived features based on observable data patterns.

- **Validation & Testing Layer**  
  Applies basic domain checks (e.g realistic exit velocity ranges) and includes unit tests to validate critical logic such as swing detection.

- **Analytics & Aggregation Layer**  
  Loads the cleaned dataset into an in-memory SQL engine (DuckDB) and executes a single aggregation query to compute hitter-level metrics.

- **Output Layer**  
  Persists clean pitch-level data and exports aggregated batter summaries for downstream use.

---

## Project Structure
```text
phillies_pipeline/
├── data/
│   └── batch_raw.json
├── src/
│   ├── __init__.py
│   ├── pipeline.py
│   └── aggregate.py
├── tests/
│   └── test_pipeline.py
├── output/
│   ├── clean_data.parquet
│   └── batter_summary.csv
├── pytest.ini
├── requirements.txt
└── README.md
```

## How to Run the Code

### 1. Create and activate a virtual environment
```bash
python -m venv venv
source venv/bin/activate
```

### 2. Install dependencies
```bash
pip install -r requirements.txt
```
### 3. Run the data pipeline
```bash
python src/pipeline.py
```
This step ingests the raw JSON, flattens and enriches the data, applies validation, and writes the clean pitch-level dataset.

### 4. Run unit tests
```bash
pytest
```
### 5. Run aggregation
```bash
python src/aggregate.py
```
This step computes batter-level metrics using DuckDB and exports the summary file.



## Output Files


- output/clean_data.parquet
Cleaned, pitch-level dataset with engineered features.

- output/batter_summary.csv
Batter-level aggregated metrics including swing count, whiff rate, and maximum exit velocity

## Feature Engineering Summary

- Exit Velocity: Extracted from summary_acts.hit.speed.mph.

- Is Swing: Derived from the presence of bat tracking data (samples_bat).

- Is Contact (Custom Feature): Derived from is_swing and the presence of exit velocity, indicating whether a swing resulted in ball–bat contact.

## Short Answer Responses
### Feature Justification

- I added an is_contact feature derived from whether a pitch involved a swing and whether exit velocity was recorded. Although I do not come from a sports background, this feature was derived directly from the structure of the tracking data. Bat tracking objectively indicates that a swing occurred, while a recorded exit velocity indicates that the ball was hit. Combining these signals provides a data-driven way to infer contact. This feature separates swings into contact versus non-contact outcomes and enables consistent computation of metrics such as whiff rate using observable data rather than subjective labels.

### Data Quality & Observability

- In a production environment, I would monitor the percentage of swings with missing bat tracking data as a data-quality metric. The pipeline would compute this rate per game or per batch and compare it to a historical baseline. If the rate increased significantly , an automated alert would be triggered and sent to the engineering team via email or Slack with relevant context such as game ID and ingestion time. This allows rapid identification of upstream data or ingestion issues before they affect downstream analytics.

### Advanced SQL Logic

- To identify the top three hardest-hit balls for each batter, I would use a SQL window function such as ROW_NUMBER() or RANK() partitioned by batter_id and ordered by exit velocity in descending order. After ranking batted-ball events within each batter group, I would filter to rows where the rank is less than or equal to three. This approach preserves event-level detail while enabling flexible per-batter comparisons, and it works well with analytical SQL engines like DuckDB.


## AI Usage Disclosure
AI tools were used for guidance on debugging, and documentation. All implementation decisions, logic, and final code were reviewed and written by me.
