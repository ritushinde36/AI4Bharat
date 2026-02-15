# Autonomous Market Research Agent - Design Document

## 1. SYSTEM ARCHITECTURE OVERVIEW

### High-Level Architecture

The system follows a multi-agent architecture pattern with the following layers:

```
┌─────────────────────────────────────────────────────────────┐
│                     Frontend Layer                          │
│              (React + AWS Amplify)                          │
└─────────────────────────────────────────────────────────────┘
                            ↓ HTTPS
┌─────────────────────────────────────────────────────────────┐
│                      API Layer                              │
│           (API Gateway + Lambda Functions)                  │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│                 Orchestration Layer                         │
│              (Step Functions + EventBridge)                 │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│                     Agent Layer                             │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │
│  │   Data       │  │   Analysis   │  │   Report     │     │
│  │  Collector   │  │    Agent     │  │  Generator   │     │
│  │   Agent      │  │              │  │    Agent     │     │
│  └──────────────┘  └──────────────┘  └──────────────┘     │
│  ┌──────────────┐                                          │
│  │    Alert     │                                          │
│  │   System     │                                          │
│  │    Agent     │                                          │
│  └──────────────┘                                          │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│                      Data Layer                             │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐  │
│  │   RDS    │  │    S3    │  │OpenSearch│  │ElastiCache│ │
│  │PostgreSQL│  │          │  │          │  │   Redis   │  │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘  │
└─────────────────────────────────────────────────────────────┘
```

### Multi-Agent System Design

The system employs four specialized autonomous agents:

1. **Data Collector Agent**: Scrapes competitor websites and social media
2. **Analysis Agent**: Processes data and generates insights using AI
3. **Report Generator Agent**: Creates intelligence reports
4. **Alert System Agent**: Monitors conditions and sends notifications


### AWS Services Integration

| Service | Purpose | Usage Pattern |
|---------|---------|---------------|
| Amplify | Frontend hosting & CI/CD | Hosts React app, auto-deploys on git push |
| API Gateway | REST API endpoints | Routes requests to Lambda functions |
| Lambda | Serverless compute | Handles API logic, lightweight processing |
| Step Functions | Workflow orchestration | Coordinates multi-agent workflows |
| ECS Fargate | Container-based scraping | Runs headless browsers for dynamic sites |
| RDS PostgreSQL | Relational database | Stores structured data (users, competitors, prices) |
| S3 | Object storage | Stores raw HTML, images, reports |
| OpenSearch | Search & analytics | Full-text search, log analytics |
| ElastiCache Redis | Caching layer | Caches API responses, session data |
| Bedrock (Claude) | AI analysis | Generates insights and recommendations |
| Comprehend | NLP & sentiment | Analyzes text sentiment |
| SNS | Pub/sub messaging | Distributes alerts to multiple channels |
| SES | Email delivery | Sends alert emails and reports |
| EventBridge | Event scheduling | Triggers scraping jobs, report generation |
| CloudWatch | Monitoring & logging | Tracks metrics, stores logs |
| Secrets Manager | Secrets storage | Stores API keys, DB credentials |
| IAM | Access control | Manages permissions across services |

### Data Flow

1. **User Interaction Flow**:
   - User → Amplify (React) → API Gateway → Lambda → RDS/S3
   - Response cached in ElastiCache for subsequent requests

2. **Scraping Flow**:
   - EventBridge (cron) → Step Functions → Data Collector Agent (ECS Fargate)
   - Scraped data → S3 (raw HTML) + RDS (structured data)
   - Data indexed in OpenSearch for search

3. **Analysis Flow**:
   - Step Functions → Analysis Agent (Lambda + Bedrock)
   - Reads from RDS → Generates insights → Stores in RDS

4. **Alert Flow**:
   - Alert Agent (Lambda) → Evaluates conditions → SNS → SES/SMS
   - Alert history stored in RDS

5. **Report Flow**:
   - EventBridge (weekly) → Report Generator Agent (Lambda + Bedrock)
   - Generates PDF → S3 → SES (email to user)

---

## 2. COMPONENT DESIGN

### A. Frontend Layer

**Technology Stack**: React 18 + TypeScript + Tailwind CSS + AWS Amplify

**Key Pages**:
1. Landing Page
2. Setup - Add Competitors
3. Setup - Configure Alerts
4. Setup - Choose Reports
5. Main Dashboard
6. Competitors View
7. Alerts Page
8. Reports Library