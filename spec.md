# Universal Transfer Protocol (UTP)  Specification v1.0

**Open Framework for Bounded Agent Authorization**

*This specification is open-source and domain-agnostic. It describes a deterministic, auditable framework for agents to request and receive bounded access to regulated data across financial services, healthcare, legal, and other regulated domains.*

---

## OVERVIEW

The Universal Transfer Protocol (UTP) is an open framework for **bounded agent authorization** in regulated data access scenarios. It produces a permanent, immutable, cryptographically anchored authorization record that enables:

- **Traceability:** Who authorized what action, when, and under what conditions
- **Bounded Scope:** Agents request specific actions (not vague "access"), and systems enforce those bounds
- **Deterministic Fallback:** If high-confidence signals are absent, the system deterministically degrades rather than making probabilistic guesses
- **Multi-Signal Intelligence:** A waterfall of independent signal evaluators (ACH codes, ISO standards, merchant classification, industry context, historical patterns) feeds confidence into final authorization
- **Audit-Ready:** Every decision is logged, signed, and cryptographically sealed—suitable for regulatory and legal proceedings

UTP is designed for multi-vendor ecosystems where no single entity controls the entire flow. Each participant (bank, fintech, data aggregator, agent provider) implements the UTP framework independently but speaks the same language.

---

## PART 1 — THE UTP CODE

### Structure

The UTP code is a fixed **6-segment, hyphen-delimited identity string**. All six segments are always present. Absent signals use a fixed-length null placeholder (preserving machine-parseable structure).

```
UTP - [DOMAIN] - [CONTEXT] - [SIGNAL-1] - [SIGNAL-2] - [ACTION]
```

### Segment Specification

| Pos | Segment | Purpose | Format | Length | Nullable | Null Placeholder |
|-----|---------|---------|--------|--------|----------|------------------|
| 1 | Prefix | Protocol version identifier | Fixed: "UTP" | 3 | Never | — |
| 2 | Domain | Regulated industry or data type | Alpha | 3 | No | — |
| 3 | Context | Industry-specific classification axis (e.g., sector, service type, entity type) | Alphanumeric | 6 | No | 000000 |
| 4 | Signal-1 | Primary authorization signal (e.g., merchant code, risk class, service code) | Alphanumeric | 4 | Yes | NULL |
| 5 | Signal-2 | Secondary authorization signal (e.g., purpose code, use case, compliance framework) | Alphanumeric | 4 | Yes | NULL |
| 6 | Action | Bounded action code (what the agent is authorized to do) | Alpha | 3 | Never | — |

### Domain Code Reference

| Code | Industry / Context | Example Use |
|------|-------------------|-------------|
| FIN | Financial Services | Bank data access, payments, lending |
| HLT | Healthcare | Patient record access, treatment authorization, billing |
| LGL | Legal Services | Document access, case management, discovery |
| TRV | Travel | Booking, itinerary management, cancellations |
| INS | Insurance | Claims, policy access, underwriting |
| RET | Retail / e-Commerce | Inventory, customer data, order management |
| EMP | Employment / HR | Payroll, benefits, employee records |
| TAX | Tax / Compliance | Tax return data, audit files, regulatory submissions |
| EDU | Education | Student records, enrollment, transcripts |
| GEN | Generic / Multi-domain | For experimental or cross-domain use cases |

### UTP Code Examples

**Financial — Service Revenue (full signal)**
```
UTP-FIN-541219-7372-SALA-SRV
```
(Financial domain, professional services sector, merchant code 7372, salary purpose, service revenue action)

**Healthcare — Patient Record Access (full signal)**
```
UTP-HLT-621111-NULL-TREA-READ
```
(Healthcare domain, physician practice sector, no MCC, treatment purpose, read-only action)

**Legal — Document Access (degraded signal)**
```
UTP-LGL-541110-NULL-NULL-DISC
```
(Legal domain, law office sector, no merchant/purpose signals, discovery access action)

**Travel — Booking Modification (full signal)**
```
UTP-TRV-479000-NULL-TRIP-MODIFY
```
(Travel domain, travel agency sector, no merchant code, trip management purpose, modification action)

