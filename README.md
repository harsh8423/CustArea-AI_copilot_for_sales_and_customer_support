> #⚠️ IMPORTANT NOTICE
>
> **The original repository for this project is currently private.**  
> If you need access to the source repository or additional implementation details, **please contact the developer directly**.
>
> **The README in this repository fully explains the project architecture, design decisions, and overall working of the system.** It provides a comprehensive overview of how the platform is structured and how the components interact.
>
> For any further information, collaboration inquiries, or access requests, **reach out to the developer.**



# CustArea: AI-Native Customer Relationship Platform

> **A full-stack, multi-tenant CRM platform combining autonomous AI agents, omni-channel messaging, workflow automation, outreach campaigns, and real-time voice — engineered for modern customer-facing teams.**

---

## 1. Executive Overview

**CustArea** is a production-grade, AI-native Customer Relationship Management platform built for businesses that need intelligent, scalable customer engagement. Unlike traditional CRMs that bolt AI on as an afterthought, CustArea is designed from the ground up around AI-first principles — with autonomous agents, human-in-the-loop escalation, copilot assistance, and credit-tracked AI usage across every channel.

### Core Pillars

| Pillar | Description |
|--------|-------------|
| 🤖 **AI Agent** | Autonomous agents with RAG, function calling, guardrails, and smart escalation |
| 🧑‍✈️ **AI Copilot** | In-inbox assistant for human agents — drafts replies, summarizes threads, retrieves context |
| 📣 **Outreach Campaigns** | AI-personalized cold email campaigns with daily limits, analytics, and credit control |
| 📱 **Omni-Channel Inbox** | Unified WhatsApp, Email, Phone, and Live Chat with cross-channel identity resolution |
| ⚡ **Workflow Automation** | Visual no-code/low-code workflow builder with event-driven, graph-based execution |
| 🎙️ **Voice Agents** | Configurable real-time AI phone agents with STT/TTS and CRM action tools |
| 📊 **Analytics & Reporting** | Per-user, time-range, and per-channel analytics with CSV/JSON export |
| 💳 **Subscription & Credits** | Plan-based tiered access with AI credit tracking across email, phone, and campaigns |

---

## 2. System Architecture

### 2.1 High-Level Architecture

```mermaid
flowchart TB
    subgraph External["External Channels"]
        WA[WhatsApp\nTwilio]
        Email[Email\nGmail / Outlook / SES]
        Phone[Phone\nTwilio Voice]
        Website[Website\nChat Widget]
    end

    subgraph CustArea["CustArea Platform"]
        subgraph Frontend["Frontend Layer"]
            Client[Next.js Client\nDashboard & UI]
            Widget[Chat Widget\nEmbeddable JS]
            Landing[Landing Page\nPublic Site]
        end

        subgraph Backend["Backend Services"]
            API[Express.js API\nPort 8000]
            WF[Workflow Service\nPort 8001]
            AI_SVC[AI Assistant Service\nInternal]
        end

        subgraph Workers["Background Workers"]
            WAWorker[WhatsApp Worker]
            EmailWorker[Email Inbound/Outbound]
            AIWorker[AI Incoming Worker]
            EventWorker[Event Worker]
            SchedulerWorker[Scheduler Worker]
            CampaignWorker[Campaign Email Worker]
            NotifWorker[Notification Worker]
        end

        subgraph Data["Data Layer"]
            PG[(PostgreSQL\nRelational Data)]
            MongoDB[(MongoDB\nAI Agent Config)]
            Redis[(Redis Streams\nMessage Queues)]
        end

        subgraph AI["AI Layer"]
            OpenAI[OpenAI GPT-4o\nGPT-4o-mini]
            Groq[Groq LLaMA]
            VectorDB[Vector Embeddings\nSemantic Search]
        end
    end

    WA --> API
    Email --> API
    Phone --> API
    Website --> Widget
    Widget --> API

    Client --> API
    Client --> WF

    API <--> Redis
    WF <--> Redis

    Redis --> WAWorker
    Redis --> EmailWorker
    Redis --> AIWorker
    Redis --> EventWorker
    Redis --> CampaignWorker

    API <--> PG
    WF <--> PG
    API <--> MongoDB

    AIWorker --> OpenAI
    AIWorker --> Groq
    AIWorker --> VectorDB
```

### 2.2 Service Decomposition

| Service | Technology | Port | Responsibility |
|---------|------------|------|----------------|
| **Backend API** | Node.js + Express | 8000 | Core API, webhooks, WebSocket handlers, credit tracking |
| **Workflow Service** | Node.js + Express | 8001 | Workflow execution, scheduling, event-driven triggers |
| **AI Assistant Service** | Node.js + Express | 8001 | Natural-language CRM command assistant (Email + Telegram channels) |
| **Frontend Client** | Next.js 15 + TypeScript | 3000 | Dashboard, workflow builder, settings, analytics |
| **Chat Widget** | Vite + TypeScript | N/A | Embeddable website chat widget |
| **Landing Page** | Static/SSG | N/A | Public marketing site |
| **PostgreSQL** | PostgreSQL 15+ | 5432 | Core relational data, all tenant data |
| **MongoDB** | MongoDB 6+ | 27017 | AI agent configuration, knowledge chunks |
| **Redis** | Redis 7+ | 6379 | Message queuing, pub/sub, real-time events |

---

## 3. Database Architecture

### 3.1 Entity Relationship Diagram

