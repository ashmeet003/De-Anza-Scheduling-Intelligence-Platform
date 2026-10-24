# Resume & LinkedIn Content

---

## Resume Bullet Points

Pick 2-4 that best match the role you're applying for.

### Software Engineering / Full-Stack Focus

- Built a full-stack scheduling conflict detection system for De Anza College (real client project via Cal Poly DX Hub), analyzing 235 academic pathways across 9 terms to identify 10,000+ course time conflicts, reducing student scheduling errors and improving on-time graduation planning
- Developed a React web application with an interactive schedule planner that queries a serverless AWS backend (Lambda, API Gateway, DynamoDB) in real time, enabling students to verify course compatibility before registration
- Designed and implemented a Python conflict detection engine that compares all section-pair combinations across 983 pathway-quarters, processing 158,000+ course pairs with time-overlap analysis and producing actionable conflict reports
- Integrated an AI-powered chatbot using Amazon Bedrock AgentCore with custom DynamoDB tools, allowing students to ask natural-language questions about pathways and scheduling conflicts

### Cloud / AWS Focus

- Architected a serverless backend on AWS (Lambda, API Gateway, DynamoDB, S3, Bedrock AgentCore) for a college schedule optimization platform serving 235 academic programs, deployed via SAM/CloudFormation with CI/CD pipeline
- Built a data pipeline ingesting PDF pathway maps and Excel class schedules into DynamoDB, normalizing course codes across systems and computing conflicts via Lambda functions with API key-secured endpoints
- Deployed an LLM-based conversational agent on Amazon Bedrock AgentCore with IAM-scoped DynamoDB read access, proxied through Lambda for secure browser-to-agent communication

### Data Engineering / Analytics Focus

- Engineered an ETL pipeline processing 235 PDF pathway maps and 9 terms of class schedule data (10,000+ sections), normalizing course identifiers and detecting scheduling conflicts across 158,000+ course pairs with a 6.4% conflict rate
- Created a self-contained interactive analytics dashboard visualizing scheduling conflicts across 57 departments and 9 academic divisions, featuring hierarchical drill-down, weekly calendar comparison, and section-pair overlap matrices

### General / Shorter Versions

- Developed a schedule conflict analyzer for De Anza College (real-world client) that detects time overlaps across 235 degree pathways, built with React, Python, AWS Lambda, and DynamoDB
- Led backend development for a college scheduling platform processing 10,000+ conflict pairs, using serverless AWS architecture and an AI chatbot for natural-language course queries
- Built an end-to-end academic scheduling tool for De Anza College — from PDF parsing and data normalization through conflict detection to a React frontend with AI chat — identifying conflicts affecting 98 pathways in the top course pair alone

---

## LinkedIn Project Description

### Title
**De Anza College Schedule Conflict Analyzer**

### Tagline
Full-stack scheduling intelligence platform that detects time conflicts across 235 academic pathways — a real-world project for De Anza College via Cal Poly DX Hub.

### Description (Post or Project Section)

Excited to share a project I built with my team for De Anza College through the Cal Poly DX Hub program!

**The Problem:** Students often discover too late that required courses in their degree pathway have overlapping class times — leading to delayed graduation and frustrating schedule changes.

**What We Built:** A complete scheduling intelligence platform that automatically detects every time conflict across all 235 academic programs for the 2025-2027 school years.

**Key Results:**
- Analyzed 983 pathway-quarters, evaluating 158,000+ course pairs
- Identified 10,173 scheduling conflicts (6.4% overall conflict rate)
- Found that COMM 1 vs COMM 10 alone affects 98 pathways — a systemically fixable issue
- Gave students a real-time planner to check conflicts BEFORE registration

**What I Built:**
- React web app with interactive conflict checker and pathway explorer
- Python conflict detection engine comparing all section time/day combinations
- Serverless AWS backend (Lambda, API Gateway, DynamoDB) for real-time queries
- AI chatbot (Amazon Bedrock AgentCore) that answers schedule questions in plain English
- Self-contained admin dashboard with calendar visualization and department-level analytics
- ETL pipeline parsing PDFs and Excel schedules into normalized, queryable data

**Tech Stack:** React, Python, AWS (Lambda, DynamoDB, API Gateway, S3, Bedrock AgentCore, SAM/CloudFormation), Vite

**Team:** Kseniia Oblasova, Rohan Gopalan, John Khazraee, and myself
**Made possible by:** Cal Poly DX Hub

#AWS #React #Python #Serverless #CloudComputing #EdTech #DataEngineering #AI #CalPoly #DeAnzaCollege

---

## Skills to Tag (LinkedIn / Resume)

- Python, JavaScript, React, HTML/CSS
- AWS Lambda, DynamoDB, API Gateway, S3, IAM, SAM/CloudFormation
- Amazon Bedrock, AgentCore, LLM Integration
- Serverless Architecture, REST APIs, CORS
- ETL Pipelines, Data Normalization, PDF Parsing
- Vite, Node.js, Git
- Data Visualization, Analytics Dashboards
- Agile Development, Cross-functional Collaboration


---

## LinkedIn Post (Ready to Copy-Paste)

---

De Anza College serves 20,000+ students across dozens of academic departments — each building class schedules independently. Coordinating across departments to ensure required courses don't overlap in time is a massive logistical challenge at that scale. Conflicts get caught late, require back-and-forth between departments, and ultimately slow down students' progress toward graduation.

Through the Cal Poly DX Hub program, our team built a centralized scheduling intelligence platform that makes these conflicts visible — instantly and at scale.

What we delivered:

I built an interactive admin dashboard that gives a bird's-eye view of conflicts across all 235 programs and 57 departments, with drill-downs by division, course pair, and term — plus a weekly calendar that lets you compare any two courses' section times side by side. Departments can now see systemic scheduling issues and coordinate proactively.

We also built a student-facing web app with a real-time conflict checker and an AI chatbot (Amazon Bedrock) for natural-language schedule queries, backed by a serverless AWS architecture (Lambda, DynamoDB, API Gateway) and a Python conflict detection engine.

The numbers:
- 235 academic pathways analyzed
- 158,000+ course-pair combinations evaluated
- 10,173 scheduling conflicts identified
- 6.4% overall conflict rate
- Top conflict (COMM 1 vs COMM 10) affects 98 pathways alone
- 9 academic terms covered (Fall 2024 – Spring 2027)

The original request was administrative — give departments visibility into cross-program conflicts. The platform we delivered serves both sides: administrators can coordinate scheduling decisions proactively, and students can verify their course plan before committing to registration.

Our team placed 2nd out of 16 teams in the program — a cohort of 100 students selected from over 1,200 applicants.

Takeaway: Coordination problems at scale aren't solved by more meetings — they're solved by making the right data visible to the right people at the right time.

Team: Kseniia Oblasova, Rohan Gopalan, John Khazraee
Made possible by Cal Poly DX Hub.

Tech: React, Python, AWS (Lambda, DynamoDB, API Gateway, Bedrock AgentCore, S3, SAM), Vite

#AWS #CloudComputing #React #Python #Serverless #EdTech #AI #CalPoly #DeAnzaCollege
