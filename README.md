# Fraud Line Detection — Multi-Agent Fraud Reporting & Triage System

A **multi-agent fraud reporting & triage system** built as an **MCP (Model Context Protocol) server** using the `@nitrostack/core` framework. It processes financial fraud tickets through a deterministic pipeline of 3 agents, producing a **Master Case Packet** with triage classification, department assignment, and legal guidance.

**Tech Stack:** TypeScript, Node.js, PostgreSQL, Zod validation, MCP protocol, `@nitrostack/core`

---

## System Architecture Flowchart

```mermaid
flowchart TD

    %% Node Definitions & Shapes
    START(["Start: Fraud Report Submitted"])
    EXT_SUB["External Ticket Submission<br><i>(Victim, Fraud Details, Fraudster Info,<br>Region, Attachments)</i>"]
    
    subgraph ORCH_CONTAINER["MCP Server Runtime"]
        MCP_SERVER["MCP Server<br><code>fraud-pipeline-mcp</code>"]
        ORCH["Orchestrator<br><code>run_fraud_pipeline</code>"]
        VAL_TICKET["Validate Input Ticket<br><i>(Zod TicketSchema)</i>"]
    end

    %% Databases / Storage
    MOCK_DB[("Mock Tickets Database<br><i>(Shared UPI: fraudster@paytm)</i>")]
    PG_DB[("PostgreSQL Database")]

    %% Agent 1 Subgraph
    subgraph AGENT1_SG["Agent 1: Triage & Classification (Deterministic Logic)"]
        direction TB
        A1_FETCH["Fetch Ticket<br><code>get_ticket</code>"]
        A1_RELATED["Fetch Related Tickets<br><code>get_related_tickets</code><br><i>(UPI, Bank, Phone, IFSC)</i>"]
        A1_CLASSIFY["Classify Fraud Type<br><i>(UPI, Card, Cheque, Phishing, etc.)</i>"]
        A1_SCALE["Estimate Scale<br><i>(Victim count, Pattern suspected)</i>"]
        A1_URGENCY["Calculate Urgency<br><i>(Low / Medium / High / Critical)</i>"]
        A1_RISK["Calculate Risk Score<br><i>(0 - 100)</i>"]
        A1_GAPS["Identify Evidence Gaps"]
        
        A1_FETCH --> A1_RELATED --> A1_CLASSIFY --> A1_SCALE --> A1_URGENCY --> A1_RISK --> A1_GAPS
    end

    %% Fan-Out / Merge Nodes
    FORK{"Parallel Execution<br>(Fan-Out)"}
    JOIN{"Merge Agent Outputs<br>(Fan-In)"}

    %% Agent 2 Subgraph
    subgraph AGENT2_SG["Agent 2: Assignment & Routing"]
        direction TB
        A2_DEPT["Get Dept Directory<br><code>get_department_directory</code><br><i>(Match Jurisdiction + Specialization)</i>"]
        A2_CAPACITY["Select Best Dept<br><i>(Capacity Ratio: Caseload / Capacity)</i>"]
        A2_AVAIL["Get Personnel Availability<br><code>get_personnel_availability</code>"]
        A2_PERSONNEL["Select Available Personnel<br><i>(Filter Status, Slice Team Size)</i>"]
        A2_TEAM["Decide Team Size<br><i>(Individual | Small Team | Full Team)</i>"]
        A2_ESCALATE["Set Escalation Flag<br><i>(True if Critical, Pattern, or Risk ≥ 80)</i>"]
        A2_OUT["Output: Agent2AssignmentOutput"]

        A2_DEPT --> A2_CAPACITY --> A2_AVAIL --> A2_PERSONNEL --> A2_TEAM --> A2_ESCALATE --> A2_OUT
    end

    %% Agent 3 Subgraph
    subgraph AGENT3_SG["Agent 3: Legal & Compliance"]
        direction TB
        A3_SEARCH["Search Legal Corpus<br><code>search_legal_corpus</code><br><i>(5 Fallback Strategies)</i>"]
        A3_MAP["Map Applicable Laws<br><i>(Name, Section, Summary, Citation)</i>"]
        A3_ACTIONS["Build Suggested Actions<br><i>(Action, Legal Basis, Urgency)</i>"]
        A3_OUT["Output: Agent3LegalOutput"]

        A3_SEARCH --> A3_MAP --> A3_ACTIONS --> A3_OUT
    end

    %% Output & Storage / Dashboard
    BUILD_PACKET["Build Master Case Packet<br><i>(Combines Ticket + Triage + Assignment + Legal)</i>"]
    DASHBOARD["Dashboard Summary<br><i>(Title, Priority, Dept, Personnel Count,<br>Legal Citations, Escalation Flag, Statutes)</i>"]
    END_NODE(["END: Assigned Authority Dashboard<br><i>(Human Enforces / Rejects / Modifies)</i>"])

    %% Data Connections & Fallbacks
    FALLBACK_DEPT["Hardcoded Dept Directory<br><i>(Fallback Data)</i>"]
    FALLBACK_LEGAL["Hardcoded Legal Corpus<br><i>(Fallback Data)</i>"]
    DEGRADATION["Graceful Degradation<br><i>(Return Empty Results on Error)</i>"]

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
    A2_DEPT -. DB Error .-> FALLBACK_DEPT
    A2_AVAIL -. DB Error .-> DEGRADATION

    %% Agent 3 DB & Fallback Flows
    PG_DB -.-> A3_SEARCH
    A3_SEARCH -. DB Error .-> FALLBACK_LEGAL

    %% Fan-In & Final Pipeline
    A2_OUT --> JOIN
    A3_OUT --> JOIN
    JOIN --> BUILD_PACKET --> DASHBOARD --> END_NODE

    %% Styling / Custom Themes
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

## Pipeline Overview

The system processes fraud reports through a **deterministic 3-agent pipeline**:

| Step | Component | Description |
|------|-----------|-------------|
| 1 | **Orchestrator** | Validates the input ticket and initiates the pipeline |
| 2 | **Agent 1 — Triage** | Classifies fraud type, estimates scale/urgency/risk, detects patterns |
| 3 | **Agent 2 — Assignment** | Routes the case to the best department and personnel |
| 4 | **Agent 3 — Legal** | Searches legal corpus for applicable laws and suggested actions |
| 5 | **Merge** | Combines all outputs into a Master Case Packet with dashboard summary |

> **Key Design Decision:** The orchestrator is **deterministic backend code**, not an LLM. LLMs decide *content* (classification, assignment, legal suggestions); the backend decides *flow*. This ensures the order of operations is auditable and reproducible.

---

## MCP Tools Exposed

| Tool | Used By | Purpose |
|------|---------|---------|
| `get_ticket(ticket_id)` | Agent 1 | Fetch raw fraud ticket by UUID |
| `get_related_tickets(criteria)` | Agent 1 | Find tickets sharing fraudster identifiers (pattern detection) |
| `get_department_directory(jurisdiction, specialization)` | Agent 2 | Find matching departments with capacity data |
| `get_personnel_availability(department_id)` | Agent 2 | Get personnel with availability status |
| `search_legal_corpus(fraud_type, jurisdiction, query?)` | Agent 3 | RAG search over legal/regulatory corpus |
| `run_fraud_pipeline(ticket)` | Orchestrator | End-to-end pipeline execution |

---

## Agent Output Schemas

### Agent 1 — Triage & Classification

```json
{
  "ticket_id": "uuid",
  "fraud_type": "upi_fraud | card_fraud | cheque_fraud | phishing | investment_scam",
  "scale": {
    "victim_count_estimate": 1,
    "pattern_suspected": false,
    "related_ticket_ids": []
  },
  "urgency": {
    "level": "low | medium | high | critical",
    "revocability_window_remaining": "estimate, not authoritative",
    "reasoning": "..."
  },
  "risk_score": 0,
  "evidence_gaps": []
}
```

### Agent 2 — Assignment & Routing

```json
{
  "ticket_id": "uuid",
  "assigned_department_id": "uuid",
  "assigned_personnel": [{ "id": "uuid", "role": "officer" }],
  "team_size_recommendation": "individual | small_team | full_team",
  "reasoning": "...",
  "escalation_flag": false
}
```

### Agent 3 — Legal & Compliance

```json
{
  "ticket_id": "uuid",
  "jurisdiction": "IN-KA",
  "applicable_laws": [
    {
      "name": "IT Act 2000",
      "section": "66C",
      "summary": "...",
      "source_url": "https://...",
      "relevance": "..."
    }
  ],
  "suggested_actions": [
    {
      "action": "...",
      "legal_basis": "IT Act 2000 66C",
      "urgency": "high",
      "citation": "https://..."
    }
  ],
  "confidence_notes": "..."
}
```

---

## Key Design Decisions

### 1. Deterministic Orchestration
The pipeline controller is backend code, not an LLM. This ensures the order of operations is auditable and reproducible — critical for legal/financial decisions.

### 2. Agent Isolation
Agents 2 and 3 only see Agent 1's structured JSON output, never the raw ticket. This keeps prompts focused and outputs independently validatable.

### 3. Graceful Degradation
Every database-backed tool has hardcoded fallback data. If PostgreSQL is unreachable, the pipeline still works with static data.

### 4. Multi-layered Legal Search
The legal corpus search uses 5 fallback strategies: exact match → keyword ILIKE → keyword without query → jurisdiction-only → broad digital-payment fallback.

### 5. Fraud Type Normalization
The system handles verbose fraud descriptions like "UPI QR Code Scam (Organized/Repeat)" and extracts canonical fraud types using keyword-to-fraud-type maps.

---

## Project Structure

```
src/
├── index.ts                              # MCP server entry point
├── app.module.ts                         # Root application module
├── modules/fraud-pipeline/
│   ├── fraud-pipeline.module.ts          # Pipeline module registration
│   ├── ticket.tools.ts                   # get_ticket, get_related_tickets tools
│   ├── triage-agent.prompts.ts           # Agent 1 prompt definitions
│   ├── triage-agent.system-prompt.ts     # Agent 1 system prompt
│   └── mock-tickets.ts                   # 4 mock tickets for testing
├── agent-2-assignment/
│   ├── assignment.tools.ts               # get_department_directory, get_personnel_availability
│   └── assignment-agent.prompts.ts       # Agent 2 prompt definitions
├── agent-3-legal/
│   ├── legal.tools.ts                    # search_legal_corpus tool
│   └── legal-agent.prompts.ts            # Agent 3 prompt definitions
├── orchestrator/
│   ├── pipeline.orchestrator.ts          # Deterministic pipeline controller
│   └── case-packet.schema.ts             # Master Case Packet schema
├── schemas/
│   ├── common.ts                         # Shared Zod schemas
│   ├── ticket.schema.ts                  # Ticket data model
│   ├── agent-output.schema.ts            # Agent 1/2/3 output schemas
│   └── department.schema.ts              # Department & Personnel schemas
├── db/
│   └── pool.ts                           # PostgreSQL connection pool
└── health/
    └── system.health.ts                  # System health check
```

---

## Getting Started

### Prerequisites
- Node.js 18+
- PostgreSQL (optional — system works with fallback data)

### Installation

```bash
# Install dependencies
npm install

# Set up environment
cp .env.example .env
# Edit .env with your DATABASE_URL if using PostgreSQL

# Run in development mode
npm run dev
```

### Environment Variables

| Variable | Description | Required |
|----------|-------------|----------|
| `DATABASE_URL` | PostgreSQL connection string | No (falls back to mock data) |
| `NODE_ENV` | `development` or `production` | No (defaults to development) |

---

## License

MIT