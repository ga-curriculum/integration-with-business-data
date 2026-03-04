# Integration with Business Data: Instructor Guide  

**Module Time:** 120 minutes  

---

## What This Module Is About

This module is where agents move from:

> Controlled lab environments  
to  
> Live production systems.

Up to this point, learners have worked with:

- Tools  
- Memory  
- Multi-agent coordination  
- Human-in-the-loop controls  

Now we introduce the most sensitive layer:

> Live business data.

This is where agentic AI transitions from demonstration to operational system.

It is also where:

- Legal risk increases  
- Security exposure increases  
- Governance becomes mandatory  
- Auditability becomes essential  

This session is about connecting agents to real data **safely**, not just successfully.

Learners should leave understanding:

- How agents connect to CRMs, spreadsheets, APIs, and databases  
- The difference between reading and writing data risk  
- Safe write patterns  
- Data governance principles  
- PII handling  
- Audit trail requirements  

---

## Core Objectives

By the end of this module, learners should be able to:

- Outline secure connection patterns for live data  
- Configure scoped read/write access  
- Implement a safe write pattern (confirm / log / stage)  
- Identify PII in agent workflows  
- Apply least privilege principles  
- Define an audit logging requirement  

---

## Must-Hit Topics

- Production vs. test data  
- Connection methods (connector, API, DB, file system)  
- Read vs write risk  
- Confirmation patterns  
- Logging and traceability  
- Access control  
- PII handling  
- Retention and deletion  
- Governance checklist  

---

# 1. Connecting Agents to Live Business Data  
**Time:** ~20 minutes  

## Purpose

Shift mindset:

> Connecting to live data is not a feature.  
> It is an operational responsibility.

---

## From Practice Data to Production Data

Explain differences:

Live business data:

- Changes continuously  
- Contains sensitive information  
- Has compliance requirements  
- Has irreversible consequences  

Ask learners:

- What would happen if your agent updated the wrong CRM record?
- Who would be responsible?

---

## Four Connection Types

---

### 1. Pre-Built Platform Connectors (Most Common)

Used in platforms such as:

- GPT Builder  
- Flowise  
- Relevance AI  
- n8n  

Examples:

- Google Sheets  
- HubSpot  
- Salesforce  
- Slack  

Best for:

- Rapid deployment  
- SaaS integrations  
- Entry-level production use  

Security focus:

- Scope access to one resource  
- Avoid full account access  
- Use dedicated agent credentials  

---

### 2. API Connections

Used for SaaS platforms without built-in connectors.

Explain:

API = structured interface for data requests.

Risk:

API keys are credentials.

Critical rule:

- Store API keys securely  
- Never embed in prompts  
- Use minimal scope  

---

### 3. Direct Database Access

Highest risk category.

Explain clearly:

Direct DB access = maximum control + maximum risk.

Rules:

- Read-only where possible  
- Access only specific tables  
- Never grant admin privileges  
- Use separate agent credentials  

Ask:

- Would you give a junior employee full database access?
- Why would you give it to an agent?

---

### 4. File & Document Systems

Examples:

- Google Drive  
- SharePoint  
- Notion  

Risk:

Unstructured data may contain sensitive documents.

Scope access to:

- Specific folder  
- Specific file  
- Specific document set  

---

## Connection Comparison Table

| Method | Ease | Risk | Control Level |
|---------|------|------|--------------|
| Pre-built connector | Easy | Medium | Moderate |
| API connection | Medium | Medium–High | Higher |
| Direct DB | Harder | High | Highest |
| File system | Medium | Medium | Moderate |

---

## Teaching Principle

> Least Privilege reduces blast radius.

Reference principle:

:contentReference[oaicite:4]{index=4} — Least Privilege guidance  

---

# 2. Reading and Writing Data Responsibly  
**Time:** ~20 minutes  

## Purpose

Clarify:

Reading ≠ Writing

Risk profiles are different.

---

## Reading Data

Relatively safer — but still risky if misconfigured.

