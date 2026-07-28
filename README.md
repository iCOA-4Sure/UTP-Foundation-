Universal Transfer Protocol (UTP)
Open Standard for Auditable Agent Authorization in Regulated Industries

![UTP Badge](https://img.shields.io/badge/UTP-v1.0-blue) ![License: Apache 2.0](https://img.shields.io/badge/License-Apache%202.0-green) ![Status: Production](https://img.shields.io/badge/Status-Production-brightgreen) Status: Production

---

WHAT IS UTP?

The Universal Transfer Protocol (UTP) is an open-source standard for agents to request authorization for accessing regulated data, and for institutions to grant, audit, and revoke that authorization.

UTP handles THREE responsibilities:
1. AUTHORIZATION — Agents request specific actions on specific data
2. AUDIT TRAIL — Every request and decision is logged immutably
3. REVOCATION — Institutions can revoke access if agents violate scope

UTP does NOT handle:
- Runtime enforcement (blocking agent requests)
- Intercepting agent behavior
- Controlling what agents do once authorized

Instead, the data provider (bank, healthcare system, etc.) enforces scope by validating tokens and rejecting out-of-scope requests.

---

THE PROBLEM UTP SOLVES

In regulated industries—financial services, healthcare, legal, travel, insurance—data access is heavily controlled.

When an AI agent needs access to customer data, three gaps exist:

1. NO STANDARD AUTHORIZATION FRAMEWORK
   Today: Each bank, healthcare system, etc. builds its own authorization logic
   Result: No interoperability, agents can't scale across institutions
   UTP: Standard protocol all institutions recognize

2. NO IMMUTABLE AUDIT TRAIL
   Today: Institutions can't prove what agents were authorized to do
   Regulators: "How do I know the agent didn't exceed scope?"
   Institution: "Um... we have logs somewhere?"
   UTP: Cryptographically sealed audit trail of every decision

3. NO CLEAR ENFORCEMENT RESPONSIBILITY
   Today: Unclear who is responsible for blocking unauthorized actions
   Result: Scope violations aren't detected or revoked
   UTP: Clear split of responsibility (UTP authorizes, your system enforces)

---

CORE INNOVATION: AUDIT-FIRST, NOT ENFORCEMENT-FIRST

Traditional API security assumes humans make reasonable decisions:
"Grant scope. Human honors it."

AI agents are different. Their understanding can drift. UTP handles this via:

Explicit Authorization — Agents request specific actions (READ, WRITE, DELETE), not vague intents

Multi-Signal Evaluation — 9 independent stages evaluate the request (not a single probabilistic guess)

Immutable Audit Trail — Every decision sealed with timestamps, actor identity, and stage-by-stage findings

Auto-Revocation — If agent violates scope, UTP auto-revokes and logs why

Compliance Proof — Audit trail proves to regulators you authorized, logged, and revoked appropriately

---

RESPONSIBILITY SPLIT: WHO DOES WHAT

UTP Responsibilities:
- Issue scoped authorization tokens
- Log every authorization decision
- Detect when agent violates scope
- Auto-revoke agents on violation
- Provide immutable audit proof

Your System Responsibilities:
- Validate tokens before granting access
- Reject requests outside token scope
- Log violations back to UTP
- Enforce access control at your API layer

Why This Matters:
- You retain control of your data
- Clear accountability (you know what you must implement)
- Regulators understand the model (authorization + enforcement + audit)
- Scalable (UTP coordinates across institutions, you control your API)

---

REAL-WORLD EXAMPLE: FINANCIAL SERVICES

Agent requests authorization:
"I need READ access to customer transactions (90 days)"
Bank routes request to UTP

UTP evaluates request:
- Agent identity verified
- Purpose legitimate
- Customer scope valid
- Rate limit OK
Decision: APPROVED
Issues JWT token with scope ["READ"]
Logs: Authorization approved at 2026-07-27T18:00:00Z

Bank's system enforces scope:
Agent uses token to make request
Bank API validates token signature
Bank API checks: Token says READ-only
Agent requests READ → Bank allows
Agent requests DELETE → Bank rejects
Bank logs: "Agent attempted DELETE, scope is READ-only"

Violation detected:
Bank logs to UTP: "Agent attempted action outside scope"
UTP records: Violation detected at 2026-07-27T18:00:15Z
UTP auto-revokes: Agent API key now invalid

Compliance proof:
Regulator: "How did you prevent unauthorized access?"
Bank: "Here's UTP audit trail showing:
   - We authorized READ-only
   - Agent attempted DELETE
   - We rejected DELETE at application layer
   - We auto-revoked agent
   All cryptographically sealed"
Regulator: Compliant

---

CORE FEATURES

Bounded Actions
Agents request specific actions (READ, WRITE, QUERY, DELETE, CONSENT, etc.), not vague data access. Each action is explicit in the token.

Multi-Signal Evaluation
9 independent stages evaluate authorization signals in parallel:
- User-defined rules (B.A.S.E.)
- Primary signal (merchant code, service type)
- Secondary signal (ISO purpose code)
- Industry context (NAICS, facility type)
- Metadata enrichment (entity resolution)
- Historical patterns (anomaly detection)
- Confidence aggregation
- Deterministic resolution
- Fallback handler

Conflicts are resolved deterministically, not probabilistically.

Deterministic Fallback
If signals are weak or missing, the system transparently degrades:
"Approve READ-only, deny WRITE" instead of guessing.

Immutable Audit Trail
Every decision is cryptographically sealed with:
- Full request details
- Stage-by-stage findings (all 9 evaluators)
- Confidence scores
- Timestamps
- Actor identity (agent, operator, human principal)
- Enforcement outcome (approved, denied, revoked)

Suitable for regulatory proceedings.

Domain-Agnostic
Same framework works for financial services, healthcare, legal, travel, insurance, etc.

Multi-Vendor Interoperability
Banks, fintechs, aggregators, and regulators all speak the same language.

Regulatory-Ready
Designed to satisfy GLBA, HIPAA, GDPR, ABA, and other compliance frameworks.

---

QUICK START

1. Read the Specification
Start here: Full UTP Specification v1.0
Takes ~30 minutes to understand core concepts.

2. Understand the Authorization + Audit Model
UTP does NOT enforce scope at runtime.
UTP DOES authorize, log, and auto-revoke.
Your system enforces at the API layer.

3. Explore the API
Review the OpenAPI spec: Gateway API v1.0
Can be imported into Swagger UI, Postman, or any OpenAPI tool.

4. See It In Action
Financial domain example:

REQUEST:
{
  "domain": "FIN",
  "requestor": {
    "agentId": "fintech-credit-scorer-v3",
    "agentOperator": "acme-fintech@example.com"
  },
  "dataSubject": {
    "subjectId": "customer-12345",
    "subjectType": "individual"
  },
  "scope": {
    "dataElements": ["transaction_date", "amount", "merchant_category"],
    "actions": ["READ", "QUERY"],
    "duration": { "durationDays": 90 },
    "readOnly": true
  },
  "signals": {
    "signal1": "7372",
    "signal2": "SALA"
  }
}

RESPONSE:
{
  "decision": "approved",
  "signalQuality": "rich",
  "utpCode": "UTP-FIN-541219-7372-SALA-READ",
  "authorizedActions": ["READ", "QUERY"],
  "scope": "customer_transactions_90_days",
  "token": "eyJ...",
  "overallConfidence": 94,
  "enforcement": {
    "enforceStrictScope": true,
    "enforceAt": "customer_system"
  }
}

---

ARCHITECTURE

The 9-Stage Waterfall

Every authorization request flows through 9 independent evaluators:

Stage 0: B.A.S.E.    → User-defined rules (highest priority)
  ↓
Stage 1: S1E         → Primary signal (merchant code, service type)
  ↓
Stage 2: S2E         → Secondary signal (ISO purpose code)
  ↓
Stage 3: ICE         → Industry context (NAICS, facility type)
  ↓
Stage 4: MDE         → Metadata enrichment (entity resolution)
  ↓
Stage 5: HPE         → Historical patterns (anomaly detection)
  ↓
Stage 6: CA          → Confidence aggregation (weighted score)
  ↓
Stage 7: DR          → Deterministic resolution (final action code)
  ↓
Stage 8: FBH         → Fallback handler (if stages 1-7 insufficient)
  ↓
DECISION: APPROVED / DENIED / ESCALATE
AUDIT LOG: All findings sealed cryptographically

Each stage:
- Operates independently (can be swapped out)
- Returns a finding + confidence score
- Passes downstream for conflict resolution
- Is logged in final audit trail

Signal Quality Tiers

Tier        Definition                           Confidence    Approval               Use Case
RULED       User rule matched                    100%          Auto-approved          "I explicitly allow Adobe → Software"
RICH        Primary + secondary signals          90-100%       Auto-approved          Full ACH code + ISO purpose + MCC
STANDARD    One rich signal                      70-85%        Auto-approved (masked) SEC code present, ISO missing
DEGRADED    Context + fallback only              50-70%        Requires escalation    Limited signal, conservative access
MINIMUM     All signals weak/absent              30-50%        Human escalation       Unknown agent, no signals

---

DOMAIN PROFILES

UTP supports multiple regulated domains. Each domain defines:

Context Axis — Industry classification (NAICS, facility type, etc.)
Signal-1 & Signal-2 — Primary and secondary authorization signals
Action Codes — What agents can do in that domain (READ, WRITE, DELETE, etc.)
Scope Boundaries — Data limits, time limits, rate limits
Enforcement Rules — Validation, masking, escalation policies
Fallback Actions — Default behavior when uncertain

Supported Domains

FIN (Financial Services) — Production-ready
HLT (Healthcare) — In progress
LGL (Legal) — In progress
TRV (Travel) — Planned
INS (Insurance) — Planned
RET (Retail / e-Commerce) — Planned
EMP (Employment / HR) — Planned
TAX (Tax / Compliance) — Planned
EDU (Education) — Planned

Interested in defining a domain? Open an issue or contribute.

---

IMPLEMENTATIONS

Reference Implementation

UTP Agent Authorization Gateway — Production-ready REST API for evaluating authorization requests.

Language: Node.js + TypeScript
Status: Production (v1.0.0)
Repo: utp-foundation/utp-gateway
Deploy: Docker, Kubernetes, AWS Lambda

Known Implementations

iCOA Pro (Financial) — Production system using UTP in accounting classification
Authentify (Financial) — Commercial reference implementation of UTP

[Your implementation here? Open a PR!]

---

USE CASES

Financial Services

Bank needs to authorize a fintech agent requesting transaction history:

Agent: "I need 90 days of checking account transactions"
Bank (via UTP): "For what purpose?"
Agent: "Credit scoring"
Bank: Evaluates via UTP → UTP-FIN-...-READ approved @ 94% confidence
Bank: Issues JWT token with scope ["READ"], entity "transactions_90_days"
Agent: Uses token to access bank API
Bank API: Validates token, enforces READ-only scope
Agent: Accesses data
Bank: Audit log shows exactly what agent did, when, why

If agent exceeds scope:

Agent (somehow): Attempts DELETE
Bank API: Validates token, sees READ-only, rejects DELETE
Bank: Logs violation to UTP
UTP: Auto-revokes agent
Agent: All future requests denied
Bank tells regulator: "Here's proof we authorized, enforced, and revoked"

Healthcare

Care coordinator bot requesting patient labs:

Agent: "I need lab results for patient-98765"
Clinic (via UTP): "On whose authorization?"
Agent: "Nurse Alice, for treatment coordination"
Clinic: Evaluates via UTP → UTP-HLT-...-READ approved @ 78% confidence
Clinic: Issues JWT token with scope ["READ"], entity "labs_only", ttl "30_days"
Agent: Uses token to access clinic API
Clinic API: Validates token, enforces scope
Agent: Accesses labs
Clinic: Audit log satisfies HIPAA requirements

Legal

Discovery agent requesting attorney-client privileged documents:

Agent: "I need all documents marked 'privileged' for case-2026-001"
Firm (via UTP): "Is this from an authorized opposing counsel agent?"
Agent: "No, I'm internal"
Firm: Evaluates via UTP → UTP-LGL-...-DISC approved @ 85% confidence
Firm: Issues JWT token with scope ["DISCOVER"], entity "case_2026_001"
Agent: Uses token to access firm API
Firm API: Validates token, enforces scope
Agent: Accesses privileged docs
Firm: Audit log proves access was authorized and audited

---

STANDARDS ALIGNMENT

UTP can be integrated with or referenced by:

FDX (Financial Data Exchange) — Agentic AI & Open Finance standards
HIPAA — Healthcare access audit and data minimization
GDPR — Purpose limitation and data subject rights
ABA Model Rules — Attorney-client privilege and discovery
FIDO — Agent identity and verification
ISO 27001 — Information security management
SOC 2 — Audit logging and access controls

---

CONTRIBUTING

We welcome contributions:

Domain Profiles — Propose or refine profiles for new industries
Action Code Dictionaries — Help define action codes for your domain
Implementations — Build UTP in your language/platform
Feedback — Report issues, suggest improvements

See CONTRIBUTING.md for guidelines.

---

LICENSE

This project is licensed under the Apache License 2.0. You are free to:

Use UTP in commercial products
Modify UTP for your needs
Distribute UTP (with attribution)

See LICENSE for full details.

---

ROADMAP

Q3 2026 (Now)

Specification v1.0 published
Financial domain profile finalized
Gateway reference implementation in production
Input to FDX AI Task Force
Commercial implementation (Authentify) launched

Q4 2026

Healthcare & legal domain profiles published
Agent registry MVP launched
Multi-tenant policy engine
Integration with major data aggregators

Q1 2027

Travel & insurance domain profiles
Broader standards alignment (HIPAA, GDPR, ABA)
Community implementations published

---

FAQ

Is UTP a replacement for OAuth?
No. OAuth handles authentication ("Who are you?"). UTP handles authorization ("What are you allowed to do?"). They work together. UTP sits downstream of OAuth.

Can I build on UTP without open-sourcing my code?
Yes. UTP is Apache 2.0 licensed. You can build proprietary implementations. We just ask that you reference the open standard.

Does UTP prevent an agent from exceeding scope?
No. UTP doesn't prevent; it records. If an agent exceeds authorized scope, the audit trail proves the violation. Enforcement (blocking, rejecting, etc.) is your system's responsibility via token validation and scope checking at your API layer.

This is by design: you retain control, UTP provides audit proof, regulators see the full picture.

Is UTP ready for production?
Yes. The framework is in production use. Deploy the reference Gateway, implement domain profile for your industry, and integrate token validation into your API layer.

Can UTP work with open-source or untrusted agents?
Yes. UTP evaluates requests from any agent and determines appropriate authorization level. Unknown agents typically receive "degraded" signal quality and may require human escalation.

What if I want to build my own stages?
Excellent. The waterfall architecture is designed for this. You can replace S1E with your own signal evaluator, or add new stages entirely. Contribute your stage back to the community!

---

CONTACT & COMMUNITY

GitHub Discussions — Ask questions, share ideas: utp-foundation/discussions
Email — rmcmillan6@ivytech.edu
Standards Bodies — input@financialdataexchange.org (FDX)


---

CITATION

If you reference UTP in academic work, standards, or publications:

Raheem McMillan (iCOA Labs). "Universal Transfer Protocol (UTP): 
Auditable Authorization for AI Agents in Regulated Industries." 
Specification v1.0, July 2026. https://utp.io

---

ACKNOWLEDGMENTS

UTP was conceived and first implemented by iCOA Labs from inspiration from the Financial Data Exchange (FDX). Special thanks to:

FDX Community (700+ participants)
Panelists: FIDO Alliance, NIST, Stripe, JPMorgan Chase, Mastercard, Prove, Skyfire
Early adopters and testers
Open-source community feedback

---

Status: Actively Maintained | Last Updated: July 2026

About

Universal Transfer Protocol: Open standard for auditable, bounded agent authorization in regulated industries. Agents request specific access. Institutions authorize, enforce, and audit. Regulators get immutable proof.