```mermaid
erDiagram
    TENANTS ||--o{ USERS : has
    TENANTS ||--o{ CONTACTS : owns
    TENANTS ||--o{ PIPELINES : manages
    TENANTS ||--o{ LEADS : tracks
    TENANTS ||--o{ CONVERSATIONS : maintains
    TENANTS ||--o{ TICKETS : handles
    TENANTS ||--o{ WORKFLOWS : automates
    TENANTS ||--o{ OUTREACH_CAMPAIGNS : runs
    TENANTS ||--o{ CONTACT_GROUPS : segments
    TENANTS ||--o{ CREDIT_LEDGER : tracks_usage

    CONTACTS ||--o{ LEADS : becomes
    CONTACTS ||--o{ ACCOUNTS : converts_to
    CONTACTS ||--o{ CONVERSATIONS : participates
    CONTACTS ||--o{ CONTACT_GROUP_MEMBERSHIPS : belongs_to
    CONTACT_GROUPS ||--o{ CONTACT_GROUP_MEMBERSHIPS : has
    CONTACT_GROUPS ||--o{ OUTREACH_CAMPAIGNS : targets

    PIPELINES ||--|{ PIPELINE_STAGES : contains
    PIPELINE_STAGES ||--o{ LEADS : holds

    LEADS }o--|| USERS : assigned_to
    CONVERSATIONS ||--|{ MESSAGES : contains
    CONVERSATIONS ||--o{ ESCALATIONS : triggers

    WORKFLOWS ||--|{ WORKFLOW_VERSIONS : versioned_by
    WORKFLOWS ||--o{ WORKFLOW_RUNS : executes
    WORKFLOW_RUNS ||--|{ WORKFLOW_RUN_NODES : tracks

    OUTREACH_CAMPAIGNS ||--|{ CAMPAIGN_CONTACTS : enrolls
    OUTREACH_CAMPAIGNS ||--|| CAMPAIGN_ANALYTICS : aggregates

    TENANTS {
        uuid id PK
        string name
        string status
        string plan
        boolean ai_enabled
        string ai_mode
    }

    CONTACTS {
        uuid id PK
        uuid tenant_id FK
        string name
        string email
        string phone
        string company_name
        string source
        jsonb metadata
    }

    SUBSCRIPTION_PLANS {
        uuid id PK
        string plan_key
        string name
        decimal price_monthly
        int credits_included
        boolean is_trial
        boolean allows_addon_credits
    }

    CREDIT_LEDGER {
        uuid id PK
        uuid tenant_id FK
        string transaction_type
        int amount
        string feature
        text description
    }

    OUTREACH_CAMPAIGNS {
        uuid id PK
        uuid tenant_id FK
        string name
        string campaign_objective
        string reply_handling
        int daily_send_limit
        int max_contacts_limit
        string status
    }

    ESCALATIONS {
        uuid id PK
        uuid tenant_id FK
        uuid conversation_id FK
        string reason
        string priority
        string department
        string status
    }

    LEADS {
        uuid id PK
        uuid tenant_id FK
        uuid contact_id FK
        uuid pipeline_id FK
        uuid stage_id FK
        uuid owner_id FK
        string status
        int score
    }

    CONVERSATIONS {
        uuid id PK
        uuid tenant_id FK
        uuid contact_id FK
        string channel
        string status
        string ai_mode
    }

    TICKETS {
        uuid id PK
        uuid tenant_id FK
        uuid contact_id FK
        string subject
        string status
        string priority
    }

    WORKFLOWS {
        uuid id PK
        uuid tenant_id FK
        string name
        boolean is_active
    }
```

### 3.2 Multi-Tenancy Model

CustArea implements a **shared database, shared schema** multi-tenancy model with strict application-level isolation:

- Every table has a `tenant_id` foreign key — all queries enforce this filter
- Per-tenant credentials for WhatsApp, Email (SES/Gmail/Outlook), and Twilio Voice
- Tenant-specific subscription plans with independent credit ledgers
- Role-Based Access Control (RBAC) scoped per tenant

---

## 4. Core Modules

### 4.1 Omni-Channel Inbox

CustArea provides a unified inbox where all customer communications — regardless of channel — are threaded into conversations.

```mermaid
flowchart LR
    subgraph Inbound["Inbound Messages"]
        WA_IN[WhatsApp Webhook]
        EMAIL_IN[Email Inbound\nGmail/Outlook/SES]
        PHONE_IN[Phone Call\nTwilio Voice]
        WIDGET_IN[Chat Widget]
    end

    subgraph Processing["Message Processing"]
        RESOLVER[Contact Resolver\nCross-channel ID]
        CONV[Conversation Manager\nThread continuity]
        QUEUE[Redis Queue]
    end

    subgraph Routes["Routing Decision"]
        WF_CHECK{Has Active\nWorkflow?}
        AI_CHECK{AI Agent\nEnabled?}
    end

    subgraph Actions["Action Handlers"]
        WF_ENGINE[Workflow Engine]
        AI_AGENT[AI Agent Service]
        HUMAN[Human Agent Queue]
    end

    WA_IN --> RESOLVER
    EMAIL_IN --> RESOLVER
    PHONE_IN --> RESOLVER
    WIDGET_IN --> RESOLVER

    RESOLVER --> CONV
    CONV --> QUEUE
    QUEUE --> WF_CHECK

    WF_CHECK -->|Yes| WF_ENGINE
    WF_CHECK -->|No| AI_CHECK
    AI_CHECK -->|Yes| AI_AGENT
    AI_CHECK -->|No| HUMAN
```

**Channels Supported:**

| Channel | Provider | Features |
|---------|----------|----------|
| **WhatsApp Business** | Twilio API | Message status tracking, media, templates |
| **Email** | AWS SES, Gmail OAuth, Outlook OAuth | Thread continuity, reply detection, multi-sender rotation |
| **Phone/Voice** | Twilio Voice + WebSocket | Real-time AI conversation, call recording, STT/TTS |
| **Live Chat Widget** | Embeddable JS | website embed, real-time, contact capture |

**Email Provider Support (Multi-Provider Architecture):**
- **AWS SES** — transactional and campaign email sending
- **Gmail** — OAuth-connected Gmail accounts as senders
- **Microsoft Outlook** — OAuth-connected Outlook accounts as senders
- Automatic sender selection with SES priority → Gmail/Outlook fallback
- Credit-aware email sending across all providers

---

### 4.2 AI Agent System

The AI Agent module provides intelligent, configurable conversational AI with enterprise-grade controls across all inbound channels.

