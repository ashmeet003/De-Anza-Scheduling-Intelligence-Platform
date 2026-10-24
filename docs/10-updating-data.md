# Updating Data (Future Maintenance Guide)

## When Would You Need to Update?

- A **new quarter's class schedule** is published (e.g., Fall 2027)
- **New pathway maps** are released (new programs, updated course requirements)
- An existing schedule is **revised** (sections added/removed/rescheduled mid-term)
- The college introduces **new terms or academic years** beyond 2027

---

## Overview of the Update Process

```
New Data Files (Excel/CSV/PDF)
    │
    ▼
Upload to S3 Bucket (or local data/ folder)
    │
    ▼
Run Loader Scripts (puts data into DynamoDB)
    │
    ▼
Run Conflict Pipeline (recomputes all conflicts)
    │
    ▼
Export Updated Results (for web app + dashboard)
    │
    ▼
(Optional) Rebuild/redeploy web app with new static data
```

---

## Part 1: Updating Class Schedules

### What You'll Get

De Anza's registrar publishes class schedules as Excel or CSV files, typically one per term. They look like this:

### Required File Format

The loader expects these columns (column names can vary slightly — the loader normalizes them):

| Column | Description | Example |
|--------|------------|---------|
| Term | Term identifier or name | `202822` or `Fall 2027` |
| CRN | Course Reference Number (unique per section) | `27469` |
| Subject | Department code | `MATH` |
| Course Number | Internal number | `D001A` |
| Section | Section identifier | `01Y` |
| Days | Meeting days | `MW`, `TR`, `MTWR`, `F` |
| Times | Time range | `08:30 am-10:45 am` |
| Instruction Method | Delivery mode | `IP`, `HB`, `OC`, `OS`, `O1` |
| Instructor | Name | `James Mailhot` |
| Room | Location | `G7`, `SC1102`, `ONLINE` |
| Division | Division code | `2PS` |

**Important formatting rules:**
- Times must be in `HH:MM am/pm-HH:MM am/pm` format (12-hour with am/pm)
- Days use single letters: M=Monday, T=Tuesday, W=Wednesday, R=Thursday, F=Friday, S=Saturday
- If a section has multiple meeting patterns (lecture + lab), include one row per pattern with the same CRN
- Online sections should have `ONLINE` in the Room column

### Term Code Format

```
Format: YYYYQQ

YYYY = Academic year (the year the academic year STARTS in fall)
QQ   = Quarter code:
       22 = Fall
       32 = Winter  
       42 = Spring

Examples:
  202622 = Fall 2025 (academic year 2026 starts fall 2025)
  202732 = Winter 2027
  202842 = Spring 2028
```

### Steps to Load a New Schedule

```bash
# 1. Place the file in the data folder (or upload to S3)
#    File naming convention: "FA 2027.csv", "WI 2028.csv", "SP 2028.csv"
#    Or Excel: "FA 2027.xlsx"

# 2. If using S3, upload:
aws s3 cp "FA 2027.csv" s3://your-bucket-name/schedules/FA 2027.csv

# 3. Run the loader
python scripts/class_schedule/load.py          # From local data/ folder
python scripts/class_schedule/load.py --s3     # From S3 bucket
```

The loader will:
- Read the file
- Normalize column names
- Generate the term_code if not explicit
- Write each row to the `deanza-class-schedule` DynamoDB table
- Skip rows with missing critical fields (no CRN, no times)

### Verifying the Load

```bash
# Quick check: query the new term
python scripts/class_schedule/query.py --term 202822 --subject MATH
```

---

## Part 2: Updating Pathway Maps

### What You'll Get

Pathway maps come as PDF files from De Anza (one PDF per department, multiple programs per PDF).

### Steps to Add New Pathways

```bash
# 1. Place new PDFs in a local folder

# 2. Parse PDFs into JSON (this extracts the course plan structure)
python scripts/course_pairing/pathway_parser.py --input /path/to/pdfs --output data/deanza_pathways_updated.json

# 3. Review the output JSON for parsing errors
#    (PDFs are messy — check that courses look right)

# 4. Load into DynamoDB
python scripts/pathways/load.py --file data/deanza_pathways_updated.json

# 5. Normalize course codes (translate to match schedule format)
python scripts/course_pairing/normalize_pathways.py
```

### JSON Format for Pathways

If you're adding pathways manually (without PDF parsing), here's the expected structure:

```json
{
  "village": "Physical Sciences and Technology",
  "program_name": "Chemistry -- Sample Transfer Pattern",
  "credential_type": "Transfer Preparation",
  "years": {
    "year_1": {
      "fall": {
        "required_courses": ["MATH 1A", "ENGL C1000", "CHEM 25"],
        "additional_courses": ["CHEM 25 may be prerequisite for CHEM 1A"]
      },
      "winter": {
        "required_courses": ["MATH 1B", "CHEM 1A"],
        "additional_courses": []
      },
      "spring": {
        "required_courses": ["MATH 1C", "CHEM 1B", "COMM C1000"],
        "additional_courses": []
      }
    },
    "year_2": {
      "fall": { "required_courses": [...], "additional_courses": [...] },
      "winter": { "required_courses": [...], "additional_courses": [...] },
      "spring": { "required_courses": [...], "additional_courses": [...] }
    }
  },
  "additional_notes": ["Some courses have prerequisites..."],
  "source_file": "2027 Chem transfer.pdf",
  "page_number": 1
}
```

