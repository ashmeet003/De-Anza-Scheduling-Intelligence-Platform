# DynamoDB (Database)

## What Is This? (Plain English)

DynamoDB is Amazon's database service. Think of it like a collection of spreadsheets in the cloud that can handle millions of lookups per second. We use it to store:

1. **Pathway data** — What courses each degree program requires, organized by quarter
2. **Class schedules** — Every section offered (days, times, rooms, instructors) for each term
3. **Conflict results** — Precomputed answers to "which courses clash in this pathway?"

The reason we use DynamoDB instead of a regular database: it's serverless (no server to manage), scales automatically, and charges only for what we use.

## The 4 Tables

### 1. `deanza-pathways` — "The Course Catalog"

Stores all 235 pathways exactly as parsed from the PDF pathway maps.

**Think of each row as**: One degree/certificate program with its full 2-year plan.

| Field | What It Contains | Example |
|-------|-----------------|---------|
| `pathway_id` (key) | Unique ID from source file | `2025 BIOL maps.pdf#2` |
| `program_name` | Full title | "Biology for Transfer -- Associate in Science for Transfer" |
| `credential_type` | What you earn | "Associate in Science for Transfer (AS-T)" |
| `village` | Academic grouping | "Health and Life Sciences" |
| `years` | The actual course plan | Nested: year_1 → fall/winter/spring → required_courses[] |
| `additional_notes` | Fine print | Prerequisites, substitution rules |

### 2. `deanza-pathways-normalized` — "The Translated Catalog"

Same as above, but with course codes translated to match the schedule format.

**Why we need this**: Pathways say "EWRT 1A" but the schedule says "ENGL C1000" (De Anza renamed courses). This table has the translated versions so we can match them.

### 3. `deanza-class-schedule` — "The Schedule of Classes"

Stores every single class section for every term. This is the raw schedule data.

**Think of each row as**: One meeting time for one section.

| Field | What It Contains | Example |
|-------|-----------------|---------|
| `term_code` (key) | Which quarter | `202622` (= Fall 2025) |
| `crn_seq` (sort key) | Section + sequence | `21574#0` |
| `subject` | Department | `CHEM` |
| `course_number` | Raw number | `D001A` |
| `days` | Meeting days | `MW` |
| `times` | Time range | `11:30 am-12:45 pm` |
| `instructor` | Who teaches it | `David Gray` |
| `room` | Where | `SC1102` or `ONLINE` |

**Important**: One section (CRN) can have multiple rows if it meets at different times. For example, a Chemistry class might have a lecture MWF and a lab TR — that's 2 rows with the same CRN.

**Term code format**: `YYYYQQ` where YY=year and QQ=quarter (22=Fall, 32=Winter, 42=Spring). So `202632` = Winter 2026.

### 4. `deanza-pathway-conflicts` — "The Answer Sheet"

Stores precomputed conflict results so we don't have to recalculate every time.

**Think of each row as**: "For this pathway, in this quarter, here are all the conflicts we found."

| Field | What It Contains | Example |
|-------|-----------------|---------|
| `pathway_id` (key) | Which pathway | `2025 BIOL maps.pdf#2` |
| `quarter_key` (sort key) | Which quarter | `year_1#fall` |
| `conflict_count` | How many course pairs overlap | `3` |
| `total_pairs` | Total pairs checked | `10` |
| `conflict_percentage` | Rate | `30.0` |
| `conflicts` | List of specific overlapping pairs | [{course_a, course_b, sections...}] |

## How Data Gets In

| Table | How It's Filled | Source |
|-------|----------------|--------|
| `deanza-pathways` | `python scripts/pathways/load.py` | JSON file parsed from PDFs |
| `deanza-pathways-normalized` | `python scripts/course_pairing/normalize_pathways.py` | Transforms from table 1 |
| `deanza-class-schedule` | `python scripts/class_schedule/load.py` | Excel/CSV schedule files |
| `deanza-pathway-conflicts` | `python scripts/pathway_conflicts/build_conflicts.py` | Computed by the conflict engine |

## How Data Gets Out (Common Queries)

| "I want to..." | What Happens | Table Used |
|----------------|-------------|------------|
| Browse all pathways | Scan the whole table | `deanza-pathways` |
| See one pathway's plan | Get by pathway_id | `deanza-pathways` |
| Find MATH sections in Fall 2025 | Query by term + subject | `deanza-class-schedule` (GSI) |
| See conflicts for Biology pathway | Query by pathway_id | `deanza-pathway-conflicts` |

## Creating the Tables

```bash
python scripts/create_tables.py
```

This reads the table definitions from `scripts/common.py` and creates all 4 tables in your AWS account. It's safe to run multiple times — it skips tables that already exist.
