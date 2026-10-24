# Conflict Engine

## What Is This? (Plain English)

The conflict engine is the core brain of this project. It answers one question: **"If a student enrolls in Course A and Course B in the same quarter, is there ANY section combination where the times overlap?"**

It's like having someone manually compare every possible schedule combination — except it does it in milliseconds for hundreds of courses.

## The Key Files

| File | What It Does | Analogy |
|------|-------------|---------|
| `conflict_engine.py` | The main comparison logic | The person comparing two schedules |
| `time_parsing.py` | Converts "9:30 am-11:20 am" into numbers | A clock reader |
| `section_lookup.py` | Finds sections from the database | A registrar looking up what's offered |
| `lambda_handler.py` | Wraps the engine for AWS Lambda | A receptionist taking requests |
| `plan_handler.py` | The student planner endpoint | A full-service advisor |

## How It Works Step by Step

### Step 1: Parse the Times

Before comparing anything, we need to turn time strings into numbers we can compare.

```
"08:30 am" → 510 minutes since midnight (8*60 + 30)
"01:30 pm" → 810 minutes since midnight (13*60 + 30)
```

Days get turned into sets:
```
"MW"   → {Monday, Wednesday}
"TR"   → {Tuesday, Thursday}  
"MTWR" → {Monday, Tuesday, Wednesday, Thursday}
```

**The overlap rule**: Two time blocks conflict if they share a day AND the times overlap. "Overlap" means: one starts before the other ends AND vice versa. Back-to-back is fine (ending at 10:20 and starting at 10:30 = no conflict).

### Step 2: Understand Sections

A "section" is one specific offering of a course. But one section can meet at multiple times:

```
CHEM 1A, Section 1 (CRN 21574):
  → Monday/Wednesday 11:30am-12:45pm  (Lecture in SC1102)
  → Monday/Wednesday 8:30am-11:20am   (Lab in SC2202)
```

Both time blocks are part of the same section. A student who enrolls in this section is committed to BOTH times.

The engine groups all rows with the same CRN into one section.

### Step 3: Compare Every Pair

For courses A and B:
1. Get all sections of A (maybe 6 sections)
2. Get all sections of B (maybe 8 sections)
3. For each of the 48 possible combinations (6 × 8):
   - Does any time block in Section_A overlap with any time block in Section_B?
   - If yes → this combination is a conflict
   - If no → this combination works

### Step 4: Report Results

Two output modes depending on who's asking:

**Lean mode** (for batch processing):
```
"3 out of 48 section combinations have a time conflict"
+ list of which specific ones clash
```

**Full mode** (for the student planner):
```
"Overlapping pairs": [list of section combos that DON'T work]
"Clear pairs": [list of section combos that DO work]
→ "You have 45 working combinations out of 48 — plenty of flexibility!"
```

## The Section Lookup (`section_lookup.py`)

Before comparing, we need to know what sections exist. This file:

1. Takes: a term code + list of course names (e.g., "Fall 2025", ["MATH 1A", "CHEM 1A"])
2. Queries DynamoDB for all sections of those subjects in that term
3. Normalizes course numbers so they match (D001A → 1A)
4. Returns: organized sections per course, plus any courses not found in the schedule

**Why courses might not be found**: The pathway says "take PHIL 4" but it's not offered that quarter. This is tracked as a "missing course."

## The Plan Handler (`plan_handler.py`)

This is the Lambda that the web app's "Check Conflicts" button calls. It ties everything together:

```
Input: { term: "Fall 2025", courses: ["MATH 1A", "CHEM 1A", "PHYS 4A"] }

Step 1: Look up all sections for each course from DynamoDB
Step 2: For each pair (MATH/CHEM, MATH/PHYS, CHEM/PHYS):
        Run the conflict engine comparison
Step 3: Calculate overall stats:
        - 2 out of 3 pairs have some overlap → 66.7%
        
Output: {
  overlap_percentage: 66.7,
  courses_found: ["MATH 1A", "CHEM 1A", "PHYS 4A"],
  overlapping_pairs: [detailed list],
  clear_pairs: [detailed list]
}
```

## Important Rules

1. **Same-course sections are never compared** — You can only enroll in ONE section of MATH 1A, so comparing MATH 1A sections against each other makes no sense

2. **Online sections may be excluded** — If a section meets in "ONLINE" room with no set meeting time, it can't physically conflict with anything

3. **Multi-row sections must ALL be clear** — If CHEM 1A has a lecture AND lab (2 time blocks), BOTH must not conflict with the other course for that combination to work

4. **Strict overlap** — Times that merely touch (one ends at 10:20, next starts at 10:30) do NOT conflict. Only actual overlaps count.

## Example

```
Can a student take MATH 1A (Sec 3: MW 8:30-10:45am) 
                and CHEM 1A (Sec 1: MW 11:30am-12:45pm, MW 8:30-11:20am)?

MATH 1A Sec 3:  M|W 8:30-10:45
CHEM 1A Sec 1:  M|W 11:30-12:45 (lecture)
                M|W 8:30-11:20  (lab)

Check: Does MATH 8:30-10:45 overlap with CHEM 8:30-11:20?
  → 8:30 < 11:20? YES  and  8:30 < 10:45? YES
  → OVERLAP on M and W!

Result: MATH 1A Sec 3 + CHEM 1A Sec 1 = CONFLICT
        (student must pick a different CHEM section or different MATH section)
```