**Generic — Fallback Only**
```
UTP-GEN-000000-NULL-NULL-OTH
```
(Generic domain, no context, no signals, fallback action)

---

## PART 2 — ACTION CODE DICTIONARY

Action Codes define **what an agent is authorized to do**. They are domain-specific but follow a consistent 3-letter naming convention. The Action Code is **never null**—the system always resolves to a bounded action.

### Action Code Categories (Universal)

Every domain defines its own Action Code set, but they map to these categories:

| Category | Purpose | Examples |
|----------|---------|----------|
| **READ** | Read-only access to data | READ, VIEW, AUDIT, QUERY |
| **WRITE** | Modify, create, or update data | WRITE, CREATE, UPDATE, DELETE |
| **CONSENT** | Grant, revoke, or modify permissions | GRANT, REVOKE, SCOPE, DELEGATE |
| **ALERT** | Monitoring, logging, or notification | ALERT, LOG, MONITOR, TRACK |
| **VERIFY** | Authentication or identity verification | VERIFY, AUTH, CHECK, VALIDATE |
| **FALLBACK** | Degraded or default action | DEFAULT, OTH, UNKNOWN |

### Financial Domain Action Codes (Example: 47 codes)

**Read Actions (8 codes)**
- `TXN` — Transaction history
- `BAL` — Account balance
- `ACT` — Account details
- `AUD` — Audit log
- `LGR` — Ledger export
- `STA` — Statement
- `POS` — Position / holdings
- `HST` — Historical analysis

**Write Actions (6 codes)**
- `XFR` — Initiate transfer
- `PAY` — Schedule payment
- `UPD` — Update account metadata
- `ACT` — Create/close account
- `SET` — Configure settings
- `APV` — Approve transaction

**Consent Actions (5 codes)**
- `GRN` — Grant access scope
- `RVK` — Revoke access scope
- `MOD` — Modify scope
- `DLG` — Delegate to sub-agent
- `ESC` — Escalate to human

**Verification/Admin (4 codes)**
- `VRY` — Verify identity
- `AUD` — Audit access
- `LOG` — Log access
- `CFG` — Configure policies

### Healthcare Domain Action Codes (Example)

**Read Actions**
- `LAB` — Lab results
- `IMG` — Imaging results
- `RX` — Prescription history
- `VIS` — Visit notes
- `INS` — Insurance/billing

**Write Actions**
- `AUTH` — Authorization for treatment
- `RX` — Write prescription
- `ORD` — Order test/imaging
- `NOTE` — Add clinical note

**Consent Actions**
- `CONS` — Consent for use/disclosure
- `ROI` — Request release of information

**Fallback**
- `OTH` — Other/unknown

---

## PART 3 — THE MULTI-AGENT WATERFALL SYSTEM

### Master Orchestration

A master orchestrator receives an authorization request and delegates it to a sequence of **independent signal evaluators**. Each evaluator examines a different signal (or multiple signals in parallel), returns a finding with confidence, and passes downstream.

The orchestrator collects all findings, **resolves conflicts deterministically**, assigns a confidence score, and issues a final authorization decision that **includes the entire audit trail**.

### The Waterfall: 9 Evaluation Stages

```
Master Orchestrator receives AuthorizationRequest
    │
    ├─ Stage 0: B.A.S.E. (Pre-classifier)
    │   └─ Check user-defined rules
    │      If match: short-circuit, return ruled signal
    │      If no match: pass through
    │
    ├─ Stage 1: Primary Signal Evaluator (S1E)
    │   └─ Evaluate domain-primary authorization signal
    │      (e.g., ACH SEC code, MCC, service classification)
    │
    ├─ Stage 2: Secondary Signal Evaluator (S2E)
    │   └─ Evaluate secondary signal
    │      (e.g., ISO purpose code, compliance framework, risk class)
    │
    ├─ Stage 3: Industry Context Evaluator (ICE)
    │   └─ Evaluate industry/sector context
    │      (e.g., NAICS, healthcare entity type)
    │
    ├─ Stage 4: Metadata Evaluator (MDE)
    │   └─ Normalize and enrich metadata
    │      (e.g., vendor name, entity classification)
    │
    ├─ Stage 5: Historical Pattern Evaluator (HPE)
    │   └─ Check historical patterns and precedents
    │      (e.g., prior authorizations, anomaly detection)
    │
    ├─ Stage 6: Confidence Aggregator (CA)
    │   └─ Weight all findings into overall confidence score
    │
    ├─ Stage 7: Deterministic Resolver (DR)
    │   └─ If high confidence: issue bounded authorization
    │      If low confidence: trigger fallback
    │
    └─ Stage 8: Fallback Handler (FBH)
        └─ If triggered: apply constrained default action
           (deterministic, not probabilistic AI)
```