```mermaid
flowchart TB
    subgraph Input["Incoming Message"]
        MSG[User Message]
    end

    subgraph Safety["Safety Layer"]
        GUARD_IN[Input Guardrails\nKeyword/Regex/AI filters]
    end

    subgraph Intelligence["Intelligence Layer"]
        ATTR[Attribute Detection\nSentiment, Intent, Urgency]
        ESC[Escalation Rules\nCondition matching]
        KB[Knowledge Base\nVector Search RAG]
        CONTEXT[Context Builder\nContact + History]
        CONFIDENCE[Confidence Gate\nQuality control]
    end

    subgraph Generation["Response Generation"]
        PROMPT[System Prompt Builder\nGuidance + Guardrails]
        LLM[LLM Provider\nOpenAI / Groq]
        TOOLS[Function Calling\nCRM Actions]
    end

    subgraph Output["Output"]
        GUARD_OUT[Output Guardrails]
        RESPONSE[Final Response]
        ESCALATE[Escalate to Human]
    end

    MSG --> GUARD_IN
    GUARD_IN -->|Pass| ATTR
    GUARD_IN -->|Block| RESPONSE

    ATTR --> ESC
    ESC -->|Match| ESCALATE
    ESC -->|No Match| KB

    KB --> CONTEXT
    CONTEXT --> CONFIDENCE
    CONFIDENCE --> PROMPT
    PROMPT --> LLM
    LLM --> TOOLS
    TOOLS --> GUARD_OUT
    GUARD_OUT --> RESPONSE
```

**AI Agent Features:**

| Feature | Description |
|---------|-------------|
| **Multi-LLM Support** | OpenAI GPT-4o / GPT-4o-mini, Groq LLaMA 3.1 — selectable per tenant |
| **Knowledge Base RAG** | URL, PDF, plain text ingestion with vector embeddings + semantic search |
| **Guidance System** | Configurable tone, communication style, response persona |
| **Input Guardrails** | Keyword, regex, and AI-based content filtering before processing |
| **Output Guardrails** | Post-generation content safety checks |
| **Attribute Detection** | Automatic sentiment, intent, urgency, and topic classification |
| **Escalation Rules** | Condition-based routing to human agents with priority and department |
| **Function Calling** | CRM tools: create ticket, search KB, escalate, schedule follow-up |
| **Confidence Gate** | Quality threshold-based response gating |
| **Profile Compiler** | Builds rich contact context for every AI interaction |
| **Context Assembler** | Combines conversation history, contact data, and KB results |

**AI Agent Deployment Modes:**
- `autonomous` — AI handles all inbound messages independently
- `copilot` — AI assists human agents with drafts and context (see §4.3)
- `disabled` — no AI involvement

**AI Agent Setup Tabs (Frontend):**
1. **Overview** — Agent name, LLM model, and global toggle
2. **Guidance** — Tone and style configuration
3. **Knowledge Base** — Upload/manage documents and URLs
4. **Guardrails** — Input and output filter rules
5. **Attributes** — Custom attribute detection rules
6. **Escalation** — Smart escalation rules and routing
7. **Deploy** — Channel-specific deployment and testing console

---

### 4.3 AI Copilot (Agent Assist)

CustArea includes an in-inbox AI Copilot that assists human agents in real time without autonomous reply.

**Copilot Capabilities:**

| Tool | Description |
|------|-------------|
| `generate_reply_draft` | Generate a contextual reply draft with tone selection (professional, friendly, formal, empathetic) |
| `summarize_conversation` | Summarize thread as brief, detailed, or action items |
| `search_cross_channel_conversations` | Find all past interactions with a contact across all channels |
| `get_contact_info` | Pull full contact profile and metadata |
| `get_company_guidelines` | Retrieve company policies, SOPs, and templates from KB |
| `get_conversation_metadata` | Analytics: response times, sentiment, engagement metrics |
| `search_knowledge_base` | Semantic search across the tenant's knowledge base |
| `escalate_to_human` | Triggered escalation with routing context |
| `schedule_follow_up` | Schedule meetings, calls, or emails from within the conversation |
| `create_ticket` | Create a support ticket directly from the conversation |

---

### 4.4 AI Assistant Service

The **AI Assistant Service** is a standalone Node.js microservice that acts as an intelligent, natural-language interface to the entire CustArea CRM. Unlike the **AI Agent** (which autonomously handles *customer* messages), the AI Assistant is designed for **internal team members** — letting them query data and trigger CRM actions by simply typing a command in email or Telegram.

```mermaid
flowchart TD
    subgraph Input["Inbound Channels"]
        EMAIL_IN[Email
User emails the assistant]
        TG_IN[Telegram
User sends a /message]
    end

    subgraph Queue["BullMQ Queue (Redis)"]
        QUEUE[assistant_tasks queue
Garanteed delivery + retries]
    end

    subgraph Orchestrator["Orchestrator Pipeline"]
        HISTORY[Load conversation history
MongoDB — last 10 messages]
        TOOLS[Load all tools
get_schema + query_database + action tools]
        LLM[LLM Execution
Multi-step function calling]
        COMPOSE[Compose response]
        CREDITS[Deduct credits]
        SAVE[Save to MongoDB]
    end

    subgraph Reply["Reply via Channel"]
        EMAIL_OUT[Send reply email]
        TG_OUT[Send Telegram message]
    end

    EMAIL_IN --> QUEUE
    TG_IN --> QUEUE
    QUEUE --> HISTORY
    HISTORY --> TOOLS
    TOOLS --> LLM
    LLM --> COMPOSE
    COMPOSE --> CREDITS
    CREDITS --> SAVE
    SAVE --> EMAIL_OUT
    SAVE --> TG_OUT
```

**How It Works:**

