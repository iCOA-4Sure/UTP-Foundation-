# Why We Built UTP: Bounded Agent Authorization for Open Finance

**Published July 2026 | UTP Foundation**

---

## The Problem

When AI agents access regulated financial data, the rules break.

For decades, financial institutions built security on a simple assumption: **An API client receives a credential with a specific scope, and honors that scope.**

An API key with `read:accounts:checking` scope does exactly that—reads the checking account, nothing more. The scope is fixed. The client's actions are predictable.

But AI agents are not predictable clients.

An agent comes in with a vague intent: *"Improve my customer's credit score."* Over a long conversation, the agent's understanding of what it's allowed to do drifts. It asks for transaction history. Then savings account details. Then payroll deposits. Each request seems reasonable in isolation, but the cumulative scope is now "basically all customer financial data."

The bank sees: "This agent is asking for more data than we agreed to."
The fintech sees: "My agent just needs to do its job."
The customer sees: "Why is my bank asking me about this? I didn't consent to this."

**The architecture of API security—one credential, one fixed scope—assumes human-like behavior from the requester. Agents don't behave that way.**

---

## The Stakes

Three stakeholder groups are affected:

### For Banks & Data Providers

You need to know:
- **What did this agent actually do?** If something goes wrong, who do you blame—the fintech, their agent provider, or a downstream ML model?
- **Can you trace the data flow?** An agent might pull data, pass it to an ML model, which passes it to a human analyst. Who is accountable if PII is exposed?
- **How do you revoke access?** If you tell an agent "stop accessing this account," how do you know it actually stopped? Does it have cached data? Did it share the data elsewhere?

### For Fintechs & Data Recipients

You want to use agents because they create better customer experiences. But:
- **You carry new legal liability.** If your agent is too aggressive in requesting data, your bank partner might shut you down.
- **You don't fully control your agents.** Third-party LLM providers have policies (e.g., "refuse credential-based scraping"). Open-source agents have no policies at all. You're liable either way.
- **Regulatory uncertainty is rising.** What does "responsible agent deployment" look like? No one knows yet.

### For Standards Bodies (like FDX)

In the FDX webinar on Agentic AI (July 2026), 700+ participants identified **7 new problem spaces introduced by AI**:

1. **Consent for fluid agents** — How do you get consent when the agent's purpose is emergent?
2. **Traceable consent** — How do you prove what a human authorized when an agent made the request?
3. **Interparty risk management** — If an agent misbehaves, how do you track downstream liability?
4. **Data storage & revocation** — When data is fed to ML models, how do you ensure it's actually deleted if access is revoked?
5. **Governed access vs. scraping** — How do you make governed API access *more attractive* than credential-based scraping?
6. **Read vs. write capabilities** — Should agents have permission to modify customer data (transfers, payments)?
7. **Agent identity & accountability** — What does it mean to identify an agent, and who's responsible if it misbehaves?

These are real. And **there's no standard framework for solving them.**

---

## The Existing Approaches (and Why They Fall Short)

### Approach 1: "Just Use More OAuth Scopes"

**The idea:** Instead of `read:accounts`, use `read:accounts:checking:unauthenticated-transfers:exclude`.

**Why it doesn't work:** OAuth scopes are *coarse*. Even with 100 fine-grained scopes, you can't express agent-specific constraints:
- "Agent can read transactions, but only for accounts where the human authorized this agent"
- "Agent can read transactions for 90 days, but not before"
- "Agent can read 1000 transactions/day, no more"

Scopes don't encode *time limits*, *request rate limits*, *purpose*, or *which agent* is making the request.

### Approach 2: "Just Implement Stronger Audit Logs"

**The idea:** Log everything, so at least you know what happened.

**Why it doesn't work:** Logging is reactive. It tells you what an agent did, but it doesn't *prevent* the agent from doing it in the first place. And if the agent was supposed to have limited access but requested more, audit logs don't explain why the bank granted it. (Was it a mistake? A policy violation? A new rule not documented?)

### Approach 3: "Let Each Institution Build Their Own"

**The idea:** Every bank implements their own agent authorization framework.

**Why it doesn't work:** Fragmentation kills the ecosystem. If JPMorgan requires one authorization format and Bank of America requires another, fintech agents become incompatible. Scale becomes impossible. Costs explode (each fintech has to integrate with 10,000 banks individually).

