# Chatbot & AI Agent

## What Is This? (Plain English)

The chatbot is like having a personal De Anza advisor available 24/7. You type a question in plain English — "What pathways include Chemistry?" or "Does PHYS 4A conflict with MATH 1C in Winter 2026?" — and it gives you an answer by actually looking up the real schedule data.

It's not just a simple FAQ bot. It uses a large language model (like ChatGPT) combined with **tools** that let it query our actual database. So it can give you real, accurate answers based on current schedule data.

## How It Works (Simple Version)

```
You type: "Does CHEM 1A conflict with MATH 1A in Fall 2025?"
    ↓
Your browser sends this to a Lambda function (a small program on AWS)
    ↓
Lambda forwards it to Bedrock AgentCore (Amazon's AI agent service)
    ↓
The AI reads your question and thinks: "I need to look up sections for both courses"
    ↓
It calls its tools: find_course_sections("CHEM 1A", "202622")
                     find_course_sections("MATH 1A", "202622")
    ↓
Tools query DynamoDB (the database) and return actual section times
    ↓
The AI compares the times and writes a natural-language answer:
    "Yes, there are overlaps on MW between CHEM 1A Sec 1 (8:30-11:20am)
     and MATH 1A Sec 3 (8:30-10:45am). However, you have 5 other MATH 1A
     sections that don't conflict."
    ↓
Answer appears in your chat widget
```

## Components

### 1. The Agent (`agentcore/agent.py`)

This is the brain. It's a Python program that:
- Has a **system prompt** telling it "You are a De Anza College schedule advisor"
- Has **tools** (functions it can call to get data):

| Tool | What It Does | Analogy |
|------|-------------|---------|
| `list_pathways` | Lists all programs, optionally filtered | Like browsing the catalog |
| `get_pathway_plan` | Gets the full 2-year course plan | Like looking at a degree worksheet |
| `get_pathway_conflicts` | Gets precomputed conflict stats | Like asking "how bad is this pathway?" |
| `find_course_sections` | Finds all sections of a course in a term | Like looking at the schedule of classes |
| `find_sections_without_room` | Finds online/TBA sections | Like filtering for online-only classes |

The AI decides which tools to use based on your question. If you ask a complex question, it might call multiple tools and combine the results.

### 2. The Proxy Lambda (`agentcore/proxy/lambda_function.py`)

**Why it exists**: Your browser can't talk directly to Bedrock AgentCore because it requires AWS credentials (like a password). So we put a Lambda function in between that has the credentials and acts as a middleman.

**What it does**:
1. Receives your message from the browser
2. Adds the proper AWS authentication
3. Forwards to AgentCore
4. Cleans up the response (removes internal "thinking" text)
5. Sends the clean answer back to your browser

### 3. Configuration (`.bedrock_agentcore.yaml`)

This file tells AWS how to run the agent:
- What container image to use (the agent is packaged like an app in a box)
- How much memory it gets
- Whether it's accessible from the internet (yes)
- What type of memory it has (session-only — it forgets between conversations)

### 4. IAM Policies (the "permissions" files)

AWS is very strict about what can access what. These JSON files are like permission slips:

- **`invoke-policy.json`** — "The proxy Lambda is allowed to talk to AgentCore"
- **`trust-policy.json`** — "Lambda service is allowed to use this role"
- **`dynamodb-read-policy.json`** — "The agent is allowed to read from our 3 database tables"

## The Two Chat Modes in the Web App

The web app has TWO chat backends:

| Mode | File | How It Works | When It's Used |
|------|------|-------------|----------------|
| Simple | `chatEngine.js` | Pattern matching — no AI, just rules | Quick common questions, works offline |
| Smart | `chatAgent.js` | Calls the AI agent via Lambda | Complex questions, needs AWS backend |

Currently, the ChatWidget uses the smart mode (`chatAgent.js`).

## Example Conversations

**Simple question:**
> "What pathways are in Health and Life Sciences?"
> → Agent calls `list_pathways(village="Health and Life Sciences")`, formats the list

**Conflict check:**
> "Can I take PHYS 4A and CHEM 1A together in Winter 2026?"
> → Agent calls `find_course_sections` for both, compares times, reports which sections work

**Follow-up (multi-turn):**
> "What about Spring 2026 instead?"
> → Agent remembers the context (same courses), checks the new term