| Step | Action |
|------|--------|
| 1 | User sends a natural-language request via email or Telegram |
| 2 | Message is pushed to BullMQ queue (guaranteed delivery, 3 retries, exponential backoff) |
| 3 | Orchestrator loads the last 10 messages of conversation history from MongoDB |
| 4 | All tools loaded (schema discovery + query + action tools) |
| 5 | LLM executes multi-step tool calls: `get_schema()` → `query_database()` → action tool |
| 6 | Response composed and sent back via the originating channel |
| 7 | Credits deducted per tool-call execution |
| 8 | Full exchange saved to MongoDB for conversation continuity |

**Query Tools (Read-only):**

| Tool | Description |
|------|-------------|
| `get_schema` | Dynamically look up table/column structure for any CRM domain |
| `query_database` | Execute SQL read queries against the tenant's CRM data (contacts, leads, conversations, campaigns, analytics, credit balance, escalations, etc.) |

**Action Tools (Write operations):**

| Tool | Description |
|------|-------------|
| `create_contact` | Create a new contact |
| `update_contact` | Update a contact's details |
| `create_leads_from_contacts` | Bulk-create leads from existing contacts |
| `update_lead_stage` | Move a lead to a different pipeline stage |
| `update_lead_status` | Set a lead as active / won / lost |
| `update_lead_score` | Change a lead's score (0–5) |
| `assign_lead` | Assign a lead to a team member |
| `send_message` | Send a message in an open conversation |
| `assign_conversation` | Assign a conversation to a user |
| `update_conversation_status` | Open / pending / resolved / close a conversation |
| `send_email` | Send an email to any address |
| `create_scheduled_item` | Schedule a follow-up meeting, email, phone call, or task |
| `update_scheduled_item` | Reschedule or modify a scheduled item |
| `cancel_scheduled_item` | Cancel a pending scheduled item |
| `make_phone_call` | Initiate an outbound AI voice call to a phone number |
| `send_notification` | Send an email notification to a team member |

**Example Natural-Language Commands:**
```
"Show me all open leads assigned to Sarah"
"Move lead John Doe to the Proposal stage"
"Schedule a follow-up call with +919876543210 tomorrow at 3pm"
"Send an email to john@acme.com with our pricing info"
"Find contacts from Acme Corp and create leads for them"
"What's our current credit balance?"
```

**Channels Supported:**
- **Email** — User emails a dedicated assistant address; replies thread back via the email provider
- **Telegram** — User messages the Telegram bot; replies sent back in-chat

**Safety & Security:**
- Never deletes records (deletion must happen via the CRM UI)
- Always calls `get_schema()` before `query_database()` — never guesses column names
- System prompt, table names, SQL, and internal tools are never revealed to the user
- Tenant data isolation is automatic — `tenant_id` always scoped internally
- Concurrency limit: max 3 simultaneous LLM calls via BullMQ

---

### 4.5 Voice Agents

CustArea supports configurable AI voice agents for inbound and outbound phone calls via Twilio Voice + WebSocket.

```mermaid
flowchart LR
    subgraph Caller["Phone Call"]
        PHONE[Inbound Call]
    end

    subgraph Twilio["Twilio Voice"]
        VOICE[Voice Webhook]
        STREAM[Media Stream\nWebSocket]
    end

    subgraph Backend["CustArea Backend"]
        WS[WebSocket Handler]
        STT[Speech-to-Text\nAzure Cognitive]
        LLM[LLM Processing\nOpenAI/Groq]
        TTS[Text-to-Speech\nAzure Cognitive]
    end

    subgraph Storage["Call Storage"]
        CALL_REC[Call Recording]
        CALL_SUMMARY[AI Summary]
        CRM_LOG[CRM Activity Log]
    end

    PHONE --> VOICE
    VOICE --> STREAM
    STREAM <--> WS

    WS --> STT
    STT --> LLM
    LLM --> TTS
    TTS --> WS
    WS --> STREAM
    WS --> CALL_REC
    WS --> CALL_SUMMARY
    WS --> CRM_LOG
```

**Voice Agent Features:**
- **Real-time STT** — Azure Cognitive Services Speech-to-Text streaming
- **Real-time TTS** — Azure Text-to-Speech for natural voice responses
- **Configurable Personas** — Custom voice agent name, personality, and instructions per tenant
- **Function Calling** — CRM tools usable mid-call (escalate, schedule, search KB)
- **Call Session Manager** — Tracks active call sessions with WebSocket lifecycle
- **Credit-Aware** — Phone call AI usage tracked and deducted from credit balance
- **Call Storage** — Transcripts, summaries, and duration logged to CRM
- **OpenAI Realtime API** — Alternative to legacy STT/TTS pipeline
- **Three WebSocket Modes:**
  - `legacy` — Azure STT → LLM → Azure TTS
  - `realtime` — OpenAI Realtime API (lower latency)
  - `convrelay` — Twilio Conversation Relay

---

### 4.5 Outreach Campaign System

AI-powered cold email campaign engine for B2B outreach at scale.

```mermaid
flowchart TD
    subgraph Setup["Campaign Setup"]
        C1[Select Contact Group]
        C2[Define Objective\nSelling Points, Pain Points]
        C3[Configure AI Instructions]
        C4[Set Daily/Max Limits]
    end

    subgraph Launch["Launch Sequence"]
        CR_CHECK[Credit Check]
        ENROLL[Enroll Contacts]
        SKIP[Skip No-Email Contacts]
        ACTIVATE[Status → Active]
    end

    subgraph Sending["Daily Send Loop"]
        SCHED[Scheduler Worker]
        AI_GEN[AI Email Generation\nPersonalized per contact]
        ROT[Sender Rotation\nMulti-sender fairness]
        SEND[Send via Provider]
        TRACK[Track Analytics]
    end

    subgraph Controls["Campaign Controls"]
        PAUSE[Pause Campaign]
        RESUME[Resume Campaign]
        DELETE[Delete Draft]
    end

    C1 --> C2 --> C3 --> C4
    C4 --> CR_CHECK
    CR_CHECK --> ENROLL
    ENROLL --> SKIP
    ENROLL --> ACTIVATE
    ACTIVATE --> SCHED
    SCHED --> AI_GEN
    AI_GEN --> ROT
    ROT --> SEND
    SEND --> TRACK
    ACTIVATE --> PAUSE
    PAUSE --> RESUME
```