### Approach 4: "Use Existing Standards (FIDO, mTLS, etc.)"

**The idea:** Cryptographically sign agent requests; verify the agent's certificate.

**Why it doesn't work:** Existing standards authenticate *who is making the request* (agent identity), but they don't specify *what the agent is authorized to do*. They solve "Is this really agent-123?" but not "Is agent-123 allowed to read this customer's transactions?" Identity is necessary but insufficient.

---

## What We Built: UTP

The Universal Transfer Protocol (UTP) is a **domain-agnostic framework for bounded agent authorization** that solves all 7 FDX problem spaces.

### Core Principles

**1. Bounded Scope (Not Vague Intent)**

An agent doesn't ask for "financial data." It asks for a specific action:
- `READ` — Transaction history for accounts X and Y, last 90 days
- `QUERY` — Balance check on account Z, once per day
- `TRANSFER` — Initiate payment under $1,000, to pre-approved recipients only

Each request is enumerated. Not guessed. Not assumed.

**2. Deterministic Evaluation (Not Probabilistic Fallback)**

UTP evaluates authorization through **9 independent stages**, each checking a different signal:
1. User-defined rules
2. Primary authorization signal (e.g., merchant code, service type)
3. Secondary signal (e.g., ISO purpose code)
4. Industry context
5. Metadata enrichment
6. Historical patterns
7. Confidence aggregation
8. Deterministic resolution
9. Fallback (if stages 1-7 insufficient)

If all rich signals are present, confidence is high (95%+). If signals are weak, the system doesn't force a probabilistic guess; it escalates to a human.

**Compare to: "Agent is requesting data, I'm not sure, let me use an ML model to decide."** That's a probabil probabilistic guess. UTP doesn't do that.**

**3. Full Audit Trail (Not Just Logs)**

