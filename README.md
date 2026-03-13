# Integration with Business Data
**Session Time:** 120 minutes  
**Format:** Instructor-Led  

---

## Prerequisites

- Completion of Sessions 1–9  
- Understanding of tool categories and how agents connect to external systems (Session 7)  
- Familiarity with long-term memory and persistent storage concepts (Session 8)  
- Understanding of privacy risks introduced by stored data (Session 8)  

---

## Learning Objectives

By the end of this session, students will be able to:

1. **Outline** the process of securely connecting agents to live business data sources such as CRMs, spreadsheets, and databases.
2. **Enable** an agent to use a persistent memory or key-value store for reading and writing data across sessions.
3. **Assess** the importance of data governance and privacy when using agents with business data, including access control, PII handling, and auditability of data access.

> Connecting agents to live business data is where agentic AI transitions from demonstration to production. It is also where the most significant risks materialize. This session is about doing it right — with security, governance, and operational reliability built in from the start.

---

## Session Breakdown

| # | Segment | Duration |
|---|---------|----------|
| 1 | Connecting Agents to Live Business Data | 20 min |
| 2 | Reading and Writing Data Responsibly | 20 min |
| 3 | Data Governance, PII, and Auditability | 15 min |
| 4 | Break | 10 min |
| 5 | Lab | 60 min |
| 6 | Wrap-Up & Reflection | 5 min |
|   | **Total** | **120 min** |

---

## 1. Connecting Agents to Live Business Data (20 min)

### From Practice Data to Real Data

In previous sessions, agents worked with information passed through their context window — user inputs, memory, and tool responses. In real deployments, agents need to connect to an organization's actual data: CRM systems, spreadsheets, order management platforms, internal databases, and document repositories.

This is a significant step up in complexity — and in responsibility. Live business data has properties that test data does not:

<img width="919" height="257" alt="image" src="https://git.generalassemb.ly/user-attachments/assets/b60b803f-8ff0-43a5-87d1-491a19c1eec6" />


Connecting an agent to live business data is not just a technical task. It is an integration and governance decision that requires coordination between the people building the agent, the data owners, and often legal and security teams.

---

### How Agents Connect to Business Data

In practice, there are four main ways an agent accesses live business data. You don't need to implement all of these — but understanding them helps you choose the right approach for each situation.

<img width="928" height="432" alt="image" src="https://git.generalassemb.ly/user-attachments/assets/1bad02c3-75f0-4487-923c-82ef1d220f74" />

---

### Choosing the Right Connection

<img width="924" height="271" alt="image" src="https://git.generalassemb.ly/user-attachments/assets/486521bc-a24f-4149-a260-f73ccd2a660b" />


**The universal rule across all four types:** Give the agent access to the minimum it needs to do its job — no more. This principle is called the **least privilege principle**, and it is the single most effective way to limit the damage if something goes wrong.

---

### Discussion Prompt (2 min)

> Think about a tool you use daily at work — a CRM, a spreadsheet, a project management platform. If an agent needed to help you with tasks in that tool, what would it need to read? Would it ever need to write? What would you NOT want it to access?

---

## 2. Reading and Writing Data Responsibly (20 min)

### The Difference Between Reading and Writing

<img width="935" height="205" alt="image" src="https://git.generalassemb.ly/user-attachments/assets/e1a8bc9c-b1fa-46a2-8a3d-7bfe4930d6de" />

An agent that reads data is relatively safe — if it retrieves wrong information, the worst outcome is usually a bad recommendation. An agent that writes data is a different story. A wrong update to a customer record, a mistaken email send, or an accidental deletion can have real consequences that persist long after the session ends.

Before giving an agent write access to any system, ask three questions:

1. **What exactly is it allowed to change?** (Define scope)
2. **Up to what level of impact can it act alone?** (Define authority)
3. **What must a human approve first?** (Define escalation triggers)

---

### Reading Data: What to Get Right

Reading sounds simple, but there are important practices that prevent errors:

<img width="924" height="226" alt="image" src="https://git.generalassemb.ly/user-attachments/assets/ac6fccf0-44b2-4db2-b640-3ac800136fce" />


```
Agent looks up order #48291 in Google Sheets → not found

Without a fallback:
  Agent makes up an answer, or crashes silently

With a fallback defined:
  Agent responds: "I wasn't able to find order #48291 in the system.
  Could you double-check the order number, or would you like me to 
  escalate this to a team member?"
```

---

### Writing Data: Three Patterns for Safety

When an agent needs to write to a business system, the pattern you choose determines how much risk you're accepting. There are three practical approaches, each appropriate for different situations.

---

#### Pattern 1: Write After Confirmation (Safest)

The agent prepares what it wants to write, shows it to the user, and waits for approval before saving anything.

```
Agent: "I'm ready to update your shipping address to:
        Rua Augusta 1200, São Paulo, SP 01305-000.
        Shall I save this to your account?"

User: "Yes."

Agent: → Updates the record
       → Confirms: "Done — your address has been updated."
```

**When to use it:** Any time the agent is modifying personal information, financial records, or anything the user would want to verify before committing. Slower, but significantly safer.

---

#### Pattern 2: Write With a Log (Balanced)

The agent writes autonomously but records every change — what was changed, what it was before, and why the agent made that decision.

```
Agent updates order status from "processing" to "expedited":

Log entry saved automatically:
  - Time: March 12, 2:35 PM
  - Agent action: Updated order #48291 status
  - Previous value: "processing"
  - New value: "expedited"
  - Reason: Customer reported delivery deadline; warehouse delay confirmed;
            policy allows expedited upgrade in this case
```

