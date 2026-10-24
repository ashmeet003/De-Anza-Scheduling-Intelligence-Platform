# Project Overview

## What Does This Project Do? (Plain English)

Imagine you're a De Anza College student picking classes for next quarter. You look at your pathway (your degree plan) and see you need MATH 1A and CHEM 1A. You register for both — but surprise, they're at the same time on Monday and Wednesday. Now you have to drop one and your graduation gets delayed.

This project **automatically finds those time clashes** for every single pathway at De Anza (235 programs!) across 9 quarters. It saves students from nasty surprises and helps administrators see where the biggest scheduling problems are.

## How It Works (Big Picture)

Think of it like a pipeline with 3 stages:

1. **Collect the data** — We grab the actual class schedules (what time each section meets) and pathway maps (what courses each degree requires, and when)

2. **Compare everything** — For each pathway, for each quarter, we check: "If a student takes these 3 courses together, do any of the available sections clash?" We do this for all 235 pathways x 6 quarters = over 1,400 comparisons

3. **Show the results** — A React web app for students to browse and check conflicts interactively, and an HTML dashboard for administrators to see the big picture

## Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────────┐
│                         DATA SOURCES                                 │
│  PDF Pathway Maps  →  Parser  →  deanza_pathways.json               │
│  Excel/CSV Schedules → Loader → DynamoDB (class schedule table)     │
└──────────────────────────────────┬──────────────────────────────────┘
                                   │
                                   ▼
┌─────────────────────────────────────────────────────────────────────┐
│                       AWS BACKEND                                     │
│                                                                       │
│  DynamoDB (4 tables)            Lambda Functions (2)                  │
│  - pathways                     - /conflicts (raw check)             │
│  - class schedules              - /plan (student planner)            │
│  - normalized pathways                                               │
│  - precomputed conflicts        Bedrock AgentCore (AI chatbot)       │
│                                   └─ Proxy Lambda                    │
└──────────────────────────────────┬──────────────────────────────────┘
                                   │
                                   ▼
┌─────────────────────────────────────────────────────────────────────┐
│                       USER-FACING APPS                                │
│                                                                       │
│  React Web App                  Analysis Dashboard (HTML)            │
│  - Browse pathways              - Admin conflict overview            │
│  - Interactive planner          - Department drill-down              │
│  - AI chat assistant            - Schedule comparison calendar       │
│                                                                       │
│  (Dashboard is also accessible from the web app via a link)          │
└─────────────────────────────────────────────────────────────────────┘
```

## Key Components

| Component | Location | What It Does |
|-----------|----------|--------------|
| Web App | `web-app/` | The website students use to browse pathways and check conflicts |
| Dashboard | `analysis-dashboard/` | A standalone page for admins to explore all conflict data |
| Agent | `agentcore/` | An AI chatbot that can answer questions about schedules |
| Scripts | `scripts/` | Python programs that load data, normalize it, and find conflicts |
| Infrastructure | `infra/` | AWS deployment configuration (what cloud resources to create) |
| Tests | `tests/` | Automated checks that the conflict logic works correctly |
| Docs | `docs/` | You're reading this! |

## How to Run

| What you want to do | Command |
|---------------------|---------|
| Start the web app | `cd web-app && npm run dev` |
| Open the dashboard | Open `analysis-dashboard/hierarchical_dashboard.html` in your browser |
| Run automated tests | `python -m pytest` from project root |
| Load schedule data into AWS | `python scripts/class_schedule/load.py` |
| Compute all conflicts | `python scripts/pathway_conflicts/build_conflicts.py` |
| Deploy to AWS | `cd infra && sam build && sam deploy` |
