# 🦅 Fraud Line Detection: The One-in-a-Million Multi-Agent Triage Pipeline

Welcome to the **most advanced, enterprise-grade multi-agent fraud reporting and triage system** built as an **MCP (Model Context Protocol) Server**. Powered by the highly exclusive `@nitrostack/core` framework, this system isn't just another pipeline—it is a *deterministic powerhouse* designed to automate and scale complex legal, assignment, and triage operations with unparalleled precision.

This is a **one-in-a-million** architecture. While others rely on unpredictable LLM routing, our system leverages absolute deterministic control over a sophisticated 3-agent intelligence swarm.

**Tech Stack:** TypeScript, Node.js, **Neon (Serverless PostgreSQL)**, Zod validation, MCP Protocol, `@nitrostack/core`

---

## 🌟 Why This Project is Elite

1. **Deterministic Orchestration:** We don't leave control flow to chance. Our backend dictates the absolute path of execution, while isolated AI agents handle the deeply complex cognitive work (classification, legal mapping, personnel assignment).
2. **Neon Serverless PostgreSQL:** Lightning-fast, infinitely scalable, serverless data access. By leveraging **Neon**, this pipeline handles massive data concurrency without breaking a sweat, ensuring instant recovery and global scale.
3. **Self-Updating API Keys:** Security isn't an afterthought. Our system architecture embraces **self-updating API keys**, drastically reducing the attack surface and ensuring high-availability operations never fail due to stale credentials.
4. **Agent Isolation:** Zero cross-contamination. Agents operate in strict information silos, receiving only the precise, validated JSON output they need. This maximizes context window efficiency and eliminates hallucination bleed.

---

## 🧠 System Architecture Flowchart

