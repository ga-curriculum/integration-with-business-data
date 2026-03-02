# Integration with Business Data
**Session Time:** 120 minutes  
**Format:** Instructor-Led  

---

## Prerequisites

- Completion of Sessions 1–9  
- Understanding of tool categories and authentication patterns (Session 7)  
- Understanding of long-term memory types and the memory pipeline (Session 8)  
- Familiarity with privacy risks introduced by persistent memory (Session 8)  

---

## Learning Objectives

By the end of this session, students will be able to:

1. **Outline** the process of securely connecting agents to live business data sources such as CRMs, ERPs, and databases.
2. **Enable** an agent to use a persistent memory or key-value store for reading and writing data across sessions.
3. **Assess** the importance of data governance and privacy when using agents with business data, including access control, PII handling, and auditability of data access.

> Connecting agents to live business data is where agentic AI transitions from demonstration to production. It is also where the most significant risks materialize. This session is about doing it right — with security, governance, and operational reliability built in from the start.

---

## Session Breakdown

| # | Segment | Duration |
|---|---------|----------|
| 1 | Connecting Agents to Live Business Data | 5 min |
| 2 | Persistent Storage: Key-Value Stores and Read/Write Patterns | 20 min |
| 3 | Data Governance, PII, and Auditability | 20 min |
| 4 | Break | 10 min |
| 5 | Lab | 60 min |
| 6 | Wrap-Up & Reflection | 5 min |
|   | **Total** | **120 min** |

---

## 1. Connecting Agents to Live Business Data (5 min)

### From Static Knowledge to Live Data

The agents built in previous sessions operated primarily on information passed through their context window — user inputs, retrieved memory chunks, and tool outputs from external APIs. In production, agents must connect to an organization's live data infrastructure: CRM systems, ERPs, data warehouses, internal databases, and document repositories.

This distinction matters because live business data has properties that static or external data does not:

<img width="929" height="262" alt="image" src="https://git.generalassemb.ly/user-attachments/assets/a58ca788-2239-42af-b6c2-82cf59369b49" />


Connecting an agent to live business data is not a configuration task — it is an integration and governance decision that requires coordination between the AI team, data engineering, security, and legal.

### The Four Connection Patterns

Business data sources expose different interfaces. Understanding which pattern applies to a given system determines how the agent tool is configured.

<img width="943" height="449" alt="image" src="https://git.generalassemb.ly/user-attachments/assets/acdbb380-e912-4f86-a3a2-c00c1f2bc36d" />


### Choosing the Right Pattern

<img width="919" height="316" alt="image" src="https://git.generalassemb.ly/user-attachments/assets/dd80c5c8-3bd6-402f-8de2-806f47dee65d" />

Database View Pattern: Create a view exposing only the columns and rows the agent needs. Grant the service account access to the view only. The agent cannot access anything outside that scope even if the tool is misused.

---

## 2. Persistent Storage: Key-Value Stores and Read/Write Patterns (20 min)

### Beyond Retrieval: Agents That Write

Sessions 6 and 8 covered how agents retrieve and use stored information. This segment addresses the other direction: how agents write data back to persistent storage, and what architectural patterns govern that process safely.

An agent that can write to persistent storage is qualitatively different from one that can only read. Writing creates durable state changes — records updated, data logged, configurations modified — that persist beyond the session and affect future interactions. This capability is what makes agents genuinely useful in business workflows, and what makes them genuinely risky if not designed carefully.

### Key-Value Stores: The Simplest Persistent Storage

A key-value store is the most straightforward form of persistent storage for agents. Data is stored as key-value pairs — a unique key maps to a value that can be retrieved, updated, or deleted.

```
Key:   "agent:state:session_48291"
Value: {
         "customer_id": "C10482",
         "issue_type": "delivery_delay",
         "steps_completed": ["identity_verified", "order_looked_up"],
         "steps_remaining": ["escalation_triggered", "email_sent"],
         "last_updated": "2024-03-12T14:32:00Z"
       }
```

**What key-value stores are good for:**
- **Session state:** Storing the agent's progress through a multi-step workflow so it can resume after interruption
- **User preferences:** Persisting user-specific settings that should apply to all future interactions
- **Caching:** Storing frequently accessed, slowly-changing data to reduce API calls
- **Counters and flags:** Tracking simple metrics (number of escalations, rate limits, feature flags)