Best practices:

- Retrieve only necessary fields  
- Scope reads to current user  
- Define fallback for missing data  
- Never expose unnecessary PII  

Ask learners:

- If the lookup fails, what should the agent do?
- Should it guess?

---

## Writing Data: Three Safety Patterns

---

### Pattern 1: Write After Confirmation (Safest)

Agent prepares change → user confirms → write occurs.

Best for:

- Personal info  
- Sensitive updates  
- High-impact changes  

Teaching phrase:

> Confirmation prevents irreversible mistakes.

---

### Pattern 2: Write With a Log (Balanced)

Agent writes autonomously but logs:

- What changed  
- When  
- Why  
- Before/after state  

Best for:

- Frequent operational updates  
- Lower-risk actions  
- Traceability requirements  

---

### Pattern 3: Draft → Stage → Commit (High Stakes)

Used for:

- Financial transactions  
- Large updates  
- High-value operations  

Flow:

1. Write to staging  
2. Automated check  
3. Human approval  
4. Commit  

Ask:

- Which pattern fits your lab agent?

---

## Write Decision Matrix

| Action Type | Recommended Pattern |
|--------------|--------------------|
| Personal data update | Confirm |
| Support note logging | Log |
| Refund below limit | Log |
| Refund above limit | Stage |
| Mass communication | Stage |

---

# 3. Data Governance, PII, and Auditability  
**Time:** ~15 minutes  

## Purpose

Move from execution to accountability.

---

## Governance Is Not Optional

Three obligations:

- Regulatory  
- Ethical  
- Business  

Reference laws:

- General Data Protection Regulation (GDPR)
- California Privacy Rights Act (CPRA)

  
Explain:

Agents are not exempt from regulation.

---

## Access Control

Best practices:

- Dedicated agent credentials  
- Read-only by default  
- Scoped resources  
- Separate agents for separate domains  

Ask:

- Should one agent access both HR and CRM?

---

## PII Handling

Define:

PII = data that identifies a person directly or indirectly.

Three design rules:

1. Access only necessary PII  
2. Avoid storing PII in logs unnecessarily  
3. Support deletion requests  

Explain deletion requirement:

Users may request erasure.

---

## Auditability

Every write must answer:

- What happened?  
- When?  
- Who (which agent)?  
- Why?  
- Before vs after state?  

Minimum viable audit log must include:

| Field | Why |
|--------|------|
| Timestamp | Sequence reconstruction |
| Agent version | Traceability |
| Action | Accountability |
| Data changed | Impact scope |
| Reasoning summary | Justification |

---

## Governance Checklist

| Requirement | Defined? |
|--------------|----------|
| Dedicated credentials | ☐ |
| Least privilege enforced | ☐ |
| PII identified | ☐ |
| Write safety pattern defined | ☐ |
| Audit logging enabled | ☐ |
| Retention window defined | ☐ |
| Deletion process defined | ☐ |
| Human oversight possible | ☐ |

Learners must apply this to their design.

---

# 4. Break  
**Time:** ~10 minutes  

Reflection prompts:

- Where is your agent over-permissioned?
- What write action worries you most?
- Who audits your system?

---

# 5. Lab Transition  
**Time:** ~60 minutes  

## Instructor Framing

> “You are not connecting data.  
> You are accepting responsibility.”

---

# Wrap-Up & Course Connection  
**Time:** ~5 minutes  

## Key Takeaways

- Live data creates real consequences  
- Least privilege limits damage  
- Reading and writing carry different risks  
- Every write must be traceable  
- PII requires explicit handling  
- Governance must be designed, not assumed  

> Production agents are not judged by intelligence.  
> They are judged by safety and accountability.

---

## Reflection Questions

- What is your highest-risk integration?
- What action should never be autonomous?
- How would you explain your audit trail to a regulator?
- Who owns data governance in your organization?

---

## Next Session

**Task Automation with Agents**

Learners will move from integration to end-to-end workflow automation using business systems and structured data.

---
