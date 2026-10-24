# Data Pipeline (ETL Scripts)

## What Is This? (Plain English)

"ETL" stands for Extract-Transform-Load — a fancy way of saying "get the raw data, clean it up, and put it somewhere useful."

Our raw data comes from two places:
- **PDF files** — De Anza publishes pathway maps as PDFs (one per program)
- **Excel/CSV files** — The registrar publishes class schedules as spreadsheets

These scripts turn that messy raw data into clean, queryable database records and then compute all the conflict analysis.

## The Pipeline (In Order)

Think of it like a factory assembly line:

```
Stage 1: COLLECT RAW DATA
  PDFs → Parse text → JSON file with all pathways
  Excel → Read rows → Ready for upload

Stage 2: LOAD INTO DATABASE
  JSON → Write to DynamoDB (pathways table)
  Excel rows → Write to DynamoDB (schedule table)

Stage 3: NORMALIZE (TRANSLATE)
  "EWRT 1A" in pathway → becomes "ENGL C1000" to match schedule
  "D001A" in schedule → becomes "1A" to match pathway

Stage 4: DETECT CONFLICTS
  For each pathway, for each quarter:
    "What courses are needed?" → Look them up in the schedule → Compare times

Stage 5: ANALYZE & EXPORT
  Aggregate all results → Statistics, reports, web-ready JSON files
```

## Key Scripts Explained

### `scripts/common.py` — Shared Utilities

**What it does**: Provides helper functions that ALL other scripts use.

- Loads your `.env` file (AWS credentials)
- Creates connections to DynamoDB
- Defines the structure of all 4 database tables

**Analogy**: Like a toolbox that every worker on the assembly line shares.

### `scripts/create_tables.py` — Table Setup

**What it does**: Creates the 4 DynamoDB tables in your AWS account.

**When to run**: Once, when setting up the project for the first time.

```bash
python scripts/create_tables.py
```

It's safe to run repeatedly — skips tables that already exist.

### `scripts/class_schedule/load.py` — Schedule Loader

**What it does**: Reads class schedule files (Excel or CSV) and uploads every row to DynamoDB.

**Where data comes from**: Either an S3 bucket or local `data/` folder. Files are named like `FA 2025.csv` (Fall 2025), `WI 2026.csv` (Winter 2026), etc.

**What each row represents**: One meeting time for one section. Example:
```
Term: 202622 | CRN: 21574 | Subject: CHEM | Number: D001A | Section: 1
Days: MW | Times: 11:30am-12:45pm | Instructor: David Gray | Room: SC1102
```

```bash
python scripts/class_schedule/load.py          # From local files
python scripts/class_schedule/load.py --s3     # From S3 bucket
```

### `scripts/pathways/load.py` — Pathway Loader

**What it does**: Reads the `deanza_pathways.json` file (all 238 parsed pathways) and uploads each one to DynamoDB.

**Where data comes from**: The JSON file was created by the PDF parser (below).

```bash
python scripts/pathways/load.py
```

### `scripts/course_pairing/normalization.py` — Course Code Translator

**What it does**: De Anza's schedule uses internal course numbers that don't match what's printed on pathway maps. This script translates between them.

**The problem**:
```
Pathway says: "COMM 1"     → Schedule says: subject=COMM, number=D001.
Pathway says: "MATH 10"    → Schedule says: subject=MATH, number=D010.
Pathway says: "CHEM 1A"    → Schedule says: subject=CHEM, number=D001A
```

**The rules**:
1. Strip the "D" prefix (De Anza internal marker)
2. Remove leading zeros (D010 → 10)
3. Remove trailing dots (D001. → 1)
4. Keep letter suffixes (D001A → 1A)
5. Keep the "C" prefix for statewide courses (C1000 stays C1000)

**Result**: Both sides now say "MATH 10", "CHEM 1A", etc. — they can be matched.

### `scripts/course_pairing/pathway_parser.py` — PDF Parser

**What it does**: Reads De Anza's PDF pathway maps and extracts the course plan.

**How**: Uses the `pdfplumber` library to read text from each page, then identifies the structure (Year 1/2, Fall/Winter/Spring, required courses, additional courses).

**Output**: `deanza_pathways.json` — a JSON file with all 238 pathways.

### `scripts/pathway_conflicts/build_conflicts.py` — The Big Run

**What it does**: The main pipeline script. Goes through EVERY pathway, EVERY quarter, looks up the courses in the schedule, and runs the conflict engine.

**How it works (simplified)**:
```
For each of 235 pathways:
  For each of 6 quarters (year1 fall/winter/spring, year2 fall/winter/spring):
    1. Get the courses needed this quarter
    2. Map the quarter to a term code (year_1#fall → 202622)
    3. Call the conflict checking service
    4. Save the results to DynamoDB
```

**How long it takes**: Several minutes (hundreds of API calls).

```bash
python scripts/pathway_conflicts/build_conflicts.py
```

### `scripts/pathway_conflicts/conflict_client.py` — API Caller

**What it does**: A helper that calls the conflict-checking service. Can work two ways:
- **Direct**: Invokes the Lambda function directly (faster, needs AWS credentials)
- **HTTP**: Calls the API Gateway URL with an API key (works from anywhere)

Used by `build_conflicts.py` above.

### `scripts/pathway_conflicts/analyze_conflicts.py` — Report Generator

**What it does**: After all conflicts are computed, this reads everything from DynamoDB and generates summary statistics:
- Which pathways have the most conflicts?
- Which course pairs clash most often?
- Which terms are worst?
- Per-village and per-credential breakdowns

**Output**: `analysis-dashboard/analysis_report.json`

### `scripts/pathway_conflicts/aggregate_by_department.py` — Department Summary

**What it does**: Groups conflicts by subject/department for the dashboard's overview charts.

**Output**: `web-app/src/data/department_conflicts.json`

### `scripts/pathway_conflicts/export_web_data.py` — Web Export

**What it does**: Exports conflict data in the format the React web app expects.

**Output**: `web-app/src/data/pathway_conflicts.json`

### `scripts/pull_s3_data.py` — Data Downloader

**What it does**: Downloads raw data files from S3 to your local machine for offline work.

## Running the Full Pipeline

```bash
# 1. Create tables (one-time setup)
python scripts/create_tables.py

# 2. Load raw schedule data into DynamoDB
python scripts/class_schedule/load.py

# 3. Load pathway data into DynamoDB
python scripts/pathways/load.py

# 4. Normalize course codes in pathways
python scripts/course_pairing/normalize_pathways.py

# 5. Run conflict detection for all pathways (takes a few minutes)
python scripts/pathway_conflicts/build_conflicts.py

# 6. Generate analysis reports
python scripts/pathway_conflicts/analyze_conflicts.py
python scripts/pathway_conflicts/aggregate_by_department.py
python scripts/pathway_conflicts/export_web_data.py
```

## Environment Requirements

You need a `.env` file in the project root:
```env
AWS_ACCESS_KEY_ID=your_key_here
AWS_SECRET_ACCESS_KEY=your_secret_here
AWS_DEFAULT_REGION=us-west-2
S3_BUCKET=your-bucket-name
CONFLICT_API_URL=https://your-api-url.amazonaws.com/prod/conflicts
CONFLICT_API_KEY=your-api-key
```

Plus Python packages: `pip install -r requirements.txt`
