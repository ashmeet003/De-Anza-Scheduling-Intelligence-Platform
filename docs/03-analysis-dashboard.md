# Analysis Dashboard

## What Is This? (Plain English)

Think of it as a "conflict heatmap" for administrators. Instead of checking one pathway at a time, this dashboard shows the **entire college's scheduling conflicts** in one place. You can drill down from the big picture (which divisions have the most problems?) all the way to individual course-pair overlaps on a weekly calendar.

It's a single HTML file — no server, no installation, no internet needed. Just open it in Chrome/Firefox and start exploring.

You can also access it from the React web app by clicking the gold "Conflict Dashboard" button in the filter bar.

## Where Is It?

- **Standalone**: `analysis-dashboard/hierarchical_dashboard.html`
- **From web app**: Click "Conflict Dashboard" in the filter row (opens in new tab at `/dashboard.html`)

## How Everything Is Embedded

Normally a website loads data from a server. This dashboard skips that entirely — all the data is baked directly into the HTML file as JavaScript variables:

- `CONFLICT_DATA` — Statistics for all 57 subjects and 9 divisions
- `PATHWAY_DATA` — 29 sample pathways with full course plans
- `SCHEDULE_MULTITERM` — Actual class schedules for 9 terms (every section's days/times/rooms/instructors)

This means the file is large (~840KB) but works anywhere, instantly.

## Features

### 1. Hierarchical Conflict Explorer

A 5-level drill-down. Click to go deeper, use breadcrumbs to go back:

| Level | What You See | Example |
|-------|-------------|---------|
| **Overview** | Total stats + all academic divisions | "Language Arts has 8,383 conflict appearances" |
| **Division** | Departments within that division | "COMM has 7,912, ESL has 327" |
| **Department** | Which other departments it clashes with | "COMM vs MATH: 819 conflicts" |
| **Pathway** | A specific degree's course plan by quarter | "Biology for Transfer — Year 1 Fall" |
| **Quarter** | Exact courses + detected conflicts for that quarter | "BIOL 6A vs MATH 1A may overlap" |

### 2. Schedule Comparison View

This is the interactive calendar tool. It answers: "If I pick Course A and Course B, can I find section times that don't clash?"

**How to use it:**
1. Click the gold "Schedule Comparison View" card on the Overview page
2. Pick a **term** (e.g., Winter 2026)
3. Choose your **anchor course** (the one you're definitely taking)
4. Choose a **comparison course** (the other one you want)
5. See the results:
   - **All Sections mode**: Shows a matrix of every section pair — green = compatible, red = clash
   - **Single Section mode**: Pick your specific section, see which comparison sections still work

The weekly calendar shows blocks on M-F from 7am-10pm with color coding:
- **Red blocks** = Your anchor course
- **Blue blocks** = Available comparison sections
- **Red-highlighted** = Sections that overlap with your anchor

### 3. Key Numbers

| Metric | Value |
|--------|-------|
| Pathways analyzed | 235 |
| Quarters evaluated | 983 |
| Conflict pairs found | 10,173 |
| Overall conflict rate | 6.41% |
| Worst offender | COMM 1 vs COMM 10 (affects 98 pathways) |
| Terms covered | Fall 2024 through Spring 2027 |

## Design

- Uses De Anza brand colors (red and gold)
- Responsive — works on desktop and tablet
- Pure vanilla JavaScript (no React, no frameworks)
- ~2,500 lines of clean HTML/CSS/JS