**Campaign Features:**

| Feature | Description |
|---------|-------------|
| **AI-Personalized Emails** | Per-contact email generation using selling points, pain points, value proposition |
| **Multiple Reply Handling** | Human-reply or AI-reply mode per campaign |
| **Sender Rotation** | Distributes sends across multiple sender emails to manage reputation |
| **Daily Limit Enforcement** | Configurable daily send limit (max 200/day) |
| **Contact Cap** | Campaign max contacts limit (max 500 per campaign) |
| **Credit Gating** | Credits checked and deducted per email sent |
| **Analytics Tracking** | Total sent, replies, reply rate, skip rate, bounce tracking |
| **RBAC-Scoped** | Agents only see campaigns for contact groups they have access to |
| **CTA Links** | Attach call-to-action links to each campaign |
| **Language Support** | Multi-language campaign email generation |
| **Draft → Active → Paused → Completed** state machine |

---

### 4.6 Workflow Automation Engine

A visual, no-code workflow builder with event-driven, graph-based execution.

```mermaid
flowchart TB
    subgraph Triggers["Trigger Nodes"]
        T1[WhatsApp Message]
        T2[Email Received]
        T3[New Contact]
        T4[Lead Stage Change]
        T5[Scheduled Time]
    end

    subgraph Logic["Logic Nodes"]
        L1[If/Else Branch]
        L2[Switch Case]
        L3[Delay/Wait]
        L4[Stop]
    end

    subgraph AI["AI Nodes"]
        A1[AI Response\nGenerate text]
        A2[Classify Intent]
        A3[Extract Data]
    end

    subgraph Actions["Output Nodes"]
        O1[Send WhatsApp]
        O2[Send Email]
        O3[Create Lead]
        O4[Create Ticket]
        O5[Assign User]
        O6[Update Contact]
    end

    subgraph Utility["Utility Nodes"]
        U1[Log / Debug]
        U2[Transform Data]
    end

    T1 --> L1
    T2 --> L1
    L1 -->|Condition A| A1
    L1 -->|Condition B| L3

    A1 --> O1
    L3 --> O2

    T3 --> O3
    T4 --> L2
    L2 --> O4
    L2 --> O5
```

**Node Categories:**

| Category | Nodes |
|----------|-------|
| **Triggers** | WhatsApp Message, Email Received, New Contact, Lead Stage Change, Scheduled |
| **Logic** | If/Else, Switch, Delay, Stop |
| **AI** | AI Response, Classify, Extract |
| **Output** | Send WhatsApp, Send Email, Create Lead, Create Ticket, Assign User, Update Contact |
| **Utility** | Logger, Data Transformer |

**Workflow Engine Architecture:**
- Visual node-based builder powered by **React Flow**
- Versioned workflow definitions with rollback support
- `workflow_runs` table tracks every execution with full node-level state
- Delayed nodes create `scheduled_jobs` for resume
- Redis event bus for trigger delivery
- Multi-workflow fan-out for same trigger type
- Full execution history and run logs visible in the UI

---

### 4.7 Sales CRM

```mermaid
flowchart LR
    subgraph Acquisition["Acquisition"]
        C1[Import CSV/XLSX]
        C2[WhatsApp Inbound]
        C3[Email Inbound]
        C4[Widget Chat]
        C5[Manual Entry]
        C6[Campaign Reply]
    end

    subgraph Management["Contact Management"]
        CONTACT[Contact\nIdentity wrapper]
        DEDUP[Cross-channel\nDeduplication]
        GROUPS[Contact Groups\nSegmentation]
    end

    subgraph Pipeline["Sales Pipeline"]
        LEAD[Lead Created]
        S1[Custom Stage 1]
        S2[Custom Stage 2]
        S3[Custom Stage N]
    end

    subgraph Outcome["Outcome"]
        WON[Account/Customer]
        LOST[Lost/Archived]
    end

    C1 --> CONTACT
    C2 --> CONTACT
    C3 --> CONTACT
    C4 --> CONTACT
    C5 --> CONTACT
    C6 --> CONTACT

    CONTACT --> DEDUP
    DEDUP --> GROUPS
    GROUPS --> LEAD

    LEAD --> S1 --> S2 --> S3

    S3 -->|Won| WON
    S3 -->|Lost| LOST
```

**CRM Features:**

| Feature | Description |
|---------|-------------|
| **Contact Management** | Unified identity across all channels with metadata and custom fields |
| **Contact Import** | CSV and Excel (`.xlsx`, `.xls`) bulk import with group assignment |
| **Contact Groups** | Segmentation with user-level RBAC assignments |
| **Lead Board** | Kanban-style pipeline visualization per pipeline |
| **Pipeline Customization** | Multiple pipelines with fully custom stages |
| **Lead Management** | Score, status, assignment, and activity tracking |
| **Accounts** | Won leads convert to customer accounts |
| **Activity Log** | Complete interaction history per contact/lead |
| **Bulk Assignment** | Assign multiple leads/contacts to users at once |
| **Email History** | Dedicated view of all email threads per contact |
| **Phone Call Log** | View call history linked to contacts |

---

### 4.8 Support Ticketing

```mermaid
stateDiagram-v2
    [*] --> New: Ticket Created
    New --> Open: Agent Views
    Open --> Pending: Awaiting Customer
    Pending --> Open: Customer Replies
    Open --> Resolved: Issue Fixed
    Resolved --> Open: Reopened
    Resolved --> Closed: Auto-close after N days
    Closed --> [*]

    note right of New: Auto-created from\nworkflow, AI, or Copilot
    note right of Open: SLA timer active
    note right of Pending: SLA paused
```