```mermaid
flowchart TD

    %% Node Definitions & Shapes
    START(["Start: Fraud Report Submitted"])
    EXT_SUB["External Ticket Submission (Victim, Fraud Details, Fraudster Info, Region, Attachments)"]
    
    subgraph ORCH_CONTAINER["MCP Server Runtime"]
        MCP_SERVER["MCP Server: fraud-pipeline-mcp"]
        ORCH["Orchestrator: run_fraud_pipeline"]
        VAL_TICKET["Validate Input Ticket (Zod TicketSchema)"]
    end

    %% Databases / Storage
    MOCK_DB[("Mock Tickets Database (Shared UPI: fraudster@paytm)")]
    PG_DB[("Neon Serverless PostgreSQL")]

    %% Agent 1 Subgraph
    subgraph AGENT1_SG["Agent 1: Triage & Classification (Deterministic Logic)"]
        A1_FETCH["Fetch Ticket: get_ticket"]
        A1_RELATED["Fetch Related Tickets: get_related_tickets (UPI, Bank, Phone, IFSC)"]
        A1_CLASSIFY["Classify Fraud Type (UPI, Card, Cheque, Phishing, etc.)"]
        A1_SCALE["Estimate Scale (Victim count, Pattern suspected)"]
        A1_URGENCY["Calculate Urgency (Low / Medium / High / Critical)"]
        A1_RISK["Calculate Risk Score (0 - 100)"]
        A1_GAPS["Identify Evidence Gaps"]
        
        A1_FETCH --> A1_RELATED --> A1_CLASSIFY --> A1_SCALE --> A1_URGENCY --> A1_RISK --> A1_GAPS
    end

    %% Fan-Out / Merge Nodes
    FORK{"Parallel Execution (Fan-Out)"}
    JOIN{"Merge Agent Outputs (Fan-In)"}

    %% Agent 2 Subgraph
    subgraph AGENT2_SG["Agent 2: Assignment & Routing"]
        A2_DEPT["Get Dept Directory: get_department_directory (Match Jurisdiction + Specialization)"]
        A2_CAPACITY["Select Best Dept (Capacity Ratio: Caseload / Capacity)"]
        A2_AVAIL["Get Personnel Availability: get_personnel_availability"]
        A2_PERSONNEL["Select Available Personnel (Filter Status, Slice Team Size)"]
        A2_TEAM["Decide Team Size (Individual | Small Team | Full Team)"]
        A2_ESCALATE["Set Escalation Flag (True if Critical, Pattern, or Risk >= 80)"]
        A2_OUT["Output: Agent2AssignmentOutput"]

        A2_DEPT --> A2_CAPACITY --> A2_AVAIL --> A2_PERSONNEL --> A2_TEAM --> A2_ESCALATE --> A2_OUT
    end

    %% Agent 3 Subgraph
    subgraph AGENT3_SG["Agent 3: Legal & Compliance"]
        A3_SEARCH["Search Legal Corpus: search_legal_corpus (5 Fallback Strategies)"]
        A3_MAP["Map Applicable Laws (Name, Section, Summary, Citation)"]
        A3_ACTIONS["Build Suggested Actions (Action, Legal Basis, Urgency)"]
        A3_OUT["Output: Agent3LegalOutput"]

        A3_SEARCH --> A3_MAP --> A3_ACTIONS --> A3_OUT
    end

    %% Output & Storage / Dashboard
    BUILD_PACKET["Build Master Case Packet (Combines Ticket + Triage + Assignment + Legal)"]
    DASHBOARD["Dashboard Summary (Title, Priority, Dept, Personnel Count, Legal Citations, Escalation Flag, Statutes)"]
    END_NODE(["END: Assigned Authority Dashboard (Human Enforces / Rejects / Modifies)"])

    %% Data Connections & Fallbacks
    FALLBACK_DEPT["Hardcoded Dept Directory (Fallback Data)"]
    FALLBACK_LEGAL["Hardcoded Legal Corpus (Fallback Data)"]
    DEGRADATION["Graceful Degradation (Return Empty Results on Error)"]

    %% Pipeline Connections
    START --> EXT_SUB --> MCP_SERVER --> ORCH --> VAL_TICKET --> AGENT1_SG
    
    %% Agent 1 DB Flow
    MOCK_DB -.-> A1_FETCH
    MOCK_DB -.-> A1_RELATED
    AGENT1_SG --> FORK
    FORK --> AGENT2_SG
    FORK --> AGENT3_SG

    %% Agent 2 DB & Fallback Flows
    PG_DB -.-> A2_DEPT
    PG_DB -.-> A2_AVAIL
    A2_DEPT -.->|DB Error| FALLBACK_DEPT
    A2_AVAIL -.->|DB Error| DEGRADATION

    %% Agent 3 DB & Fallback Flows
    PG_DB -.-> A3_SEARCH
    A3_SEARCH -.->|DB Error| FALLBACK_LEGAL

    %% Fan-In & Final Pipeline
    A2_OUT --> JOIN
    A3_OUT --> JOIN
    JOIN --> BUILD_PACKET --> DASHBOARD --> END_NODE

    %% Styling
    classDef orchClass fill:#6b21a8,stroke:#581c87,color:#ffffff,font-weight:bold;
    classDef agent1Class fill:#1e40af,stroke:#1e3a8a,color:#ffffff,font-weight:bold;
    classDef agent2Class fill:#15803d,stroke:#166534,color:#ffffff,font-weight:bold;
    classDef agent3Class fill:#c2410c,stroke:#9a3412,color:#ffffff,font-weight:bold;
    classDef dbClass fill:#4b5563,stroke:#374151,color:#ffffff,font-weight:bold;
    classDef fallbackClass fill:#991b1b,stroke:#7f1d1d,color:#ffffff,stroke-dasharray: 5 5;
    classDef startEndClass fill:#0f172a,stroke:#0284c7,color:#ffffff,font-weight:bold;

    class MCP_SERVER,ORCH,VAL_TICKET orchClass;
    class A1_FETCH,A1_RELATED,A1_CLASSIFY,A1_SCALE,A1_URGENCY,A1_RISK,A1_GAPS agent1Class;
    class A2_DEPT,A2_CAPACITY,A2_AVAIL,A2_PERSONNEL,A2_TEAM,A2_ESCALATE,A2_OUT agent2Class;
    class A3_SEARCH,A3_MAP,A3_ACTIONS,A3_OUT agent3Class;
    class MOCK_DB,PG_DB dbClass;
    class FALLBACK_DEPT,FALLBACK_LEGAL,DEGRADATION fallbackClass;
    class START,END_NODE startEndClass;
```