**Common key-value stores used in agentic systems:**

| Store | Type | Best For |
|---|---|---|
| Redis | In-memory (with persistence) | Session state, caching, real-time counters |
| DynamoDB | Managed NoSQL (AWS) | Scalable, low-latency production workloads |
| Firestore | Managed NoSQL (Google Cloud) | Real-time sync, mobile and web applications |
| Upstash | Serverless Redis | Low-overhead, edge-compatible deployments |
| LangChain Memory backends | Abstracted key-value | Agent memory via LangChain's memory interface |

### The Read/Write Lifecycle

Every persistent storage integration should define a complete read/write lifecycle — the sequence of operations the agent performs, the conditions under which each operation is triggered, and the safeguards applied at each step.

#### Read Operations

```
TRIGGER:    Session starts / new user message received
OPERATION:  Read state for current session or user
QUERY:      key = "agent:state:{session_id}" OR "agent:user:{user_id}"
RESULT:     Inject retrieved state into prompt as context
FALLBACK:   If key does not exist → initialize with default state
```

**Read safety principles:**
- Always scope reads by the authenticated user or session identifier — never read across users
- Validate the structure of retrieved data before injecting it into the prompt — malformed or tampered state objects can corrupt the agent's reasoning
- Log every read with timestamp, key accessed, and session identifier

#### Write Operations

```
TRIGGER:    Defined checkpoint in the workflow (e.g., step completed, session ending)
OPERATION:  Write updated state
KEY:        "agent:state:{session_id}"
VALUE:      Updated state object with new step completion and timestamp
VALIDATION: Confirm write was successful before proceeding
FALLBACK:   If write fails → log failure, continue session without state persistence,
            flag for manual review
```

**Write safety principles:**
- Never write to a key outside the agent's defined scope — use namespaced keys that prevent accidental cross-contamination
- Include a timestamp on every write — this enables auditing and conflict resolution
- Implement optimistic locking for concurrent write scenarios: read the current value, verify it has not changed since last read, then write — preventing race conditions in multi-agent or multi-session environments
- For destructive writes (overwriting or deleting), implement a soft-delete pattern: mark as deleted rather than physically removing, until a retention policy triggers actual deletion

### Write Patterns for Business Data

Beyond session state, agents in production workflows write to business systems: CRM records, ticketing systems, order management, financial platforms. These writes require additional controls because their effects propagate beyond the agent into the organization's operational systems.

#### Pattern 1: Write-After-Confirmation

The agent prepares the write, presents it to the user or a human reviewer for confirmation, and executes only after receiving explicit approval.

```
Agent: "I'm ready to update your shipping address to:
        123 Main St, São Paulo, SP 01310-100.
        Shall I save this to your account?"

User:  "Yes."

Agent: → crm_update(customer_id="C10482", field="shipping_address", 
                    value="123 Main St, São Paulo, SP 01310-100")
       → Confirmation sent to user
```

**Best for:** Writes that modify personally identifiable information, financial records, or any data the user would want to verify before committing.

#### Pattern 2: Write-with-Audit-Trail

The agent executes the write autonomously but logs every write operation with the reasoning that justified it, the agent's session identifier, the timestamp, and the previous value.

```
Audit log entry:
{
  "timestamp": "2024-03-12T14:35:22Z",
  "agent_session": "session_48291",
  "operation": "UPDATE",
  "table": "orders",
  "record_id": "ORD-48291",
  "field": "status",
  "previous_value": "processing",
  "new_value": "expedited",
  "reasoning": "Customer reported delivery deadline of Friday; warehouse delay confirmed; 
                policy allows expedited upgrade for warehouse-caused delays.",
  "authorized_by": "system_prompt_rule_7"
}
```

**Best for:** High-frequency operational writes where confirmation would create unacceptable friction, but auditability is required.

#### Pattern 3: Staged Write (Draft → Review → Commit)

The agent writes to a staging area first. A review step — automated or human — validates the staged write before it is committed to the production system.