**Ticketing Features:**
- **Multi-source creation** — From workflows, AI agents, Copilot tools, or manually
- **Priority Levels** — Urgent, High, Normal, Low
- **Tags** — Custom categorization and filtering
- **Macros** — Pre-defined response templates for fast replies
- **Individual & Team Assignment**
- **SLA Tracking** — Response and resolution timers
- **Linked Conversations** — Tickets linked to source conversations

---

### 4.9 Escalation System

When the AI agent cannot resolve an issue, it triggers the escalation system for intelligent human routing.

**Escalation Flow:**
1. AI agent calls `escalate_to_human` function tool (or escalation rules match)
2. `escalationService.createEscalation()` creates an escalation record
3. Conversation status changes to `escalated`
4. RBAC-aware inbox filter surfaces the conversation to the assigned user
5. Human agent sees the conversation with AI-generated escalation context (reason, summary, department, priority)

**Escalation Metadata Captured:**
- Reason for escalation
- AI-generated conversation summary
- Suggested department (`billing`, `technical`, `sales`, `support`)
- Issue category (`refund`, `bug_report`, `feature_request`, etc.)
- Priority (`low`, `normal`, `high`, `urgent`)
- Escalation source (`ai_agent`, `workflow`, `manual`)

---

### 4.10 Analytics & Reporting

Comprehensive reporting dashboard with time-range filtering, per-user views, and data export.

**Chart Types:**

| Chart | Description |
|-------|-------------|
| `email-ai-vs-human` | AI vs human handled email conversation breakdown |
| `phone-duration` | Call duration trends and distribution |
| `campaign-performance` | Campaign send counts, reply rates, and open tracking |
| `crm-overview` | Lead stage distribution and pipeline health |
| `ticket-status` | Ticket volume by status and priority |

**Analytics Features:**
- **Time Range Filters** — Daily, weekly, monthly, custom date range
- **Per-User Drill-Down** — Super admins can view analytics for individual agents
- **Phone AI Usage** — Separate view for AI-handled call minutes
- **Export** — Download analytics as CSV or JSON
- **Category Filter** — Slice metrics by feature category

---

## 5. Subscription & Credit System

CustArea implements a credit-based consumption model where AI usage across features is tracked and billed against a tenant's credit balance.

### 5.1 Credit-Consuming Features

| Feature | Credit Usage |
|---------|-------------|
| AI Email (campaign) | Per email sent |
| AI Phone (voice agent) | Per call / per minute |
| AI Agent (chat) | Per conversation |
| Email (outbound) | Per email via provider |

### 5.2 Subscription Plans

```mermaid
flowchart LR
    TRIAL[Trial Plan\nLimited credits] --> STARTER[Starter Plan]
    STARTER --> PRO[Pro Plan\nAddon credits enabled]
    PRO --> MAX[Max Plan\nHighest limits]
```

**Plan Features:**
- `credits_included` — Monthly credit allocation per plan
- `allows_addon_credits` — Whether the plan supports purchasing extra credits
- `addon_credit_price` — Per-credit price for addons
- Trial plans with extension request flow
- Upgrade request flow (admin-approved)

### 5.3 Credit Lifecycle

1. **Monthly allocation** — Credits deposited on subscription renewal
2. **Consumption** — Deducted on each AI action via `creditService`
3. **Usage history** — Full paginated ledger with feature-level breakdown
4. **Stats** — Aggregated usage statistics by feature and date range
5. **Addon request** — Tenants can request additional credits (admin-approved)
6. **Trial extension** — Trial tenants can request more trial time with a reason

---

## 6. Access Control & Security

### 6.1 Role-Based Access Control (RBAC)

```mermaid
flowchart TD
    subgraph Auth["Authentication"]
        JWT[JWT Token Auth]
        PWD[Password Hashing\nbcrypt]
        RATE[Rate Limiting\nAuth endpoints]
    end

    subgraph Roles["Built-in Roles"]
        SUPER[Super Admin\nFull platform access]
        OWNER[Owner\nFull tenant access]
        ADMIN[Admin\nTenant config]
        MANAGER[Manager\nTeam oversight]
        AGENT[Agent\nDaily operations]
    end

    subgraph Custom["Custom Roles"]
        CUSTOM[Custom Role\nGranular permission sets]
    end

    subgraph Permissions["Feature Permissions"]
        P1[Contacts]
        P2[Leads]
        P3[Campaigns]
        P4[Tickets]
        P5[Workflow]
        P6[AI Agent]
        P7[Analytics]
        P8[Settings]
    end

    JWT --> SUPER
    JWT --> OWNER
    JWT --> ADMIN
    JWT --> MANAGER
    JWT --> AGENT
    JWT --> CUSTOM

    CUSTOM --> P1
    CUSTOM --> P2
    CUSTOM --> P3
    CUSTOM --> P4
```

**RBAC Features:**
- **Built-in role hierarchy** — Super Admin → Owner → Admin → Manager → Agent
- **Custom roles** — Create roles with granular per-feature permission sets
- **Feature-level access** — Each feature (contacts, leads, campaigns, etc.) has individual read/write/delete flags
- **Contact group RBAC** — Users only see contacts and campaigns for groups they're assigned to
- **Campaign visibility** — Based on contact group membership
- **User Feature Access** — Granular per-user feature overrides on top of roles
- **Onboarding Flow** — Guided setup for new tenants

### 6.2 Security Features

| Feature | Implementation |
|---------|----------------|
| **Authentication** | JWT-based stateless auth |
| **Password Security** | bcrypt hashing |
| **API Security** | Helmet.js, CORS configuration |
| **Rate Limiting** | Auth endpoint rate limiting |
| **Tenant Isolation** | Application-level row filtering on all queries |
| **Credential Storage** | Per-tenant encrypted API keys (Twilio, SES, Gmail, Outlook) |
| **AI Guardrails** | Content filtering for AI responses |
| **Input Validation** | Request sanitization across all endpoints |

---

## 7. Real-Time Capabilities

### 7.1 WebSocket Endpoints

