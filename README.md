<div align="center">

# TestAI 

# Intelligent Test Case Generation for Regulated Software

TestAI is an end-to-end AI platform that converts software requirement documents (PRDs, BRDs, SRS) into
regulation-aware, traceable test cases. It was built as a submission for the Schneider Hackathon 2026
(Automating Test Case Generation with AI).

![Test AI](testai.png)

The system targets healthcare and regulated industries where manual test authoring is time-consuming,
prone to gaps, and must align with frameworks such as GDPR, HIPAA, FDA 21 CFR Part 11, CCPA, and SOC 2.

Live demo: https://test-ai-gcp.vercel.app  

---

## Problem Statement

Engineering teams operating in regulated domains — healthcare, finance, industrial control — spend
disproportionate time writing test cases manually from lengthy requirement documents. This process is
error-prone, difficult to audit, and rarely keeps pace with document churn. Compliance traceability
(the direct linkage from a regulatory article to a specific test) is almost never maintained in
practice, creating audit exposure.

TestAI solves this by running every uploaded PDF through a multi-stage AI pipeline that understands
both the intent of a requirement and the regulatory context it must satisfy, then produces structured,
traceable test cases that can be exported directly to Jira.

---

## System Architecture

```mermaid
flowchart TD
    U([User / Engineer]) -->|Uploads PRD or SRS PDF| FE

    subgraph FE["Frontend  (Next.js 15 on Vercel)"]
        direction TB
        AUTH["Google OAuth via NextAuth"]
        UI["Chat Interface + PDF Viewer"]
        EXPORT["Jira Export Workflow"]
    end

    FE -->|Multipart POST| API

    subgraph API["API Layer  (FastAPI on Cloud Run)"]
        direction TB
        EP1["/extract-document"]
        EP2["/extract-mask"]
        EP3["/rag-enhance"]
        EP4["/build-knowledge-graph"]
        EP5["/generate-ui-tests"]
    end

    API -->|PDF bytes| DOCAI
    DOCAI -->|Structured chunks + entities| DLP
    DLP -->|PII-masked chunks| RAG
    RAG -->|Compliance-enriched context| KG
    KG -->|Graph nodes + edges| GEM

    subgraph GCP["Google Cloud Platform"]
        direction TB
        DOCAI["Document AI\nText extraction + entity detection"]
        DLP["Cloud DLP\nPII detection and masking (GDPR mode)"]
        RAG["Vertex AI RAG\nCompliance corpus retrieval"]
        KG["Knowledge Graph Builder\nRequirements, Regulations, Articles, Tests"]
        GEM["Gemini 2.0 Flash\nTest case synthesis"]
    end

    GEM -->|Structured test cases| FE
    FE -->|OAuth push| JIRA["Jira Cloud\n(ALM Integration)"]

    classDef fe    fill:#0055CC,stroke:#003d99,color:#fff
    classDef api   fill:#00695C,stroke:#004D40,color:#fff
    classDef gcp   fill:#1565C0,stroke:#0D47A1,color:#fff
    classDef step  fill:#1976D2,stroke:#1565C0,color:#fff
    classDef ext   fill:#E65100,stroke:#BF360C,color:#fff
    classDef user  fill:#4A148C,stroke:#311B92,color:#fff

    class U user
    class FE,AUTH,UI,EXPORT fe
    class API,EP1,EP2,EP3,EP4,EP5 api
    class DOCAI,DLP,RAG,KG,GEM gcp
    class JIRA ext
```

---

## Use Case Diagram

```mermaid
flowchart LR
    A([Test Engineer]) --- UC1
    A --- UC2
    A --- UC3
    A --- UC4
    A --- UC5
    A --- UC6

    PM([Project Manager]) --- UC6
    PM --- UC7

    AUD([Compliance Auditor]) --- UC8
    AUD --- UC9

    subgraph SYSTEM["TestAI Platform"]
        UC1["Authenticate via Google"]
        UC2["Upload Requirement Document"]
        UC3["Review Extracted Requirements"]
        UC4["Select Test Case Categories"]
        UC5["Generate AI Test Cases"]
        UC6["Export Test Cases to Jira"]
        UC7["Browse Test Coverage Report"]
        UC8["Inspect Compliance Traceability"]
        UC9["Audit PII Masking (GDPR)"]
    end

    UC2 -.->|includes| UC3
    UC3 -.->|extends| UC4
    UC4 -.->|includes| UC5
    UC5 -.->|extends| UC6
    UC5 -.->|extends| UC8

    classDef actor  fill:#4A148C,stroke:#311B92,color:#fff
    classDef uc     fill:#01579B,stroke:#013A6B,color:#fff
    classDef sys    fill:#1B5E20,stroke:#0A3D0A,color:#fff

    class A,PM,AUD actor
    class UC1,UC2,UC3,UC4,UC5,UC6,UC7,UC8,UC9 uc
```