### Stage Definitions (Generic)

| Stage | Name | Purpose | UTP Segment Owner | Input | Output |
|-------|------|---------|-------------------|-------|--------|
| 0 | B.A.S.E. | Pre-classifier rule engine | None | User rules + request | Rule match + action (if fired) |
| 1 | S1E | Primary signal evaluator | Signal-1 | Domain-primary signal | Signal value + confidence |
| 2 | S2E | Secondary signal evaluator | Signal-2 | Secondary signal | Signal value + confidence |
| 3 | ICE | Industry context evaluator | Context | Industry/sector codes | Context value + confidence |
| 4 | MDE | Metadata enrichment | None | Vendor name, entity info | Normalized metadata + confidence |
| 5 | HPE | Historical pattern evaluator | None | Prior decisions, history | Pattern match + confidence |
| 6 | CA | Confidence aggregator | None | All stage findings | Weighted confidence score |
| 7 | DR | Deterministic resolver | Action | Confidence + request | Final action code |
| 8 | FBH | Fallback handler | Action | Low confidence flag | Default action code |

### Key Principle: Deterministic Degradation

**If a signal is absent or weak, the system does not probabilistically guess.** Instead:

1. It transparently documents which signals were found/not found
2. It cascades to the next evaluator
3. If all rich signals fail, it applies a **deterministic fallback** (not an ML model making a probabilistic guess)
4. The fallback action is conservative (e.g., READ-ONLY, no WRITE) and logged with low confidence

This is why B.A.S.E. (user rules) and FBH (fallback) are critical: they ensure humans or explicit rules always have a say, and the system is never flying blind.

---

## PART 4 — THE FULL UTP AUTHORIZATION RECORD

Every authorization decision produces a **UTP Authorization Record** (UTAuth) that includes:

```typescript
interface UTAuth {

  // ── UTP CODE ──────────────────────────────────────────
  utpCode: string
  // Format: UTP-[DOMAIN]-[CONTEXT]-[SIGNAL-1]-[SIGNAL-2]-[ACTION]
  // Example: UTP-FIN-541219-7372-SALA-READ
  // Never null, never variable in structure

  utpVersion: "1.0"
  prefix: "UTP"
  domain: string        // FIN, HLT, LGL, TRV, etc.

  // ── MASTER ORCHESTRATOR IDENTITY ──────────────────────
  orchestratorId: string
  // Format: ORK-[DOMAIN]-[YEAR]-[INSTANCE]
  // Example: ORK-FIN-2026-001

  orchestratorVersion: string
  authorizationTimestamp: string  // ISO 8601
  
  cryptographicHash: string
  // Post-Quantum Cryptographic hash sealing entire record
  // Makes record tamper-evident and unforgeable

  // ── PRE-CLASSIFIER + STAGE BLOCKS ──────────────────────
  // Each stage owns its block. All blocks always present.

  base: {
    stageName: "B.A.S.E."
    stage: 0
    utpSegment: null
    ruleMatched: boolean
    ruleId: string | null
    ruleName: string | null
    ruleAction: {
      actionCode: string | null
    } | null
    signalFound: boolean
    confidence: 0 | 100  // Either fired (100) or didn't (0)
    executionMs: number
  }

  s1e: {
    stageName: "S1E"
    stage: 1
    utpSegment: 4      // Owns Signal-1 segment
    segmentValue: string
    signalValue: string | null
    signalName: string | null
    signalFound: boolean
    confidence: number  // 0-100
    executionMs: number
  }

  s2e: {
    stageName: "S2E"
    stage: 2
    utpSegment: 5      // Owns Signal-2 segment
    segmentValue: string
    signalValue: string | null
    signalName: string | null
    signalFound: boolean
    confidence: number
    executionMs: number
  }

  ice: {
    stageName: "ICE"
    stage: 3
    utpSegment: 3      // Owns Context segment
    contextValue: string
    contextName: string
    signalFound: boolean  // Always true — context always resolvable
    confidence: number
    executionMs: number
  }

  mde: {
    stageName: "MDE"
    stage: 4
    utpSegment: null   // No segment owned
    rawMetadata: any
    normalizedMetadata: any
    signalFound: boolean
    confidence: number
    executionMs: number
  }

  hpe: {
    stageName: "HPE"
    stage: 5
    utpSegment: null
    patternMatchFound: boolean
    matchedPattern: string | null
    priorAuthorization: any | null
    confidence: number
    executionMs: number
  }

  ca: {
    stageName: "CA"
    stage: 6
    utpSegment: null
    scoreInputs: {
      stageName: string
      confidence: number
      weight: number
    }[]
    weightedScore: number  // 0-100, final confidence
    executionMs: number
  }

  dr: {
    stageName: "DR"
    stage: 7
    utpSegment: 6      // Owns Action segment
    actionCode: string
    actionName: string
    stageResolved: number  // Which stage drove the decision
    conflictsDetected: boolean
    conflictResolution: string | null
    overallConfidence: number
    executionMs: number
  }

  fbh: {
    stageName: "FBH"
    stage: 8
    utpSegment: 6      // Co-owns Action if triggered
    fallbackTriggered: boolean
    fallbackReason: string | null
    fallbackAction: string | null
    confidence: number
    executionMs: number
  }

  // ── DECISION METADATA ──────────────────────────────────
  decision: 'approved' | 'denied' | 'escalated'
  signalQuality: 'ruled' | 'rich' | 'standard' | 'degraded'
  // ruled    = B.A.S.E. rule matched (user intent wins)
  // rich     = Primary + secondary signals found
  // standard = One rich signal, others absent
  // degraded = Only context/fallback available

  // ── REQUEST & AUTHORIZATION SCOPE ─────────────────────
  requestId: string
  requestor: {
    agentId: string
    agentVersion: string | null
    agentOperator: string
    requestorType: 'agent' | 'human' | 'system'
    humanPrincipal: string | null  // Human who authorized the agent
  }

  dataSubject: {
    subjectId: string
    subjectType: 'individual' | 'organization' | 'account'
  }

  scope: {
    dataElements: string[]     // Specific fields authorized
    actions: string[]          // Specific actions authorized
    duration: {
      startsAt: string         // ISO 8601
      expiresAt: string        // ISO 8601
    }
    requestsPerDay: number | null
    readOnly: boolean
  }

  // ── ENFORCEMENT METADATA ───────────────────────────────
  enforcement: {
    enforceStrictScope: boolean
    escalateOnViolation: boolean
    maskSensitiveFields: boolean
    requireHumanApproval: boolean
    escalationContacts: string[]
  }

  // ── AUDIT & COMPLIANCE ─────────────────────────────────
  source: 'direct' | 'fdx' | 'api' | 'internal'
  sourceTransactionId: string | null
  ingestedAt: string
  
  auditLog: {
    timestamp: string
    event: string
    actor: string
    details: any
  }[]

  // ── REGULATORY/DOMAIN-SPECIFIC ─────────────────────────
  compliance: {
    framework: string  // GLBA, HIPAA, GDPR, etc.
    framework_version: string
    certifiedCompliant: boolean
  }
}
```

---

## PART 5 — SIGNAL QUALITY TIERS

### Tier 1: RULED (B.A.S.E. match)

```
decision: approved
signalQuality: ruled
stageResolved: 0
overallConfidence: 100
base.ruleMatched: true
base.ruleName: "Adobe → SFT (Software/Subscriptions)"
```

**When:** User defined an explicit rule matching this request.
**Confidence:** 100 — user intent is deterministic.
**Scope:** Bounded by the rule definition.

### Tier 2: RICH (Primary + Secondary signals)

```
decision: approved
signalQuality: rich
stageResolved: 1
overallConfidence: 95+
s1e.signalFound: true
s2e.signalFound: true
```