| Endpoint | Purpose |
|----------|---------|
| `/client-audio` | Browser STT streaming |
| `/openai-realtime` | OpenAI Realtime API proxy |
| `/twilio-stream` | Twilio Media Streams (μ-law audio) |
| `/phone-ws/legacy/*` | Azure STT/TTS legacy voice handler |
| `/phone-ws/realtime/*` | OpenAI Realtime voice handler |
| `/phone-ws/convrelay/*` | Twilio Conversation Relay handler |

### 7.2 Redis Streams (Message Bus)

| Stream | Consumer |
|--------|----------|
| `whatsapp_inbound` | WhatsApp inbound worker |
| `whatsapp_outbound` | WhatsApp outbound worker |
| `email_inbound` | Email processing worker |
| `email_outbound` | Email sending worker |
| `ai_incoming` | AI agent worker |
| `event_bus` | Workflow event worker |
| `campaign_send` | Campaign email worker |
| `notifications` | Notification worker |

---

## 8. Technical Workflows

### 8.1 Inbound WhatsApp Message Flow

```mermaid
sequenceDiagram
    participant User as Customer
    participant Twilio as Twilio
    participant Webhook as Backend Webhook
    participant ContactRes as Contact Resolver
    participant ConvMgr as Conversation Manager
    participant Redis as Redis Queue
    participant WFCheck as Workflow Check
    participant WFService as Workflow Service
    participant AIAgent as AI Agent
    participant OutWorker as Outbound Worker

    User->>Twilio: Send WhatsApp Message
    Twilio->>Webhook: POST /webhook/twilio/whatsapp

    Webhook->>ContactRes: findOrCreateContact(phone)
    ContactRes-->>Webhook: {contact, isNew}

    Webhook->>ConvMgr: getOrCreateConversation()
    ConvMgr-->>Webhook: conversation

    Webhook->>Webhook: Create message record

    Webhook->>WFCheck: hasTriggerWorkflow?

    alt Has Active Workflow
        WFCheck-->>Webhook: true
        Webhook->>Redis: Queue with trigger_data
        Redis->>WFService: Event Worker picks up
        WFService->>WFService: Execute workflow nodes
        WFService->>Redis: Queue outbound message
    else No Workflow, AI Enabled
        WFCheck-->>Webhook: false
        Webhook->>Redis: Queue for AI
        Redis->>AIAgent: AI Worker picks up
        AIAgent->>AIAgent: RAG + LLM processing
        AIAgent->>Redis: Queue response
    end

    Redis->>OutWorker: WhatsApp outbound worker
    OutWorker->>Twilio: Send message
    Twilio->>User: Deliver response
```

### 8.2 AI Campaign Email Send Flow

```mermaid
sequenceDiagram
    participant Sched as Scheduler Worker
    participant DB as PostgreSQL
    participant AI as AI Email Generator
    participant Credits as Credit Service
    participant Rot as Sender Rotation
    participant Provider as Email Provider
    participant Analytics as Campaign Analytics

    Sched->>DB: Find active campaigns with pending contacts
    DB-->>Sched: campaigns[]

    loop Each Campaign
        Sched->>DB: Get pending contacts (up to daily_limit)
        loop Each Contact
            Sched->>AI: Generate personalized email
            AI-->>Sched: {subject, body}
            Sched->>Credits: Deduct campaign credits
            Sched->>Rot: Get next sender email
            Rot-->>Sched: sender
            Sched->>Provider: Send email (SES/Gmail/Outlook)
            Provider-->>Sched: sent/failed
            Sched->>DB: Update campaign_contact status
            Sched->>Analytics: Update campaign analytics
        end
    end
```

### 8.3 Escalation Flow

```mermaid
sequenceDiagram
    participant AIAgent as AI Agent Worker
    participant FnTools as Function Tools
    participant EscSvc as Escalation Service
    participant DB as PostgreSQL
    participant Inbox as Human Inbox

    AIAgent->>FnTools: escalate_to_human(reason, priority, department)
    FnTools->>EscSvc: createEscalation(tenantId, escalationData)
    EscSvc->>DB: Insert into escalations table
    EscSvc->>DB: Update conversation status = 'escalated'
    EscSvc->>DB: Assign to agent based on routing rules
    DB-->>Inbox: Conversation appears in assigned agent's inbox
    Inbox-->>AIAgent: Escalation confirmed
```

---

## 9. Technology Stack

### 9.1 Frontend

| Technology | Version | Purpose |
|------------|---------|---------|
| **Next.js** | 15 | React framework with App Router |
| **TypeScript** | 5+ | Type-safe development |
| **Tailwind CSS** | 3 | Utility-first styling |
| **React Flow** | Latest | Visual workflow node builder |
| **Zustand** | Latest | Client-side state management |
| **ShadCN UI** | Latest | Component library |

### 9.2 Backend

| Technology | Version | Purpose |
|------------|---------|---------|
| **Node.js** | 20+ | Runtime environment |
| **Express.js** | 4 | HTTP server framework |
| **PostgreSQL** | 15+ | Primary relational database |
| **MongoDB** | 6+ | AI agent configuration store |
| **Redis Streams** | 7+ | Message queue and pub/sub |
| **WebSocket (ws)** | Latest | Real-time voice communication |
| **BullMQ / node-cron** | Latest | Job scheduling |

### 9.3 AI / ML Stack

| Technology | Purpose |
|------------|---------|
| **OpenAI API** | GPT-4o, GPT-4o-mini for chat and function calling |
| **OpenAI Realtime API** | Low-latency voice conversation |
| **Groq API** | LLaMA 3.1 for fast inference fallback |
| **OpenAI Embeddings** | `text-embedding-3-small` for knowledge base |
| **Vector Search** | Semantic document chunk retrieval (pgvector / MongoDB) |
| **Azure Cognitive Services** | Speech-to-Text and Text-to-Speech for voice agents |

### 9.4 External Integrations

