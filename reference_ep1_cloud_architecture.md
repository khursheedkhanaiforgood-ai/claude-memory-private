---
name: EP1 Cloud Architecture & Security — Official White Paper
description: Official Extreme Networks white paper confirming EP1 microservices, Kubernetes, GDC/RDC model, shift-left anonymization, AI Core 3-component architecture, HITL-default with RBAC-scoped AI. T1 source.
type: reference
originSessionId: 6edeb609-0aad-4457-8d76-2accad2dd249
---
## Source
`/Users/khukhan/Downloads/flipbook-with-notes.pdf` — Extreme Networks official, ©2025, 13 pp. Read May 19 2026.
Also added to ep1-deployment-guide.html RAG corpus as T1 source (v2.6).

## Infrastructure Architecture (Confirmed)

### Two-Tier Data Center Model
- **GDC (Global Data Center)**: AWS-only. Hosts authentication, device redirection, AI Core (LLM, Orchestrator, IAM, Data Hub).
- **RDC (Regional Data Center)**: One per geography. Hosted on AWS / Azure / GCP. Customer VIQ lives here. Device data processed and anonymized here before going to GDC.
- **VIQ (Virtual IQ)**: Per-organization multi-tenancy unit, scoped to an RDC.

### Platform Architecture (Confirmed)
- **"Extreme Platform ONE is a microservice-driven SaaS application that extensively uses container-based solutions"** — official text
- Container orchestration: **Kubernetes** (confirmed in Shared Responsibility Model section)
- CI/CD: multi-stage change control, staging → UAT → production maintenance windows
- Cloud providers (sub-processors): AWS (dominant), Google GCP, Microsoft Azure

### Key Architectural Properties
- **EP1 is NOT in the data path** — network operates independently if EP1 goes down
- **Shift-left anonymization**: device telemetry anonymized INSIDE RDC before leaving region — AI Core sees only anonymized data
- Data residency: data never moved out of organization's region if prohibited by law
- Encryption: CAPWAP=DTLS, HTTPS=TLS 1.2/1.3, AES-256 at rest (IaaS-managed keys; customers cannot manage own keys)
- No raw packet captures stored
- Backup: daily, 30-day retention, replicated between RDCs. Cannot restore individual records — only entire RDC.

## AI Core Architecture — Three Components

### 1. Orchestration Layer
- Safety guardrails: prompt injection protection, hallucination mitigation, jailbreak blocking, bias monitoring, copyright attribution
- Centralized AI model catalog and registration

### 2. Services Layer
- Multi-agent interaction + complex workflow engine
- Model types: LLMs + SLMs + traditional ML models ("Model as a Service")
- Agent communication infrastructure: versioning, lifecycle, change management, QA, security
- Real-time + batch inference

### 3. Data Hub
- **Data Ingestion Service**: structured + unstructured from databases, APIs, file systems
- **Data Lake**: scalable, real-time + batch processing
- **Knowledge Graph**: complex entity relationships (technical, operational, business, content metadata)
- **Metadata Management**: data lineage, ownership, purpose, usage, classification
- **RAG as a Service**: named infrastructure component (not a feature bolt-on)
- **Auto Compliance**: proactive governance monitoring

## AI Governance Architecture

### HITL is Default
- "The default level of agency is set with a Human-In-The-Loop (HITL) configuration"
- AI acts with **same RBAC as the initiating user** — cannot exceed operator permissions
- All AI activity logged with same access controls as user who initiated it

### Data Flow: Device → User
```
Extreme Device → [Anonymization in RDC] → Data Hub (GDC) → AI Service → Orchestrator → IAM → User
                                            ↑
                                   AI/Data Governance Service
                                   LLM (cloud, GDC)
```

### AI Expert Privacy
- Queries to AI Expert visible ONLY to employee who initiated them
- Can only be shared explicitly by that employee

## PII Data Collected (Per Appendix)
DevOps/Engineering access: MAC, IP, hostname, radio attributes, AP association, roaming history, VLAN, SSID, wireless stats, error rates, app usage. If 802.1x: username. If PPSK: PPSK username/email. Guest CWP: phone/email if required.
GTAC: no access unless customer shares. AWS/Azure/GCP: no access. Vendors: no access.

## Shared Responsibility
**Extreme**: OS, containers, Kubernetes, storage, DR, SLA, security patches, encryption
**Customer**: Device configs, firmware updates, VIQ backup, credentials, internet connectivity/firewall for CAPWAP

## API Policy
"API-driven architecture." Advance notice before deprecation. Backward compat maintained during migration window.
