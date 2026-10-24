# De Anza College Schedule Conflict Analyzer

A full-stack application that detects and visualizes scheduling conflicts across De Anza College's 235+ academic pathways (degree/certificate programs) for the 2025-2027 academic years.

Students often discover too late that required courses in their pathway overlap in time. This project automatically identifies those conflicts using real class schedule data, empowering students to plan ahead and administrators to improve course offerings.

## Features

### Pathway Explorer (React Web App)
- Browse all 235 pathways with search, filter by credential type or academic village
- View the full 2-year course map for any pathway (Fall/Winter/Spring quarters)
- **Interactive Planner** - Select a term and courses, check conflicts in real time against live schedule data
- **AI Chat Assistant** - Ask natural-language questions about pathways, courses, and conflicts powered by Amazon Bedrock AgentCore

### Conflict Analysis Dashboard (Static HTML)
- Self-contained single-file dashboard - no server needed, just open in a browser
- Hierarchical drill-down: Division > Department > Pathway > Quarter
- Top conflict pairs visualization with bar charts
- **Schedule Comparison View** - Compare any two courses side by side on a weekly calendar grid
- Overlap matrix showing which section combinations work and which don't
- Covers 9 academic terms (Fall 2024 through Spring 2027)

### Conflict Engine
- Detects time overlaps across all section pairs for courses in a pathway quarter
- Handles multi-meeting sections (lecture + lab on different days/times)
- Distinguishes online vs in-person sections
- Produces both summary statistics and detailed per-section-pair results

### AI Chatbot (Bedrock AgentCore)
- Natural-language interface to query pathway data
- Tools: list pathways, get course plans, check conflicts, find sections
- Session-based memory for multi-turn conversations
- Deployed as a managed container on AWS

## Architecture

```
PDF Pathway Maps ──► Parser ──► DynamoDB (pathways)
                                      │
Excel/CSV Schedules ──► Loader ──► DynamoDB (class-schedule)
                                      │
                                      ▼
                              Conflict Engine
                              (Lambda Functions)
                                      │
                    ┌─────────────────┼─────────────────┐
                    ▼                 ▼                  ▼
             React Web App    HTML Dashboard    Bedrock AgentCore
             (Vite + React)   (standalone)      (AI Chatbot)
```

## Backend Services (AWS)

| Service | Role |
|---------|------|
| **DynamoDB** | 4 tables storing pathways, normalized pathways, class schedules, and precomputed conflicts |
| **Lambda** | 2 functions: `/conflicts` (raw conflict check) and `/plan` (student planner endpoint) |
| **API Gateway** | REST API with CORS, usage plans, and optional API key auth |
| **S3** | Raw data storage (schedule Excel files, pathway JSON) |
| **Bedrock AgentCore** | Managed LLM agent runtime with DynamoDB tools |
| **ECR + CodeBuild** | CI/CD for building and deploying the agent container |
| **SAM (CloudFormation)** | Infrastructure-as-code for all serverless resources |

## Quick Start

### Prerequisites
- Node.js 18+
- Python 3.10+
- AWS CLI configured with credentials (for backend features)
- SAM CLI (for deploying infrastructure)

### Clone the Repo

```bash
git clone https://github.com/<your-username>/de-anza-student-schedule.git
cd de-anza-student-schedule
```

### Run the Web App (Frontend Only)

```bash
cd web-app
npm install
npm run dev
```

Opens at `http://localhost:5173`. Browsing pathways works immediately with bundled static data. The interactive planner and chat require the AWS backend.

### Open the Dashboard

No build step needed:

```
Open analysis-dashboard/hierarchical_dashboard.html in any browser
```

Everything is self-contained - data is embedded in the HTML file.

### Deploy the Backend (AWS)

```bash
# 1. Install Python dependencies
pip install -r requirements.txt

# 2. Create DynamoDB tables
python scripts/create_tables.py

# 3. Load data into DynamoDB
python scripts/class_schedule/load.py
python scripts/pathways/load.py
python scripts/course_pairing/normalize_pathways.py

# 4. Deploy Lambda functions + API Gateway
cd infra
sam build
sam deploy --guided

# 5. Build conflict data
python scripts/pathway_conflicts/build_conflicts.py

# 6. (Optional) Deploy the chatbot agent
# See agentcore/ directory and .bedrock_agentcore.yaml
```

### Run Tests

```bash
python -m pytest
```

## Project Structure

```
├── web-app/                  # React frontend (Vite)
│   ├── src/components/       # UI components (SearchFilter, ProgramList, ChatWidget, etc.)
│   ├── src/chat/             # Dual chat system (local engine + LLM agent)
│   └── src/data/             # Pathway + conflict JSON data
├── analysis-dashboard/       # Standalone HTML conflict explorer
├── agentcore/                # Bedrock AgentCore chatbot
│   ├── agent.py              # Agent with DynamoDB tools
│   └── proxy/                # Lambda proxy for browser access
├── scripts/                  # Python ETL pipeline
│   ├── class_schedule/       # Schedule data ingestion
│   ├── pathways/             # Pathway data ingestion
│   ├── conflicts/            # Conflict detection engine + Lambda handlers
│   ├── course_pairing/       # Course code normalization
│   └── pathway_conflicts/    # Batch conflict builder + analysis
├── infra/                    # SAM template (Lambda + API Gateway)
├── tests/                    # Pytest test suite
├── docs/                     # Detailed component documentation
└── requirements.txt          # Python dependencies
```

## Documentation

Detailed docs for each component are in the `docs/` folder:

| File | Topic |
|------|-------|
| [01-project-overview.md](docs/01-project-overview.md) | Architecture and data flow |
| [02-web-app.md](docs/02-web-app.md) | React app structure and components |
| [03-analysis-dashboard.md](docs/03-analysis-dashboard.md) | HTML dashboard features |
| [04-chatbot-agent.md](docs/04-chatbot-agent.md) | Bedrock AgentCore setup and tools |
| [05-dynamodb.md](docs/05-dynamodb.md) | Table schemas and access patterns |
| [06-aws-services.md](docs/06-aws-services.md) | Lambda, API Gateway, SAM deployment |
| [07-conflict-engine.md](docs/07-conflict-engine.md) | Core scheduling conflict logic |
| [08-data-pipeline.md](docs/08-data-pipeline.md) | ETL scripts and data flow |

## Key Stats

- **235** academic pathways analyzed
- **983** pathway-quarters evaluated
- **10,173** scheduling conflict pairs detected
- **9 terms** of schedule data (Fall 2024 - Spring 2027)
- **6.41%** overall conflict rate across all course pairs

## Environment Variables

Create a `.env` file in the project root:

```env
AWS_ACCESS_KEY_ID=your_key
AWS_SECRET_ACCESS_KEY=your_secret
AWS_DEFAULT_REGION=us-west-2
S3_BUCKET=your-s3-bucket-name
CONFLICT_API_URL=https://your-api.execute-api.us-west-2.amazonaws.com/prod/conflicts
CONFLICT_API_KEY=your-api-key
```

## Acknowledgments

This project was made possible through the **Cal Poly DX Hub** initiative. Special thanks to the DX Hub team for providing the platform, mentorship, and resources that brought this project to life.

### Team

- **Ashmeet Kaur**
- **Kseniia Oblasova**
- **Rohan Gopalan**
- **John Khazraee**

## License

This project is for educational purposes as part of De Anza College schedule optimization research, developed in collaboration with Cal Poly DX Hub.