Every authorization decision is cryptographically sealed and includes:
- What each stage found (or didn't find)
- The confidence of each finding
- Which stage drove the final decision
- Whether a user rule fired
- Whether fallback was triggered

If something goes wrong, you don't ask "What happened?" You ask "Here's what the audit record says." The record is tamper-evident and suitable for legal proceedings.

**4. Multi-Vendor Interoperability (Not Proprietary)**

UTP is open-source and domain-agnostic. Banks don't need to build their own. Fintechs don't need to integrate with each bank individually. Everyone speaks the same language.

**5. Degradation Without Failure**

If an agent request has weak signals:
- UTP doesn't deny everything
- UTP doesn't probabilistically guess
- UTP transparently degrades (e.g., "We can approve READ-ONLY, but not WRITE")
- UTP escalates for human review if truly uncertain

---

## How UTP Solves the 7 FDX Problem Spaces

### 1. Consent for Fluid Agents

**Problem:** An agent starts with intent to "improve credit score," but over time asks for more data.

**UTP Solution:**
- The agent must request *bounded actions*, not vague intents
- Each action has an expiration time and data scope
- The human reviews what the agent asked for and explicitly authorizes it
- If the agent later asks for something outside the authorized scope, the bank rejects it

The scope doesn't drift because it's bounded upfront.

### 2. Human-in-the-Loop & Traceable Consent

**Problem:** How do you prove a human authorized an agent?

**UTP Solution:**
- Every authorization request includes the `humanPrincipal` field (email of the human who said "yes")
- The UTP record includes a timestamp and signature from the human's account
- It's cryptographically sealed and suitable for regulatory audit

Traceability is built in, not bolted on.

### 3. Interparty Risk Management

**Problem:** Agent misbehaves → fintech says "Not my problem, it's the agent provider" → agent provider says "Not my problem, it's the fintech" → data is exposed, no one takes responsibility.

**UTP Solution:**
- Every authorization record identifies the *agent operator* (the company running the agent)
- Every action is logged with the agent's ID
- If the agent violates its authorized scope, the bank can prove it (using the UTP audit trail)
- Liability flows to the agent operator who exceeded the scope

No ambiguity.

### 4. Data Storage, Revocation, and Model Training

**Problem:** Raw data fed to ML models; hard to revoke; unclear if model training is still using revoked data.

**UTP Solution:**
- UTP authorization specifies what data can be used for model training (if any)
- When access is revoked, the revocation is timestamped and logged
- Any data retention beyond the revocation date violates the authorization
- Audit logs show exactly when data access ended

You can't accidentally keep training on revoked data.

### 5. Governed "Front Door" vs. Scraping

**Problem:** Credential-based scraping (agents logging into bank websites) is hard to detect and block.

**UTP Solution:**
- Governed API access (via UTP) is *easier* than scraping
- Banks can implement policies: "Agents requesting via UTP get pre-authorized scope in 1 second; agents scraping get blocked"
- UTP becomes the path of least resistance

Incentives align toward safe access.

### 6. Read vs. Write Capabilities

**Problem:** If agents start requesting WRITE access, do we need standardized write APIs?

**UTP Solution:**
- Action codes explicitly distinguish READ, WRITE, and CONSENT actions
- Banks can set policies: "Agents can have READ, but only WRITE under these conditions"
- The UTP framework scales from read-only to fully transactional

Standards-based write access becomes possible (not proprietary).

### 7. Agent Identity & Accountability

**Problem:** Is agent identity a new segment in the API, or do existing frameworks already handle it?

**UTP Solution:**
- Agent identity is built into every UTP record (agentId, agentOperator, trustLevel)
- The auditor can trace which agent made the request, who operates it, and what policies were applied
- No new credentials needed; identity flows from the authorization record

Traceability is the default.

---

## Why Open-Source?

We could have built UTP as proprietary infrastructure and monetized it.

Instead, we open-sourced it because:

1. **Standards need trust.** If UTP is proprietary, banks won't adopt it (too risky). If it's open-source, they'll adopt it, fork it, extend it, and—most importantly—trust it.

2. **Ecosystem scale requires interoperability.** If every bank implements UTP differently, the whole point fails. Open governance ensures everyone speaks the same language.

3. **Regulatory clarity requires consensus.** When regulators (SEC, CFPB, HIPAA, etc.) eventually set rules for agent authorization, they'll reference open standards, not proprietary products. We want UTP to be that standard.

4. **Our business isn't selling UTP. It's selling implementations.** We built the first implementation (in financial services accounting). Others will build implementations in healthcare, legal, travel, insurance. The open framework creates a rising tide.

---

## Who Built This?

This specification originated from iCOA Labs, which built and operates the first production implementation of UTP in (Authentify) Agent Authorization in Beta, and iCOA Pro, transaction classification in accounting software. Currently in development. 

The framework is ready for community input and pilot testing.


---

## What's Next?

### For FDX

We're submitting UTP as input to the AI Task Force (fall 2026). The specification is ready. The reference implementation is in production. The documentation is complete.

FDX can:
- Adopt UTP as-is (reference it in FDX standards)
- Modify UTP to align with FDX's vision
- Use UTP as inspiration for your own framework

We're not claiming UTP is the only solution. We're claiming it's a *working* solution, and we'd like FDX to consider it.

### For Healthcare, Legal, Travel, etc.

UTP is domain-agnostic. The financial domain profile is published. We're working with partners to define profiles for:
- Healthcare (HIPAA-compliant patient record access)
- Legal (attorney-client privilege, discovery management)
- Travel (booking, itinerary management, cancellations)
- Insurance (claims, underwriting, policy access)

Interested? Email: Rmcmillan6@ivytech.edu

### For Banks & Fintechs

We're releasing:
- **UTP Agent Authorization Gateway** — A reference implementation that any bank or fintech can deploy
- **iCOA Pro (Financial Module)** — The first production system using UTP
- **UTP Registry** — An open agent identity registry (similar to FIDO or other standards bodies)

You can adopt UTP today, not tomorrow.

---

## Closing

The era of vague agent consent is over.

Agents are too powerful, too autonomous, and too important to the future of Open Finance to leave authorization to assumption and audit logs.

We built UTP because the problem is real, the stakes are high, and the existing approaches don't work.

We open-sourced it because solving this requires an industry-wide consensus, not a proprietary moat.

And we're inviting you to join us in making agent-safe open finance the default.

---

**For questions, feedback, or to get involved:**
- GitHub: https://github.com/utp-foundation
- Email: rmcmillan6@ivytech.edu
- FDX: input@financialdataexchange.org
