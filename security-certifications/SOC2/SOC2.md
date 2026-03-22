# SOC 2 Compliance — Complete Guide for Beginners

> **Author's Note:** This document explains SOC 2 compliance from the ground up. No prior security knowledge is needed. By the end, you'll understand what it is, why it matters, and how to get it.

---

## Table of Contents

1. [What is SOC 2?](#1-what-is-soc-2)
2. [SOC 1 vs SOC 2 vs SOC 3 — What's the Difference?](#2-soc-1-vs-soc-2-vs-soc-3)
3. [The 5 Trust Service Criteria (TSC)](#3-the-5-trust-service-criteria-tsc)
4. [SOC 2 Type 1 vs Type 2](#4-soc-2-type-1-vs-type-2)
5. [Why is SOC 2 Needed?](#5-why-is-soc-2-needed)
6. [What Value Does SOC 2 Add?](#6-what-value-does-soc-2-add)
7. [Who Needs SOC 2?](#7-who-needs-soc-2)
8. [How to Get SOC 2 — Step-by-Step](#8-how-to-get-soc-2--step-by-step)
9. [Key Controls and Requirements](#9-key-controls-and-requirements)
10. [Timeline and Cost](#10-timeline-and-cost)
11. [Tools That Help](#11-tools-that-help)
12. [Quick Reference Cheatsheet](#12-quick-reference-cheatsheet)

---

## 1. What is SOC 2?

**SOC 2** stands for **System and Organization Controls 2**.

It is a **security framework** created by the **AICPA** (American Institute of Certified Public Accountants) that defines standards for how technology companies should protect customer data.

Think of it like this:

> Imagine you're a business handing over sensitive customer data to a software vendor. How do you know the vendor is actually keeping that data safe? SOC 2 answers that question — it's an independent audit that confirms the vendor has the right security controls in place.

### Key Facts

- Created by: **AICPA** (American Institute of Certified Public Accountants)
- Introduced: **2010**
- Report type: **Attestation** (not a certification — explained below)
- Who issues it: **Licensed CPA firms only**
- It is **voluntary** — no law requires it, but customers often demand it

### Attestation vs. Certification — What's the Difference?

Most people say "SOC 2 certified" but technically it's an **attestation**:

| Certification | Attestation |
|---------------|-------------|
| Fixed pass/fail requirements | Flexible — you define what's in scope |
| Universal standards | Tailored to your specific business |
| Example: ISO 27001 | Example: SOC 2 |

With SOC 2, a CPA firm audits your security controls and issues an **opinion letter** — a professional statement saying your controls are designed well (and/or working effectively). This is more flexible, but equally trusted by the market.

---

## 2. SOC 1 vs SOC 2 vs SOC 3

The AICPA created three SOC report types. They are **not levels** of difficulty — they serve different purposes.

| | SOC 1 | SOC 2 | SOC 3 |
|--|-------|-------|-------|
| **What it covers** | Controls that affect customers' **financial statements** | Controls around **data security, privacy, and operations** | Same as SOC 2, but summarized for public sharing |
| **Who uses it** | Payroll processors, financial data handlers | SaaS companies, cloud providers, tech vendors | Same as SOC 2 users who want public proof |
| **Audience** | Customer's financial auditors | Customers, prospects (shared under NDA) | General public (posted on website) |
| **Is it confidential?** | Yes | Yes | No — it's public |
| **Has Type 1/Type 2?** | Yes | Yes | No |

**In simple terms:**
- **SOC 1** = "Our controls won't mess up your financial reporting"
- **SOC 2** = "We keep your data safe and our systems secure"
- **SOC 3** = "Here's a public summary of our SOC 2 audit"

Most software companies pursuing security compliance focus on **SOC 2**.

---

## 3. The 5 Trust Service Criteria (TSC)

SOC 2 audits are based on 5 categories called **Trust Service Criteria (TSC)**. These define what areas the auditor will evaluate.

### Only Security is Required. The rest are optional.

---

### Category 1: Security (REQUIRED)

**Also called:** "Common Criteria" (CC1–CC9)

**What it means:** Is your system protected from unauthorized access and attacks?

This covers 9 sub-areas:

| Code | Area | Plain English |
|------|------|---------------|
| CC1 | Control Environment | Does your company have a security-first culture and clear governance? |
| CC2 | Communication & Information | Does security information flow properly across your organization? |
| CC3 | Risk Assessment | Do you formally identify and manage risks? |
| CC4 | Monitoring Activities | Do you continuously check that your controls are working? |
| CC5 | Control Activities | Do you have policies and procedures to reduce risk? |
| CC6 | Logical & Physical Access | Who can access your systems and data? Is it restricted properly? |
| CC7 | System Operations | Do you monitor system events, logs, and incidents? |
| CC8 | Change Management | When you change code or systems, is it controlled and reviewed? |
| CC9 | Risk Mitigation | Do you manage vendor risks and have backup plans? |

**Real examples of controls:** Multi-factor authentication (MFA), firewalls, encryption, access reviews, penetration testing, security training.

---

### Category 2: Availability (Optional)

**What it means:** Is your system available when customers need it?

This doesn't mandate a specific uptime — it checks whether you have plans and controls to meet *your own uptime commitments*.

**Examples:** Disaster recovery plans, system backups, infrastructure redundancy, performance monitoring.

**Who typically includes this:** SaaS companies with uptime SLAs.

---

### Category 3: Processing Integrity (Optional)

**What it means:** Does your system process data completely, accurately, and on time?

**Examples:** Input validation, transaction logs, data reconciliation, error handling.

**Who typically includes this:** Companies processing financial transactions, payroll, or any service where data accuracy is critical.

---

### Category 4: Confidentiality (Optional)

**What it means:** Is confidential business information (trade secrets, contracts, business data) protected from exposure?

**Examples:** Data classification policies, access restrictions, NDAs, encryption.

**Who typically includes this:** B2B companies handling sensitive client documents, legal tech, financial services.

---

### Category 5: Privacy (Optional)

**What it means:** Is personal information (PII) collected, used, and stored properly according to your privacy policy?

This aligns closely with GDPR and CCPA requirements.

**Examples:** Privacy notices, consent management, data deletion procedures, breach notification.

**Who typically includes this:** Consumer apps, healthcare tech, e-commerce, any company subject to GDPR/CCPA.

---

## 4. SOC 2 Type 1 vs Type 2

Within SOC 2, there are two "types" of reports. Think of them as shallow vs. deep.

| | SOC 2 Type 1 | SOC 2 Type 2 |
|--|-------------|-------------|
| **What it checks** | Are your controls *designed* correctly? | Are your controls *actually working* consistently? |
| **Time scope** | A single point in time | 3–12 months of observation |
| **Depth** | Shallow — confirms controls exist | Deep — confirms controls work day-to-day |
| **Timeline** | 2–5 months total | 8–15 months total |
| **Typical cost** | $15,000–$30,000 | $30,000–$100,000 |
| **Best for** | Startups needing quick proof | Companies selling to enterprises |
| **Market acceptance** | Good for early deals | Required by most enterprise customers |

### Which Should You Get?

**Common path:**
1. Start with **Type 1** — proves you have controls in place, unblocks initial enterprise deals
2. Transition to **Type 2** — proves controls work over time, required for most large enterprise customers

Some mature companies skip Type 1 and go directly to Type 2.

---

## 5. Why is SOC 2 Needed?

### The Business Reality

SOC 2 is voluntary by law — but in practice, it's become **functionally mandatory** for B2B software companies.

Here's why:

**1. Customers are asking for it**
> 85% of enterprise buyers require a SOC 2 report before signing contracts.
> 70% of deals are delayed or lost due to non-compliance.

When a large company evaluates a software vendor, one of the first things their security team asks for is: "Do you have a SOC 2 report?" Without one, you're often eliminated from the process entirely.

**2. Enterprise procurement requires it**
Large enterprises in finance, healthcare, legal, and government require vendors to pass security reviews. SOC 2 is the standard document that satisfies these reviews.

**3. Investors expect it**
About 70% of venture capital investors prefer SOC 2-compliant startups as portfolio companies — they see it as a sign of operational maturity.

**4. It reduces real security risks**
Data breaches cost organizations an average of **$4.9 million in 2024** (IBM report). SOC 2's required controls directly reduce the probability of breaches, unauthorized access, and data leaks.

**5. Regulatory overlap**
SOC 2 controls overlap significantly with HIPAA (healthcare), GDPR (European privacy), CCPA (California privacy), and PCI DSS (payment cards). Achieving SOC 2 gives you a head start on these regulations.

---

## 6. What Value Does SOC 2 Add?

### For Business Growth

- **Unlocks enterprise sales** — "SOC 2 compliant" removes a blocker in the sales process for large customers
- **Shortens sales cycles** — instead of filling out 200-question security questionnaires, you share one document
- **78% of enterprise clients** require SOC 2 from service providers
- One documented example: A cloud provider secured several enterprise clients after achieving SOC 2 Type 2, generating **$20 million in additional revenue** in a single year

### For Customer Trust

- Independent, third-party validation is far more credible than self-attestation ("we take security seriously")
- Demonstrates ongoing commitment, not just a one-time check
- Reduces customer churn caused by security concerns

### For Internal Maturity

- Forces your team to document policies, implement proper access controls, and build security culture
- Creates a foundation to pursue other frameworks (ISO 27001, HIPAA, PCI DSS, FedRAMP) with less duplication
- Establishes repeatable processes for incident response, change management, and risk assessment

### For Legal Protection

- Demonstrates due diligence in the event of a data breach — reduces liability
- Helps meet contractual obligations with customers who have security requirements in their contracts
- Supports GDPR and CCPA compliance through overlapping controls

---

## 7. Who Needs SOC 2?

### You Likely Need SOC 2 If:

- You're a **SaaS company** storing, processing, or transmitting customer data
- You're a **cloud infrastructure or hosting provider**
- You're a **managed service provider (MSP)**
- You're selling to **enterprise customers** in any industry
- You're in **healthcare tech, fintech, legal tech, or HR tech**
- Customers are including it in **RFPs and vendor questionnaires**
- You're raising **venture capital** funding
- You're pursuing **government contracts**

### Signs You Need SOC 2 Right Now

- A customer has asked for your SOC 2 report
- A deal has stalled pending a security review
- A prospect has sent you a security questionnaire
- A competitor has SOC 2 and is winning deals because of it

### You Probably Don't Need SOC 2 If:

- You're a pure consumer app with no B2B component
- You're a very early-stage startup with no enterprise customers yet (though pursuing it early pays off later)
- Your industry uses SOC 1 instead (e.g., you're primarily a financial data processor)

---

## 8. How to Get SOC 2 — Step-by-Step

Getting SOC 2 is a structured process with four main phases.

---

### Phase 1: Preparation (2–9 months)

**Step 1 — Define Your Scope**

Determine which systems, infrastructure, and data flows will be included in the audit. This includes:
- Systems that collect, store, or process customer data
- Cloud servers and networks supporting those systems
- Third-party tools and vendors with access to in-scope systems
- People who have access to in-scope systems

**Step 2 — Choose Your Trust Service Criteria**

Security (CC1–CC9) is always required. Decide whether to add Availability, Processing Integrity, Confidentiality, and/or Privacy based on your product and customer obligations.

> Tip: Start with just Security for your first SOC 2. Adding more criteria increases scope and cost.

**Step 3 — Run a Gap Analysis**

Compare your current controls against the TSC requirements. Identify what's missing, undocumented, or inconsistently followed. This gives you a remediation roadmap.

**Step 4 — Implement and Document Controls**

Fix the gaps. This involves:
- Writing security policies (incident response, acceptable use, access control, etc.)
- Configuring technical controls (MFA, encryption, logging, vulnerability scanning)
- Setting up access reviews and employee offboarding procedures
- Establishing change management and code review processes
- Setting up security monitoring and alerting

**Step 5 — Collect Evidence**

Build a library of evidence proving your controls exist and are followed:
- Access logs and audit trails
- Signed policy documents
- Employee training completion records
- Vulnerability scan results
- Vendor assessment records

**Step 6 — Train Employees**

All staff must understand their security responsibilities, how to report incidents, and how to recognize phishing attempts.

---

### Phase 2: Engage an Auditor

**Step 7 — Select a CPA Firm**

Only a **licensed CPA firm** can issue a valid SOC 2 report. When choosing:
- Confirm they specialize in SOC 2 (not all CPA firms do)
- Look for experience in your industry
- Ask for sample reports and client references
- Compare pricing and timelines
- Consider whether they're familiar with your technology stack

**Step 8 — Optional: Readiness Review**

Many auditors offer a pre-audit dry run. This catches remaining gaps before the official audit begins and reduces the risk of audit findings.

---

### Phase 3: The Audit

**For Type 1:**
The auditor reviews your documentation, interviews your team, and inspects configurations — all at a single point in time.

**For Type 2:**
**Step 9 — Observation Period (3–12 months)**
The auditor observes your controls operating in real-time over the agreed period. Controls must be consistently followed — no exceptions.

**Step 10 — Fieldwork**
The auditor tests controls by reviewing logs, sampling transactions, interviewing personnel, and verifying evidence.

---

### Phase 4: Report and Maintain

**Step 11 — Management Response**
If the auditor finds issues (called "exceptions" or "deficiencies"), you provide a management response explaining the issue and your remediation plan.

**Step 12 — Final Report**
The auditor delivers the final SOC 2 report, which includes:
- Description of your system (written by you)
- Auditor's opinion on control design (Type 1) or design + effectiveness (Type 2)
- List of controls tested and findings

**Step 13 — Share the Report**
The SOC 2 report is **confidential** — it's shared with customers and prospects under NDA, not published publicly. (SOC 3 is the public version.)

**Step 14 — Annual Renewal**
SOC 2 Type 2 must be renewed every year. Compliance is an ongoing process, not a one-time event.

---

## 9. Key Controls and Requirements

These are the most common controls you'll need to implement:

### Access Control
- Multi-factor authentication (MFA) on all systems
- Role-based access control (RBAC) — least privilege principle
- Formal process for adding and removing user access
- Quarterly access reviews to verify who still needs access
- Privileged access management for admin accounts

### Encryption
- Data encrypted at rest (AES-256 or equivalent)
- Data encrypted in transit (TLS 1.2 or higher)
- Encryption key management

### Vulnerability Management
- Regular vulnerability scanning (minimum quarterly)
- Annual penetration testing
- Patch management with defined remediation timelines
- Secure software development practices

### Incident Response
- Documented incident response plan with defined roles
- Procedure for notifying affected customers
- Post-incident review process

### Change Management
- Code review and approval before deployment
- Separate development, staging, and production environments
- Rollback procedures

### Risk Management
- Annual risk assessment with a documented risk register
- Vendor/third-party risk assessments

### Required Policy Documents
- Information Security Policy
- Acceptable Use Policy
- Access Control Policy
- Incident Response Policy
- Business Continuity / Disaster Recovery Policy
- Vendor Management Policy
- Data Classification and Retention Policy
- Password Policy

### Employee Security
- Security awareness training at onboarding + annually
- Background checks for employees with access to sensitive systems

---

## 10. Timeline and Cost

### Timeline

| Stage | SOC 2 Type 1 | SOC 2 Type 2 |
|-------|-------------|-------------|
| Gap analysis + remediation | 1–3 months | 3–6 months |
| Observation period | None | 3–12 months |
| Official audit | 2–6 weeks | 1–5 weeks |
| Report delivery | ~1 week | 2–6 weeks |
| **Total** | **2–5 months** | **8–15 months** |

**What affects timeline:**
- Maturity of your existing security controls
- Number of systems in scope
- How many Trust Service Criteria you include
- How quickly you remediate gaps

### Cost

| Component | Typical Range |
|-----------|---------------|
| CPA firm audit (Type 1) | $7,500–$20,000 |
| CPA firm audit (Type 2) | $10,000–$50,000 |
| Readiness consulting (optional) | $5,000–$20,000 |
| Compliance automation platform | $7,500–$30,000/year |
| New security tooling | $5,000–$30,000+ |
| **Total all-in (typical small company)** | **$30,000–$100,000** |

**What drives cost higher:**
- Many systems in scope
- Multiple Trust Service Criteria
- Poor starting security posture (more remediation needed)
- Using a large national or Big 4 CPA firm

---

## 11. Tools That Help

### Compliance Automation Platforms

These tools connect to your cloud environment and SaaS tools (AWS, GitHub, Okta, Slack, etc.), automate evidence collection, run continuous monitoring, and manage the audit process end-to-end. Using one of these significantly reduces the manual effort required.

| Platform | Best For | Notable Strength |
|----------|----------|-----------------|
| **Vanta** | Startups, first-time SOC 2 | Fastest setup, 200+ integrations |
| **Drata** | Scale-ups, technical teams | Deep automation, multi-framework support |
| **Secureframe** | Mid-market | Pre-built templates, structured onboarding |
| **Sprinto** | SMBs wanting minimal manual effort | End-to-end automation |
| **Thoropass** | One-vendor approach | Combined software + auditor |

### Common Supporting Tools

| Category | Tools |
|----------|-------|
| Identity & Access Management | Okta, Azure AD, JumpCloud |
| Endpoint Security | CrowdStrike, SentinelOne, Jamf |
| Vulnerability Scanning | Qualys, Tenable, Detectify |
| Security Monitoring (SIEM) | Splunk, Datadog, Sumo Logic |
| Penetration Testing | Cobalt, HackerOne, Synack |
| Password Management | 1Password, Bitwarden |
| Secret Management | HashiCorp Vault, AWS Secrets Manager |

### Specialized SOC 2 CPA Firms

- **A-LIGN** — large firm specializing in SOC 2, FedRAMP, ISO 27001
- **Linford & Company** — boutique SOC 2 specialist
- **Schellman** — well-known for SOC 2 and FedRAMP
- **Big 4** (Deloitte, EY, PwC, KPMG) — used by large enterprises

---

## 12. Quick Reference Cheatsheet

| Topic | Key Fact |
|-------|----------|
| Created by | AICPA |
| Year introduced | 2010 |
| Report type | Attestation (not certification) |
| Issued by | Licensed CPA firms only |
| Required category | Security (CC1–CC9) |
| Optional categories | Availability, Processing Integrity, Confidentiality, Privacy |
| Type 1 | Point-in-time design check |
| Type 2 | 3–12 month effectiveness check |
| Type 1 timeline | 2–5 months |
| Type 2 timeline | 8–15 months |
| Typical all-in cost | $30,000–$100,000 |
| Best automation tools | Vanta, Drata, Secureframe, Sprinto |
| Renewal frequency | Annually |
| Enterprise buyer requirement | 85% require SOC 2 before signing |
| Legal requirement | Voluntary (but expected in practice) |
| Difference from ISO 27001 | SOC 2 is an attestation report; ISO 27001 is a certification — both are widely respected |

---

## Summary

SOC 2 is the security trust standard for B2B technology companies. It was created by the AICPA to give customers a standardized way to verify that software vendors protect data responsibly.

**The short version:**
1. It audits your security controls against 5 criteria — Security is mandatory, the rest are optional
2. Type 1 checks if controls are designed right; Type 2 checks if they actually work over time
3. It's not legally required, but practically essential if you sell to enterprises
4. Getting it takes 2–15 months and $30K–$100K depending on your situation
5. Compliance automation tools (Vanta, Drata, etc.) significantly reduce the burden
6. It must be renewed every year — it's an ongoing practice, not a checkbox

---

*Sources: AICPA, Vanta, Drata, Secureframe, Linford & Company, A-LIGN, IBM Cost of a Data Breach Report 2024*