---

## Processing Pipeline

```mermaid
sequenceDiagram
    autonumber
    actor U as User
    participant FE as Frontend
    participant API as FastAPI (Cloud Run)
    participant DOCAI as Document AI
    participant DLP as Cloud DLP
    participant RAG as Vertex AI RAG
    participant KG as Knowledge Graph
    participant GEM as Gemini 2.0 Flash

    U->>FE: Upload PDF
    FE->>API: POST /generate-ui-tests
    API->>DOCAI: Extract text and entities
    DOCAI-->>API: Chunked text + requirements + compliance entities
    API->>DLP: Mask PII across all chunks
    DLP-->>API: Masked chunks with per-chunk PII stats
    API->>RAG: Query compliance corpus per chunk
    RAG-->>API: Retrieved compliance context documents
    API->>KG: Build requirement-regulation graph
    KG-->>API: Nodes (Requirements, Regulations, Articles) + Edges
    API->>GEM: Generate test cases with full context
    GEM-->>API: Structured test cases with traceability links
    API-->>FE: Complete test suite JSON
    FE-->>U: Rendered test cases with PDF highlights
    U->>FE: Export to Jira
    FE-->>U: Issues created in selected project
```
</div>

<div>
---

## Repository Structure

```
Schneider_Hack/
├── schneider_backend/          # Python FastAPI service
│   ├── api_server_modular.py   # Entry point — all endpoints
│   ├── modules/                # Processing pipeline modules
│   │   ├── document_ai.py      # Document AI extraction
│   │   ├── dlp_masking.py      # PII detection and masking
│   │   ├── rag_enhancement.py  # Vertex AI RAG queries
│   │   ├── knowledge_graph.py  # Graph construction and analysis
│   │   ├── test_generation.py  # Gemini test case generation
│   │   └── mock_data_loader.py # Local mock data utilities
│   ├── mockData/               # Sample documents and test fixtures
│   ├── Dockerfile              # Cloud Run container definition
│   └── requirements.txt        # Python dependencies
│
└── schneider_frontend/         # Next.js 15 application
    ├── src/
    │   ├── app/                # Next.js App Router pages
    │   │   ├── a/chat/         # Primary chat + workflow interface
    │   │   ├── login/          # Authentication page
    │   │   └── api/            # Route handlers (Document AI, Gemini, Jira)
    │   ├── components/         # Shared UI components
    │   ├── hooks/              # Custom React hooks
    │   └── utils/              # API clients and helpers
    └── package.json
```

---

## Technology Stack

| Layer | Technology | Purpose |
|---|---|---|
| Frontend | Next.js 15, React 19, TypeScript | Application shell and UI |
| Styling | Sass | Component and layout styles |
| Authentication | NextAuth.js (Google OAuth) | Secure user sessions |
| Document extraction | Google Cloud Document AI | PDF text and entity parsing |
| Privacy | Google Cloud DLP | PII detection and GDPR masking |
| Retrieval augmentation | Vertex AI RAG | Compliance corpus retrieval |
| AI generation | Gemini 2.0 Flash (Vertex AI) | Test case synthesis |
| Graph engine | Custom Python graph builder | Requirement-regulation traceability |
| ALM integration | Jira Cloud REST API (OAuth 2.0) | Test case export |
| Backend runtime | FastAPI, Uvicorn | REST API server |
| Container | Docker, Google Cloud Run | Serverless deployment |
| Frontend hosting | Vercel | Edge-deployed Next.js |

---

## Supported Compliance Frameworks

- GDPR (EU General Data Protection Regulation 2016/679)
- HIPAA (Health Insurance Portability and Accountability Act 1996)
- FDA 21 CFR Part 11 (Electronic Records and Signatures)
- CCPA (California Consumer Privacy Act 2018)
- SOC 2 Type II

---

## Quick Start

### Backend

```bash
cd schneider_backend
python -m venv venv
source venv/bin/activate          # Windows: venv\Scripts\activate
pip install -r requirements.txt

# Set required environment variables
export PROJECT_ID=<your-gcp-project-id>
export LOCATION=us
export PROCESSOR_ID=<your-docai-processor-id>
export RAG_CORPUS_NAME=<your-rag-corpus-name>
export RAG_LOCATION=europe-west3

python api_server_modular.py
# Server available at http://localhost:8080
```

### Frontend

```bash
cd schneider_frontend
yarn install

# Create .env.local with the following variables:
# GOOGLE_CLIENT_ID, GOOGLE_CLIENT_SECRET
# NEXTAUTH_SECRET, NEXTAUTH_URL
# NEXT_PUBLIC_API_BASE_URL
# JIRA_CLIENT_ID, JIRA_CLIENT_SECRET, JIRA_CALLBACK_URL

yarn dev
# Application available at http://localhost:3000
```

</div>

---

## License

This project was developed as a hackathon submission. All rights reserved by the contributors.