---

## Part 3: Recomputing Conflicts

After new schedule or pathway data is loaded, you need to rerun the conflict analysis.

```bash
# Recompute conflicts for ALL pathways (takes several minutes)
python scripts/pathway_conflicts/build_conflicts.py

# Or if you only added one new term and want to be targeted:
python scripts/pathway_conflicts/build_conflicts.py --term 202822
```

### What This Does Behind the Scenes

1. Reads every normalized pathway from DynamoDB
2. For each pathway's each quarter:
   - Maps the quarter to a term code (e.g., year_1#fall → whichever term that maps to)
   - Pulls all sections for the required courses from DynamoDB
   - Sends them to the conflict Lambda (or calls the engine directly)
   - Stores the result in `deanza-pathway-conflicts` table
3. Overwrites any existing results for that pathway+quarter combo

---

## Part 4: Regenerating Reports & Web Data

After conflicts are recomputed:

```bash
# Generate the analysis report (used by dashboard)
python scripts/pathway_conflicts/analyze_conflicts.py
# Output: analysis-dashboard/analysis_report.json

# Generate department-level aggregation (used by dashboard)
python scripts/pathway_conflicts/aggregate_by_department.py
# Output: web-app/src/data/department_conflicts.json

# Export conflict data for the web app
python scripts/pathway_conflicts/export_web_data.py
# Output: web-app/src/data/pathway_conflicts.json
```

---

## Part 5: Updating the Web App & Dashboard

### Web App (React)

The web app bundles its data at build time. After exporting new data:

```bash
cd web-app
npm run build
```

Then deploy the new `dist/` folder to your hosting.

### Dashboard

The dashboard HTML file has data embedded inside it. To update it with new data, you'd need to:

1. Regenerate `department_conflicts.json` and `analysis_report.json` (steps above)
2. Update the embedded `CONFLICT_DATA` object in the HTML file with the new JSON
3. Update `SCHEDULE_MULTITERM` with any new term's schedule data
4. Copy the updated HTML to `web-app/public/dashboard.html`

This is currently a manual step. In the future, a script could automate embedding updated JSON into the HTML template.

---

## Part 6: Adding a Brand New Academic Year

If De Anza adds a full new year (e.g., 2028-2029):

### 1. Define New Term Codes

| Quarter | Term Code |
|---------|-----------|
| Fall 2028 | `202922` |
| Winter 2029 | `202932` |
| Spring 2029 | `202942` |

### 2. Update the Web App Config

In `web-app/src/config.js`, add entries to the `TERMS` array:

```javascript
{ code: "202922", label: "Fall 2028", yearKey: "year_1", quarterKey: "fall" },
{ code: "202932", label: "Winter 2029", yearKey: "year_1", quarterKey: "winter" },
{ code: "202942", label: "Spring 2029", yearKey: "year_1", quarterKey: "spring" },
```

### 3. Update the Dashboard

Add the new term codes to the `TERMS` and `TERM_OPTIONS` objects in the HTML file.

### 4. Load the New Schedules

Follow Part 1 above with the new term's CSV/Excel files.

### 5. Update Pathway Mapping

If new pathways reference these new terms, update the quarter-to-term mapping in `build_conflicts.py` so `year_1#fall` maps to the correct new term code.

---

## Part 7: S3 Bucket Organization

If using S3 for data storage, here's the recommended folder structure:

```
s3://your-bucket-name/
├── schedules/
│   ├── FA 2025.csv
│   ├── WI 2026.csv
│   ├── SP 2026.csv
│   ├── FA 2026.csv
│   └── ... (add new files here)
├── pathways/
│   ├── deanza_pathways.json
│   └── deanza_pathways_2028.json  (if updated)
└── raw-pdfs/
    ├── 2025 BIOL maps.pdf
    ├── 2025 CHEM transfer.pdf
    └── ... (source PDFs for reference)
```

---

## Quick Reference: Full Update Checklist

When a new quarter's schedule is published:

- [ ] Get the CSV/Excel file from the registrar
- [ ] Verify columns match expected format (see Part 1)
- [ ] Upload to S3 or place in local `data/` folder
- [ ] Run `python scripts/class_schedule/load.py`
- [ ] Run `python scripts/pathway_conflicts/build_conflicts.py`
- [ ] Run `python scripts/pathway_conflicts/analyze_conflicts.py`
- [ ] Run `python scripts/pathway_conflicts/aggregate_by_department.py`
- [ ] Run `python scripts/pathway_conflicts/export_web_data.py`
- [ ] Rebuild web app: `cd web-app && npm run build`
- [ ] Update dashboard HTML if needed
- [ ] Deploy updated files

---

## Troubleshooting

| Problem | Likely Cause | Fix |
|---------|-------------|-----|
| Loader skips rows | Missing CRN, times, or subject | Check source file for blank cells |
| "Course not found" in conflicts | Course code mismatch | Run normalization again, check mapping |
| New term not showing in web app | Config not updated | Add term to `config.js` TERMS array |
| Conflict counts seem wrong | Stale data in DynamoDB | Re-run build_conflicts for that term |
| Dashboard shows old data | Embedded data not refreshed | Regenerate and re-embed JSON |