The agent acts without friction — but every action is traceable. A human can review the log at any time and understand exactly what happened and why.

**When to use it:** High-frequency operational updates where confirmation would be too slow, but accountability is still required.

---

#### Pattern 3: Draft First, Then Commit (For High-Stakes Writes)

The agent prepares the change but writes it to a draft or staging area first. A review step — automated or human — validates it before it goes into the live system.

```
Agent prepares a refund for $740:

Step 1: Writes to draft area → "Pending: refund of $740 for order #48291"
Step 2: Automated check → Is $740 within the agent's authority? NO (limit is $500)
Step 3: Escalation → Human reviewer receives notification and approves or rejects
Step 4: If approved → Refund is processed in the live system
```

**When to use it:** Complex or high-value operations where mistakes are hard to reverse — financial transactions, large-scale updates, anything that affects multiple downstream systems.

---

### The Write Decision Matrix

<img width="923" height="328" alt="image" src="https://git.generalassemb.ly/user-attachments/assets/05ef1579-eb8d-4bdb-b2ef-a4a0cabbe563" />


---

## 3. Data Governance, PII, and Auditability (15 min)

### Why Governance Is Not Optional

When an agent connects to live business data, it becomes responsible for how that data is accessed, used, and protected — in the same way a human employee would be. The fact that the actions are automated doesn't reduce the responsibility; in some ways, it increases it, because an agent can act at a scale and speed that no human can.

Three types of obligation apply:

<img width="929" height="278" alt="image" src="https://git.generalassemb.ly/user-attachments/assets/17d17d16-32bb-427f-ae09-fd2a2041e5cb" />

---

### Access Control: The Minimum Necessary Rule

The most effective single principle for data governance in agentic systems is also the simplest: **give the agent access only to what it needs to do its job**.

In practice, this means:

- **Separate credentials for the agent.** The agent should have its own login or API key — not a shared account, not a human user's credentials. If the agent's credentials are compromised, you can revoke them without affecting anyone else.

- **Read-only where possible.** If the agent only needs to retrieve data, configure it with read-only access. Don't add write access "just in case."

- **Scoped to specific resources.** An agent helping with customer support should access customer records — not HR data, not financial reports, not engineering systems. Define this scope explicitly at setup time.

- **Separate agents for separate domains.** If one agent needs access to customer data and another needs access to employee data, keep them separate. Don't build a single super-agent with access to everything.

---

### Handling PII: Personal Data in the Agent Pipeline

Personal Identifiable Information (PII) — names, email addresses, phone numbers, financial data, health information — requires specific care at every stage of the agent's workflow.

**What counts as PII?**  
Any data that can identify a specific person, directly or indirectly. An email address is PII. A customer ID is PII if it can be linked back to an individual. Even seemingly anonymous data can become PII when combined with other data the organization holds.

**Three practices to get right:**

<img width="931" height="203" alt="image" src="https://git.generalassemb.ly/user-attachments/assets/6883bd53-1d4c-45c6-9923-0e9e28be9d3c" />

---

### Auditability: Being Able to Explain What the Agent Did

In any professional context — and especially in regulated industries — you need to be able to answer questions like: "What did the agent do?" "When?" "Why?" "What data did it access?"

This is the **auditability requirement**, and it shapes how agent systems must be built.

**What an audit trail needs to capture:**

| What to record | Why it matters |
|---|---|
| Timestamp | When did the action happen? |
| Which agent (and version) | Which system took the action? |
| What action was taken | What operation was performed? |
| What data was accessed or changed | What was the scope of the action? |
| What it looked like before and after | Can the change be understood and reversed? |
| Why the agent took this action | Was the decision justified? |

An audit log that only records *what* happened — without the *why* — is not useful for understanding failures or demonstrating compliance.

**The practical minimum:** At the start, make sure every data write your agent performs is logged somewhere — even if it's just a simple spreadsheet or a log file. A simple log that exists is far more useful than a sophisticated system that was never implemented.

---

### The Data Governance Checklist

Before deploying an agent that connects to live business data:

<img width="696" height="421" alt="image" src="https://git.generalassemb.ly/user-attachments/assets/b68d7fd4-6004-4e4a-adda-ee176b8a4e77" />

---

## 4. Break (10 min)

---

## 5. Lab (60 min)

https://pages.git.generalassemb.ly/Build-AI-Agents/Integration-With-Business-Data/

---

## 6. Wrap-Up & Reflection (5 min)

### Key Takeaways
- Live Data Means Real Consequences — Design Accordingly
- Least Privilege Is the Most Effective Safety Control
- Every Write Should Be Explainable

---

## Resources

- [Google Workspace — Connected Apps and Access Control](https://support.google.com/a/answer/7281227)
- [HubSpot — Private App Authentication](https://developers.hubspot.com/docs/api/private-apps)
- [n8n — Credential Management Best Practices](https://docs.n8n.io/credentials/)
- [Relevance AI — Data Source Connections](https://relevanceai.com/docs)
- European Commission — *GDPR Article 25: Data Protection by Design and Default*
- LGPD — *Lei Geral de Proteção de Dados Pessoais* (Brazil, Law 13.709/2018)
- [OWASP — Sensitive Data Exposure](https://owasp.org/www-project-top-ten/2017/A3_2017-Sensitive_Data_Exposure)
- NIST — *Principle of Least Privilege* (SP 800-53)

---

*General Assembly · Build AI Agents · BAA 16.0 · © 2026*
