# AWS Services & Infrastructure

## What Is This? (Plain English)

The backend runs entirely on AWS (Amazon Web Services) using "serverless" architecture. This means we don't maintain any servers — AWS runs our code on demand and charges only when someone actually uses it. When no one's using the app, it costs almost nothing.

Everything is defined in a **SAM template** (`infra/template.yaml`) — a configuration file that tells AWS "create these resources for me." Think of it like a recipe that AWS follows to set up our entire backend automatically.

## Services We Use

| Service | Plain English | Technical Description |
|---------|--------------|----------------------|
| **DynamoDB** | A cloud database (like spreadsheets that scale) | NoSQL key-value store, on-demand billing |
| **Lambda** | Small programs that run on demand (no server) | Serverless compute, pay-per-invocation |
| **API Gateway** | The "front door" that receives web requests | REST API with routes, CORS, throttling |
| **S3** | Cloud file storage (like Google Drive for data) | Object storage for raw Excel/CSV/JSON files |
| **Bedrock AgentCore** | AI brain that answers questions | Managed LLM agent with tool-use capability |
| **ECR** | Storage for the chatbot's container | Docker image registry |
| **CodeBuild** | Automatic builder/deployer for the chatbot | CI/CD pipeline |
| **IAM** | Permission system (who can access what) | Roles, policies, trust relationships |

## The Two Lambda Functions

### Lambda 1: `/conflicts` (The Raw Checker)

**What it does**: You give it a list of class sections with their times, it tells you which ones overlap.

**Analogy**: Like giving someone two schedules printed on paper and asking "do any of these overlap?"

**Access**: Requires an API key (only our backend scripts use this)

**Example**:
```
You send: "Here are 5 sections with their days/times"
It returns: "Sections A and C overlap on Monday 9-10am"
```

### Lambda 2: `/plan` (The Student Planner)

**What it does**: You give it a term and course names, it looks up ALL available sections from the database, then checks every combination for conflicts.

**Analogy**: Like telling an advisor "I want to take these 3 courses next quarter" and they check the entire schedule to tell you which section combos work.

**Access**: Public (the web app calls this directly)

**Example**:
```
You send: "Fall 2025, MATH 1A + CHEM 1A + ENGL C1000"
It returns: "66% of course pairs have at least one overlap. 
             MATH 1A vs CHEM 1A: 4 section combos clash, 11 combos work.
             CHEM 1A vs ENGL C1000: All section combos work!"
```

### Lambda 3: `/chat` (The Chatbot Proxy)

**What it does**: Passes your chat message to the AI agent and returns the answer.

**Why it exists**: Your browser can't talk directly to Bedrock AgentCore (needs AWS credentials). This Lambda acts as a secure middleman.

## API Gateway — The Front Door

API Gateway is what connects the internet to our Lambda functions. It:
- Gives us a URL (like `https://xyz.execute-api.us-west-2.amazonaws.com/prod/plan`)
- Adds CORS headers (so browsers are allowed to call it)
- Rate-limits requests (so nobody can spam our API)
- Optionally requires an API key (for the `/conflicts` endpoint)

**Rate limits**: 10 requests/second, burst up to 20, max 1000/day for the keyed endpoint.

## How Everything Connects

```
Student's Browser
    │
    ├── GET pathway data ──→ (bundled in the app, no API call)
    │
    ├── POST /plan ──→ API Gateway ──→ Plan Lambda ──→ DynamoDB
    │                                       │
    │                                       └── conflict_engine.py
    │
    └── POST /chat ──→ API Gateway ──→ Chat Lambda ──→ Bedrock AgentCore
                                                            │
                                                            └── DynamoDB
```

## Deploying (Making It Live)

The SAM template handles everything. From the `infra/` folder:

```bash
sam build          # Packages the code
sam deploy --guided  # First time: asks you questions about naming/region
sam deploy         # After that: just re-deploys
```

This creates:
- 2 Lambda functions
- 1 API Gateway with 2 routes
- IAM roles with proper permissions
- Usage plan + API key

## Environment Variables

Each Lambda gets these from the SAM template (you don't set them manually):
- `SCHEDULE_TABLE` — Which DynamoDB table has the class schedules
- `PATHWAYS_TABLE` — Which table has pathway data
- `CONFLICTS_TABLE` — Which table has precomputed results
- `CONFLICT_API_URL` — URL of the conflicts Lambda (used internally)

## Cost

With serverless, you pay only for actual usage:
- **DynamoDB**: ~$0 for low traffic (on-demand pricing)
- **Lambda**: First 1M requests/month are free
- **API Gateway**: First 1M requests/month are free
- **Bedrock AgentCore**: Pay per AI invocation (~$0.01-0.05 per chat message)

For a student project, this typically stays within AWS free tier.