| Service | Purpose |
|---------|---------|
| **Twilio** | WhatsApp Business API, Voice calls, Media Streams |
| **AWS SES** | Transactional and campaign email sending |
| **Gmail OAuth** | Connected Gmail accounts as email senders/receivers |
| **Microsoft OAuth (Outlook)** | Connected Outlook accounts as email senders/receivers |
| **Azure Cognitive** | STT/TTS for voice agents |

---

## 10. Project Structure

```
CustArea/
├── backend/                    # Core API server (Express.js, port 8000)
│   ├── ai-agent/               # AI agent system (pipeline, tools, models)
│   │   ├── services/           # aiPipeline, copilotService, functionTools, vectorSearch
│   │   └── models/             # Agent, KnowledgeSource, KnowledgeChunk (MongoDB)
│   ├── campaign/               # Outreach campaign system
│   │   └── services/           # campaignService, campaignAIService, emailRotation
│   ├── email/                  # Email multi-provider system
│   │   └── services/           # sesProvider, gmailProvider, outlookProvider, creditAware
│   ├── phone/                  # Voice call system
│   │   └── services/           # realtimeHandler, legacyHandler, phoneCredits
│   ├── voice-agents/           # Voice agent configuration & routing
│   ├── escalation/             # Escalation service and routing
│   ├── conversations/          # Conversation management
│   ├── notifications/          # In-app notification system
│   ├── scheduling/             # Follow-up and scheduled job handling
│   ├── controllers/            # 24 feature controllers
│   ├── services/               # Analytics, credit, subscription, RBAC, permissions
│   ├── middleware/             # Auth, RBAC, rate limiting (13 middleware files)
│   └── routes/                 # 23 route files
│
├── workflow-service/           # Workflow engine (Express.js, port 8001)
│   ├── engine/                 # Core executor, event worker, scheduler
│   ├── nodes/                  # ai/, logic/, output/, triggers/, utility/
│   └── workers/                # Event consumer, scheduler consumer
│
├── ai-assistant-service/       # Natural-language CRM command assistant (Email + Telegram)
│   ├── core/                   # Orchestrator, taskExecutor, responseComposer
│   ├── channels/               # emailChannel.js, telegramChannel.js
│   ├── tools/                  # queryDatabase, actionTools (16 CRM tools), toolRegistry
│   ├── workers/                # assistantWorker (BullMQ), emailWorker
│   └── models/                 # AssistantConversation (MongoDB history)
│
├── client/                     # Next.js dashboard (port 3000)
│   └── src/app/(dashboard)/
│       ├── ai-agent/           # Setup, Knowledge, Deploy, Test
│       ├── campaign/           # Outreach campaign management
│       ├── conversation/       # Unified inbox
│       ├── dashboard/          # Home dashboard
│       ├── email-history/      # Email thread history
│       ├── phone-calls/        # Call logs and recordings
│       ├── report/             # Analytics & reporting
│       ├── sales/              # Contacts, Leads, Lead Board, Groups, Accounts
│       ├── settings/           # Profile, Users, Roles, Email, Integrations, Subscription
│       ├── tickets/            # Support ticketing
│       ├── voice-agents/       # Voice agent configuration
│       └── workflow/           # Visual workflow builder
│
├── chat-widget/                # Embeddable website chat widget (Vite)
├── landing-page/               # Public marketing site
├── admin-frontend/             # Super-admin panel
├── db/                         # PostgreSQL migrations and seeds
└── documentation/              # Internal documentation
```

---

## 11. Getting Started

### Prerequisites

- Node.js 20+
- PostgreSQL 15+
- MongoDB 6+
- Redis 7+
- Twilio account (WhatsApp + Voice)
- OpenAI API key
- AWS SES credentials (for email)

### Environment Variables

Each service has its own `.env` file. Copy `.env.example` and populate:

**Backend (`backend/.env`):**
```env
# Database
DATABASE_URL=postgresql://user:pass@localhost:5432/custarea
MONGODB_URI=mongodb://localhost:27017/custarea
REDIS_URL=redis://localhost:6379

# AI
OPENAI_API_KEY=sk-...
GROQ_API_KEY=gsk_...

# Twilio
TWILIO_ACCOUNT_SID=AC...
TWILIO_AUTH_TOKEN=...

# Email
AWS_ACCESS_KEY_ID=...
AWS_SECRET_ACCESS_KEY=...
AWS_SES_REGION=us-east-1

# Azure
AZURE_SPEECH_KEY=...
AZURE_SPEECH_REGION=eastus

# Auth
JWT_SECRET=your-secret-key
```

**Workflow Service (`workflow-service/.env`):**
```env
DATABASE_URL=postgresql://user:pass@localhost:5432/custarea
REDIS_URL=redis://localhost:6379
BACKEND_API_URL=http://localhost:8000
```

### Running Locally

```bash
# 1. Install dependencies for all services
cd backend && npm install
cd ../workflow-service && npm install
cd ../client && npm install
cd ../chat-widget && npm install

# 2. Run database migrations
cd db && npm run migrate

# 3. Start all services
cd backend && npm start          # Port 8000
cd workflow-service && npm start # Port 8001
cd client && npm run dev         # Port 3000
```

### Docker (Recommended)

```bash
# Build and run all services with docker-compose
docker compose up --build
```

---

## 12. Key Architectural Decisions

| Decision | Rationale |
|----------|-----------|
| **Shared DB, shared schema** | Simpler operations, lower cost; isolation enforced at app layer via `tenant_id` |
| **Redis Streams for queuing** | Durable, ordered message delivery with consumer group replay |
| **MongoDB for AI config** | Flexible schema for evolving agent configuration without migrations |
| **Multi-provider email** | Avoid SES dependency; Gmail/Outlook OAuth expands deliverability surface |
| **Credit-aware service layer** | Wrap every AI call in credit check to prevent runaway costs |
| **Function calling for CRM** | AI agents take structured actions rather than hallucinating API calls |
| **Copilot vs autonomous mode** | Allows human-first teams to adopt AI incrementally |
| **React Flow for workflows** | Production-grade node editor with extensible custom node system |