```
Stage 1: Agent writes to staging table
         → orders_staging: {order_id: "48291", new_status: "expedited"}

Stage 2: Validation check
         → Does order_id exist? ✅
         → Is "expedited" a valid status transition from "processing"? ✅
         → Does this customer account have a pending dispute? ❌ (no)
         → PASS: promote to production

Stage 3: Commit to production
         → orders: {order_id: "48291", status: "expedited"}
```

**Best for:** Complex business rules where the validity of a write depends on conditions that are worth checking explicitly before committing — financial transactions, status transitions in regulated workflows, data that feeds downstream automated processes.

---

## 3. Data Governance, PII, and Auditability (20 min)

### Why Governance Is Not Optional

When an agent connects to live business data, it becomes a data processor in the legal and regulatory sense of the term. This means the organization is accountable for how the agent accesses, uses, stores, and exposes that data — regardless of whether the access was automated or human-initiated.

Three categories of obligation apply:

**Regulatory obligations:** GDPR, CCPA, HIPAA, PCI-DSS, and other regulations define specific requirements for how personal and sensitive data must be handled. Automated AI systems are not exempt.

**Contractual obligations:** Data sharing agreements, vendor contracts, and customer terms of service may restrict how data can be used. An agent that uses customer data for a purpose not covered by the original consent agreement creates legal exposure.

**Operational obligations:** Business continuity, incident response, and change management processes must account for agent behavior. An agent that modifies business data autonomously must be part of the organization's data management and incident response framework.

### Access Control: Who (and What) Can See What

Access control for agents follows the same principles as access control for humans, with one critical addition: agents can act at a scale and speed that no human can. A human employee who is granted excessive permissions may misuse them occasionally; an agent with excessive permissions can misuse them thousands of times per hour before anyone notices.

**The three layers of access control for agents:**

#### Layer 1: Authentication

Establish a verified identity for the agent that is distinct from any human user account. This means:
- Dedicated service accounts for each agent (not shared credentials)
- Credentials that are specific to the agent's deployment environment
- Audit logs that attribute every data access to the specific agent service account

Never use a human user's credentials for an agent. If the agent's credentials are compromised, revocation must not affect the human user — and the agent's access history must be separately auditable.

#### Layer 2: Authorization

Define precisely what the authenticated agent is permitted to access and modify.

**Role-based access control (RBAC):** Assign the agent a role with a defined permission set. The role grants access to specific resources (tables, API endpoints, folders) and specific operations (read, write, delete).

```
Role: customer_support_agent
Permissions:
  customers table:      READ (name, email, tier, address)
  orders table:         READ (order_id, status, items, tracking)
  interactions table:   READ + WRITE (own session records only)
  financial_records:    NO ACCESS
  admin_settings:       NO ACCESS
```

**Attribute-based access control (ABAC):** For more granular control, access decisions are based on attributes of the data being accessed — not just the resource type.

```
Rule: Customer support agent may READ customer records
      WHERE customer.active = TRUE
      AND customer.region IN agent.authorized_regions
      AND NOT customer.vip_flag = TRUE  [VIP accounts require senior agent]
```

#### Layer 3: Network and Infrastructure Controls

Even correctly authenticated and authorized requests should only be possible from approved network origins. Agents should operate within a defined network boundary — a VPN, a private cloud subnet, or an IP allowlist — so that compromised credentials cannot be used from an arbitrary location.

### PII Handling: Personal Data in the Agent Pipeline

Personal Identifiable Information (PII) — names, email addresses, phone numbers, financial data, health information, and any other data that can identify an individual — requires specific handling at every stage of the agent pipeline.

#### Identification: What Is PII in Your Context?

PII is context-dependent. An email address is PII. A customer ID is PII if it can be linked back to an individual through any other data the organization holds. A support ticket number may or may not be PII depending on whether it is linked to account information.

Before deploying an agent that touches business data, conduct a **data flow mapping**: trace every piece of data the agent accesses, processes, stores, or transmits, and classify each as PII or non-PII. This mapping is required for GDPR compliance and is good practice regardless of regulatory context.

#### Minimization: Only Handle the PII You Need

An agent that accesses a customer record should retrieve only the fields it needs for the current task — not the full record.

```
Instead of:
  SELECT * FROM customers WHERE customer_id = 'C10482'

Use:
  SELECT name, account_tier, last_interaction 
  FROM customers 
  WHERE customer_id = 'C10482'
```