---

## ⚡ The Deterministic Pipeline

The system processes massive fraud reports through a highly controlled, zero-latency 3-agent pipeline:

| Phase | System Element | Core Function |
|------|-----------|-------------|
| **00** | **Orchestrator** | Validates the input ticket with Zod schemas and initiates the strict determinist loop. |
| **01** | **Agent 1 — Triage** | Ingests data, classifies fraud vectors, estimates blast radius, urgency, and assesses risk. |
| **02** | **Agent 2 — Assignment** | Queries Neon DB to route cases to specialized departments based on real-time capacity and metrics. |
| **03** | **Agent 3 — Legal** | Performs a multi-layered RAG search against legal corpus mapping infractions to strict statutes. |
| **04** | **Compiler** | Fuses structured JSON outputs from all agents into the **Master Case Packet** ready for final human enforcement. |

---

## 🛠️ MCP Tools Exposed

These tools bridge the gap between our agents and absolute backend data authority:

- `get_ticket(ticket_id)` - Retrieves verified case files instantly.
- `get_related_tickets(criteria)` - Cross-references UPI, Bank accounts, and IPs for deep pattern/syndicate detection.
- `get_department_directory(jurisdiction, specialization)` - Uses load-balancing algorithms to find matching legal/cyber departments.
- `get_personnel_availability(department_id)` - Assigns available agents in real-time.
- `search_legal_corpus(fraud_type, jurisdiction, query?)` - Employs a robust 5-tier fallback search protocol over legal text.
- `run_fraud_pipeline(ticket)` - The main detonator for the end-to-end execution.

---

## 🏗️ Project Structure

Clean, modular, and built to scale horizontally:

```
src/
├── index.ts                              # MCP server entry point
├── app.module.ts                         # Root application module
├── modules/fraud-pipeline/
│   ├── fraud-pipeline.module.ts          # Core Pipeline Registration
│   ├── ticket.tools.ts                   # Tool Access Layer
│   └── triage-agent.prompts.ts           # Strict Agent System Prompts
├── agent-2-assignment/                   # Department/Personnel Routing Logic
├── agent-3-legal/                        # Advanced Compliance & Statute Engine
├── orchestrator/
│   ├── pipeline.orchestrator.ts          # The Deterministic Brain
│   └── case-packet.schema.ts             # Output Schemas
├── schemas/                              # Zod Truth Sources
├── db/
│   └── pool.ts                           # Serverless Neon DB Connections
└── health/                               # Telemetry & System Health
```

---

## 🚀 Getting Started

Experience the power of the pipeline on your own machine. 

### Prerequisites
- Node.js 18+
- Neon PostgreSQL connection string

### Installation & Launch

```bash
# 1. Install dependencies
npm install

# 2. Setup your highly secure environment variables
cp .env.example .env

# Edit .env with your Neon DATABASE_URL and Self-Updating API configurations

# 3. Ignite the server
npm run dev
```

### Environment Variables

| Variable | Description | Required |
|----------|-------------|----------|
| `DATABASE_URL` | PostgreSQL connection string | No (falls back to mock data) |
| `NODE_ENV` | `development` or `production` | No (defaults to development) |

## 🛡️ License

Private, Exclusive & Highly Confidential.
*(Operating under MIT for standard open-source framework integrations)*