**When:** Primary signal (e.g., ACH SEC code, merchant code) and secondary signal (e.g., ISO purpose) both found.
**Confidence:** 90-100 — strong multi-signal convergence.
**Scope:** Full requested scope approved.

### Tier 3: STANDARD (One rich signal)

```
decision: approved
signalQuality: standard
stageResolved: 2
overallConfidence: 75-85
s1e.signalFound: true
s2e.signalFound: false  (segmentValue: NULL)
```

**When:** One rich signal found; others missing or weak.
**Confidence:** 70-85 — reasonable but not perfect.
**Scope:** Requested scope approved, but may be masked (sensitive fields redacted).

### Tier 4: DEGRADED (Context + fallback only)

```
decision: approved
signalQuality: degraded
stageResolved: 4
overallConfidence: 50-70
s1e.signalFound: false
s2e.signalFound: false
ice.signalFound: true
fbh.fallbackTriggered: false
```

**When:** No rich signals; only context/industry classification available.
**Confidence:** 50-70 — best guess, not ideal.
**Scope:** Conservative (READ-only, no WRITE). May require escalation.

### Tier 5: MINIMUM (Fallback triggered)

```
decision: escalated
signalQuality: degraded
stageResolved: 8
overallConfidence: 30-50
fbh.fallbackTriggered: true
fbh.fallbackReason: "Insufficient signal confidence"
fbh.fallbackAction: "DEFAULT"
enforcement.requireHumanApproval: true
```

**When:** All signal evaluators returned low confidence; fallback handler triggered.
**Confidence:** 30-50 — system unsure, explicit escalation required.
**Scope:** No access granted; escalate to human.

---

## PART 6 — DOMAIN-SPECIFIC INSTANTIATION

### How to Define a Domain

Each regulated domain (FIN, HLT, LGL, etc.) publishes its own **UTP Domain Profile**:

1. **Context Axis** — What classification scheme (NAICS for finance, facility type for healthcare, etc.)
2. **Signal-1 & Signal-2** — What are the primary and secondary authorization signals
3. **Action Code Dictionary** — What actions are agents allowed to perform
4. **Scope Boundaries** — What data elements can be accessed
5. **Enforcement Rules** — What validation/masking is required
6. **Fallback Actions** — What default action for low-confidence cases

### Example: Financial Domain Profile

**Domain:** FIN
**Context Axis:** NAICS 6-digit code (industry classification)
**Signal-1:** ACH SEC code (CCD, PPD, CTX, WEB, TEL) or merchant MCC
**Signal-2:** ISO 20022 purpose code (SALA, TRAD, TAXS, etc.)
**Action Codes:** 40+ codes (READ, WRITE, TRANSFER, PAYMENT, VERIFY, etc.)
**Scope Boundaries:** Specific account IDs, date ranges, transaction types, amount limits
**Enforcement:** PCI compliance, audit logging, rate limiting
**Fallback:** READ-only transaction history, no write access, escalate for modifications

### Example: Healthcare Domain Profile

**Domain:** HLT
**Context Axis:** HIPAA entity type (covered entity, BAA, etc.) + facility type
**Signal-1:** Service type code (LAB, IMG, PHARMACY, BEHAVIORAL, etc.)
**Signal-2:** Data use purpose (TREAT, PAY, OPS, RESEARCH, etc.)
**Action Codes:** 20+ codes (READ, WRITE, AUTH, CONSENT, OTH)
**Scope Boundaries:** Specific patient ID, data types (lab, imaging, notes), sensitivity level
**Enforcement:** HIPAA audit log, minimum necessary standard, revocation on access termination
**Fallback:** No access; escalate to compliance officer

---

## PART 7 — USE CASES

### Use Case 1: Fintech Requesting Customer Bank Data via Agent

```
Requestor: Agent ID "fintech-credit-scorer-v3"
RequestorType: agent
Operator: fintech-acme@example.com
HumanPrincipal: john-doe@acme.com

DataSubject: customer-12345
Scope:
  - dataElements: [date, amount, merchant_category, balance]
  - actions: [READ, QUERY]
  - duration: 90 days
  - readOnly: true

Result:
  utpCode: UTP-FIN-541219-7372-SALA-READ
  decision: approved
  signalQuality: rich
  overallConfidence: 94
  enforcement: [strictScope, maskSensitiveFields]
```