This principle — data minimization — reduces exposure, reduces the attack surface if the agent is compromised, and reduces the scope of a privacy incident if data is inadvertently logged or stored.

#### Masking and Redaction in Logs

Agent execution logs are a significant PII risk. When the agent reasons about customer data, that reasoning trace — including the data itself — may be written to logs for debugging and observability purposes. Those logs must not contain raw PII.

**Masking approach:**
```
Raw agent reasoning (not safe to log):
  "Customer Maria Silva (maria.silva@email.com) has an overdue 
   balance of $340 on account C10482."

Masked log entry (safe to store):
  "Customer [MASKED_NAME] ([MASKED_EMAIL]) has an overdue 
   balance of $[MASKED_AMOUNT] on account [MASKED_ID]."
```

Masking should happen at the logging layer — the agent reasons with real data, but the log infrastructure applies masking before writing to storage. Masking at the agent level (instructing the LLM not to include PII in its reasoning trace) is unreliable — the LLM cannot be consistently instructed to self-censor.

### Auditability: The Requirement to Explain Every Decision

In any regulated environment — and increasingly in unregulated ones — organizations must be able to explain what an agent did, when it did it, and why. This is the auditability requirement, and it is one of the most important architectural constraints on production agent systems.

**What an audit trail must contain:**

| Field | Purpose |
|---|---|
| Timestamp | When the action occurred |
| Agent identity | Which agent (and version) took the action |
| Session identifier | Which interaction triggered the action |
| Action taken | What operation was performed |
| Data accessed | Which records were read or modified |
| Previous state | What the data looked like before modification |
| New state | What the data looks like after modification |
| Reasoning summary | Why the agent took this action (from the reasoning trace) |
| Human authorization | If applicable, who approved the action |

An audit trail that contains only the action taken — without the reasoning that justified it — is insufficient for regulatory purposes and useless for debugging systematic failures.

**Immutability:** Audit logs must be immutable. An agent (or a human) must not be able to modify or delete audit records. This typically requires writing logs to an append-only store — a write-once S3 bucket, an immutable database table, or a dedicated audit logging service.

### The Data Governance Framework for Agents

Bringing the three elements together — access control, PII handling, and auditability — into an operational framework:

```
BEFORE DEPLOYMENT:
  ✅ Data flow mapping completed — all PII identified
  ✅ Service account created with minimum required permissions
  ✅ Network controls configured (VPN, IP allowlist)
  ✅ PII masking implemented in logging layer
  ✅ Audit log schema defined and immutable store configured
  ✅ Retention policies defined for all stored data
  ✅ Data processing agreement reviewed (if applicable)

DURING OPERATION:
  ✅ Every data access logged with full context
  ✅ PII masked in all logs before storage
  ✅ Write operations logged with previous and new state
  ✅ Authorization errors flagged as security events (not just errors)
  ✅ Anomalous access patterns monitored and alerted

AFTER INCIDENTS:
  ✅ Audit trail available for full reconstruction of agent actions
  ✅ Data access scope of incident determinable from logs
  ✅ Affected records identifiable for notification if required by regulation
```

---<

## 4. Break (10 min)

---

## 5. Lab (60 min)

*Lab details in the Lab Guide document.*

---

## 6. Wrap-Up & Reflection (5 min)


> Next session: **Task Automation with Agents** 
---

## Resources

- [Salesforce — Connected App OAuth Authentication](https://developer.salesforce.com/docs/atlas.en-us.api_rest.meta/api_rest/intro_oauth_and_connected_apps.htm)
- [Google Cloud — Service Account Best Practices](https://cloud.google.com/iam/docs/best-practices-service-accounts)
- [AWS — IAM Best Practices for Least Privilege](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)
- [LangChain — Redis Memory Integration](https://python.langchain.com/docs/integrations/memory/redis_chat_message_history/)
- European Commission — *GDPR Article 25: Data Protection by Design and Default*
- NIST — *Privacy Framework* (NIST Privacy Framework Version 1.0)
- [OWASP — Sensitive Data Exposure](https://owasp.org/www-project-top-ten/2017/A3_2017-Sensitive_Data_Exposure)