### Use Case 2: Healthcare Agent Requesting Patient Records

```
Requestor: Agent ID "care-coordinator-bot-v1"
RequestorType: agent
Operator: clinic-compliance@healthsystem.org
HumanPrincipal: nurse-alice@healthsystem.org

DataSubject: patient-98765
Scope:
  - dataElements: [lab_results, visit_notes]
  - actions: [READ, AUTH]
  - duration: 30 days
  - readOnly: false

Result:
  utpCode: UTP-HLT-621111-NULL-TREA-READ
  decision: approved
  signalQuality: standard
  overallConfidence: 78
  enforcement: [requireHumanApproval, escalateOnViolation]
```

### Use Case 3: Insufficient Signal — Fallback

```
Requestor: Agent ID "unknown-scraper"
RequestorType: agent
Operator: unknown

DataSubject: customer-xyz
Scope:
  - dataElements: [all]
  - actions: [READ, WRITE]

Result:
  utpCode: UTP-GEN-000000-NULL-NULL-OTH
  decision: escalated
  signalQuality: degraded
  overallConfidence: 35
  fbh.fallbackTriggered: true
  fbh.fallbackReason: "Requestor not recognized; insufficient signal"
  enforcement: [requireHumanApproval=true]
```

---

## PART 8 — INTEROPERABILITY & STANDARDS

### For FDX (Financial Data Exchange)

UTP can be integrated into FDX's API specification as:

1. **Request Header:** Agent includes UTP-bounded authorization when requesting data
2. **Response Header:** API returns UTP authorization record alongside data
3. **Registry:** FDX maintains agent registry (agent ID → operator → risk profile)
4. **Compliance:** UTP audit logs satisfy FDX downstream accountability requirements

### For Other Standards Bodies

UTP is domain-agnostic and can be referenced by:

- **HIPAA** (healthcare access audit)
- **ABA** (legal ethics — agent authorization audit trail)
- **IATA/GDPR** (travel + data protection)
- **ISO 27001** (information security management)

---

## PART 9 — IMPLEMENTATION GUIDELINES

### Required Components

1. **Orchestrator Engine** — Receives requests, delegates to stages, resolves conflicts
2. **Stage Evaluators** — Each stage is independently implementable (can swap implementations)
3. **Rule Engine** (B.A.S.E.) — User-defined authorization rules
4. **Audit Logger** — Every decision logged, signed, sealed
5. **Cryptographic Signer** — Post-quantum cryptographic hash
6. **Admin Dashboard** — View/manage policies, rules, audit logs

### Optional Components

1. **Historical Pattern Evaluator** — Anomaly detection, prior authorization matching
2. **Risk Scorer** — Integration with external risk engines
3. **Metadata Enrichment** — Entity resolution, vendor normalization
4. **Multi-tenant Policy Engine** — Per-bank or per-institution policies

---

## PART 10 — OPEN GOVERNANCE

This specification is **open-source and community-driven**:

- **License:** Apache 2.0 (permissive, suitable for commercial use)
- **Contributions:** GitHub pull requests welcome
- **Domain Profiles:** Community can propose new domains (TRV, LGL, EDU, etc.)
- **Action Codes:** Domain maintainers define and publish action code dictionaries
- **Standards Alignment:** Community working groups can align UTP with HIPAA, GDPR, etc.

**Not Proprietary:** No single vendor owns UTP. iCOA published the reference implementation, but other vendors can build and extend independently.

---

## REFERENCES

- ISO 20022 (Financial Message Standards)
- ACH (Automated Clearing House) SEC Codes
- Merchant Category Codes (MCC)
- NAICS (North American Industry Classification)
- HIPAA (Health Insurance Portability and Accountability Act)
- GDPR (General Data Protection Regulation)
- FDX (Financial Data Exchange) Standards
- FIDO (Fast Identity Online) Frameworks

---

**For questions, contributions, or domain-specific extensions, visit or https://github.com/utp-foundation/** or Email: rmcmillan6@ivytech.edu
