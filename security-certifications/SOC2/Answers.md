# SOC 2 — Answers

---

## Block 1 — Understanding the Trust Service Criteria (TSC)

> SOC 2 audits are based on 5 Trust Service Criteria (TSC): Security, Availability, Processing Integrity, Confidentiality, and Privacy. Security (CC) is the only mandatory one.

---

### Q1. Can a company fail the other 4 optional criteria if they choose to include them in scope? If yes, what happens — does the whole audit fail, or just that criterion?

**Yes, you can receive adverse findings on any TSC you include — including the optional ones.**

But SOC 2 doesn't work like a pass/fail exam. Here's how it actually works:

The auditor issues an **opinion** on each criterion in scope. There are three possible outcomes:

| Opinion | What It Means |
|---------|---------------|
| **Unqualified (Clean)** | Controls are suitably designed and/or operating effectively — no significant issues |
| **Qualified** | Controls are mostly sound but there are specific, limited exceptions noted |
| **Adverse** | Controls have material weaknesses — the auditor cannot provide assurance |
| **Disclaimer** | I couldn't get enough evidence to form any opinion |

Qualified - it means the report is Qualified for the opinion, also meaning 'Overall the controls are fine, but there is a specific issue I cannot ignore — so my opinion comes with that caveat attached'

A qualified or adverse opinion on one optional TSC does **not** automatically invalidate the entire report. The report still gets issued. Each TSC section gets its own opinion. For example, your Security criteria could receive a clean opinion while your Availability criteria receives a qualified one.

**What happens practically:**
- The report gets issued with the findings documented
- You provide a "management response" explaining the issue and your remediation plan
- Customers who review the report will see the exception — this can affect their trust and decision to work with you
- You address the gaps and the next annual audit should reflect the remediation

**The key takeaway:** A finding in an optional TSC is not catastrophic, but it is visible to every customer you share the report with. This is why choosing scope carefully matters.

---

### Q2. What is the strategic purpose of voluntarily including additional TSCs in scope if they introduce more risk of failure?

This is a sharp question. The answer: **the business upside outweighs the audit risk, when chosen deliberately.**

Here's the strategic logic:

**1. Customer requirements drive it**
Your customers may specifically require certain TSCs. For example:
- A healthcare customer may require the **Privacy** TSC to feel confident their patient-adjacent data is protected
- A customer with an SLA may require **Availability** to verify you have uptime controls

If your customers ask for it, excluding it leaves a gap in their trust.

**2. It signals depth and maturity**
Including Availability or Confidentiality beyond the bare minimum of Security signals that you take compliance seriously — not just checking a box. This matters in competitive enterprise sales.

**3. Regulatory alignment**
- Including **Privacy** TSC helps demonstrate GDPR/CCPA alignment
- Including **Confidentiality** helps with contractual obligations around protecting client IP
- This reduces duplicated work when pursuing other compliance frameworks later

**4. The risk is manageable**
If you've already implemented the controls properly, the additional TSC doesn't add audit risk — it just adds audit scope. The risk only materializes if you include a TSC without having the controls in place (which you'd discover during a gap analysis before the audit begins).

**The strategic rule:** Only include optional TSCs when (a) customers require it, (b) you have the controls in place to support it, or (c) it aligns with regulatory obligations you already have.

---

### Q3. Who performs and verifies a SOC 2 audit — what are their qualifications and who governs them?

**A SOC 2 report can only be issued by a licensed CPA (Certified Public Accountant) firm operating under AICPA standards.**

Here's the breakdown:

**Who performs it:**
- A **CPA firm** that specializes in attestation engagements
- The engagement is led by a **licensed CPA** who signs the final opinion letter
- Supporting staff (associates, IT auditors) may assist with testing — but the signing CPA is legally responsible for the opinion

**What qualifications are required:**
- The firm must hold a valid CPA license issued by a state board of accountancy
- The firm must follow **SSAE 18** (Statement on Standards for Attestation Engagements No. 18) — the technical standard governing SOC 2 audits
- The firm must maintain **independence** from the client — they cannot audit a company they have a financial interest in or a significant business relationship with

**Who governs them:**
- The **AICPA** (American Institute of CPAs) sets the standards and the Trust Services Criteria
- **State boards of accountancy** license CPA firms and have authority to revoke licenses for misconduct
- The **PCAOB** (Public Company Accounting Oversight Board) governs auditors of public companies — but for SOC 2 (which covers private companies mostly), the AICPA is the primary standard-setter
- Firms that perform SOC audits are subject to **peer reviews** — another CPA firm periodically reviews their work quality

**Important nuance:** Compliance automation platforms (Vanta, Sprinto, Drata) are **not** auditors. They prepare you for the audit. Only a CPA firm can issue the report.

---

## Block 2 — Getting a SOC 2 Type 2 Report (cost, process, payment)

> Assuming a SOC 2 Type 2 audit costs approximately $15,000.

---

### Q1. What is the end-to-end process to obtain a SOC 2 Type 2 report — what are the stages, timelines, and key decisions involved?

Here is the full end-to-end process:

**Stage 1: Define Scope** (1–2 weeks)
- Identify which systems, services, and data flows are in scope
- Choose which Trust Service Criteria to include (at minimum: Security)
- Key decision: broad scope = more work + cost; narrow scope = faster + cheaper but less useful to customers

**Stage 2: Gap Analysis** (2–4 weeks)
- Compare current controls against TSC requirements
- Identify missing policies, undocumented procedures, and weak controls
- Output: a prioritized remediation list

**Stage 3: Remediation** (2–6 months)
- Implement missing technical controls (MFA, encryption, logging, etc.)
- Write and publish required security policies
- Set up employee training
- Establish vendor management and access review processes
- Key decision: use a compliance automation tool (Vanta/Sprinto/Drata) to streamline this or do it manually

**Stage 4: Select and Engage a CPA Firm** (2–4 weeks)
- Interview 2–3 CPA firms that specialize in SOC 2
- Sign an engagement letter
- Agree on the observation period start date

**Stage 5: Observation Period** (3–12 months)
- This is unique to Type 2 — the auditor observes your controls operating in real time
- You must consistently follow your controls during this entire period
- Evidence is collected continuously (logs, access reviews, training records, etc.)
- Key decision: 3-month observation = faster report but less credible; 12-month = most credible, especially for first-time reports

**Stage 6: Fieldwork / Audit Testing** (4–8 weeks)
- Auditor reviews documentation, interviews staff, samples evidence
- They test whether controls actually worked during the observation period

**Stage 7: Draft Report Review** (1–2 weeks)
- Auditor shares draft findings
- You provide management responses for any exceptions found

**Stage 8: Final Report Issued** (1 week)
- CPA firm delivers the signed SOC 2 Type 2 report
- You now have a confidential document you can share with customers under NDA

**Total timeline:** 8–15 months from start to report

---

### Q2. Who exactly performs the audit — what type of firm or individual is authorized to issue a SOC 2 report?

**Only a licensed CPA firm operating under AICPA SSAE 18 standards can issue a SOC 2 report.**

- The firm must hold an active CPA license
- The **engagement partner** (a licensed CPA) personally signs and is legally accountable for the opinion
- The firm must be independent of the company being audited (no financial conflict of interest)
- The firm must pass periodic peer reviews of their audit quality

Non-CPA security firms, compliance platforms, and consultants **cannot** issue SOC 2 reports — even if they help you prepare for one.

Examples of firms authorized to issue SOC 2 reports:
- Specialized boutiques: A-LIGN, Linford & Company, Schellman, Sensiba
- Regional CPA firms with IT audit practices
- Big 4 firms (Deloitte, EY, PwC, KPMG) for larger enterprises

---

### Q3. Who do you pay, and what do you get in return — a report, a certificate, or something else?

**You pay the CPA firm. You receive a SOC 2 report — not a certificate.**

Here's what that means in practice:

**What you pay for:**
- The CPA firm's professional fees for conducting the audit engagement
- This covers gap analysis (if included), observation period review, fieldwork, and report drafting
- If using a compliance automation tool (Sprinto, Vanta, etc.), you pay them separately for their software platform

**What you receive:**
- A **SOC 2 Type 2 report** — a formal PDF document (typically 30–100+ pages)
- The report contains:
  - A description of your system (written by your management)
  - The auditor's opinion letter (signed by the CPA)
  - A detailed list of the controls tested and the auditor's findings for each
  - Any exceptions noted and your management responses

**What you do NOT receive:**
- A certificate, badge, or seal (SOC 2 doesn't work like ISO 27001)
- A public credential — the report is confidential
- A permanent status — it expires annually and must be renewed

**How customers access it:**
- You share the report directly with customers or prospects, typically under an NDA
- Some companies use a trust portal (e.g., via Vanta or TrustCloud) where customers can request access and download the report in a controlled way

---

### Q4. If the company's infrastructure is hosted on AWS, how does that affect the audit scope and process?

This is an important and commonly misunderstood area. The concept is called **Shared Responsibility**.

**AWS's responsibility vs. your responsibility:**

| Layer | Who is Responsible | What It Means for Your Audit |
|-------|-------------------|------------------------------|
| Physical data center security | AWS | You don't need to audit this — AWS's own SOC 2 covers it |
| Hypervisor, network infrastructure | AWS | Same — covered by AWS's SOC 2 |
| Operating system configuration | You (EC2) or AWS (managed services) | You audit what you configure |
| Application code, database | You | You audit this entirely |
| Access to your AWS account | You | IAM, MFA, access reviews — all on you |
| Data encryption in your app | You | Your controls, your audit |

**How AWS affects your audit practically:**

1. **AWS is a "sub-service organization"** — your auditor will note that certain infrastructure controls are provided by AWS and rely on AWS's own SOC 2 report
2. **Your auditor will request AWS's SOC 2 report** (available free via AWS Artifact) to verify that AWS's controls are adequate for the portions they manage
3. **Your scope focuses on your application layer** — access management, application security, data handling, monitoring, and incident response
4. **Using AWS managed services reduces your scope** — for example, using RDS (managed database) means AWS handles patching; using EC2 means you handle patching yourself

**The bottom line:** Using AWS simplifies some controls (physical security, hardware maintenance) but you are still fully responsible for how you configure and use AWS — IAM policies, security groups, encryption settings, logging — all of that is in scope for your audit.

---

## Block 3 — Understanding the SOC 2 Report you already have from AWS

> My company has a SOC 2 Type 2 report associated with AWS.

---

### Q1. Did my company generate/commission this report, or is this AWS's own SOC 2 report for their infrastructure that we inherited by using AWS?

**This is almost certainly AWS's own SOC 2 report — not your company's.**

Here's how to tell:

- AWS publishes its own SOC 2 Type 2 report covering Amazon Web Services infrastructure (data centers, hypervisors, physical security, network controls, etc.)
- Any AWS customer can download this report for free from **AWS Artifact** (the compliance document portal inside the AWS console)
- The report is issued to **Amazon Web Services, Inc.** — not to your company

Your company "has" this report in the sense that you have access to it and can reference it. But it is not **your** SOC 2 report.

**The distinction matters:**
- AWS's SOC 2 proves that AWS's infrastructure is secure
- It says nothing about whether *your application running on AWS* is secure
- Your customers who ask "do you have a SOC 2?" are asking about your application and your controls — not AWS's

To have your **own** SOC 2 report, your company would need to commission a separate audit from a CPA firm scoped to your application and operations.

---

### Q2. What is the difference between AWS's SOC 2 report and my company's own SOC 2 report?

| | AWS's SOC 2 Report | Your Company's SOC 2 Report |
|--|-------------------|----------------------------|
| **What it covers** | AWS's physical infrastructure, data centers, hypervisors, managed services | Your application code, your AWS account configuration, your team's access and processes |
| **Who commissioned it** | Amazon Web Services | Your company |
| **Who is the auditee** | Amazon Web Services, Inc. | Your company |
| **What customers learn from it** | AWS's data centers are physically secure and well-managed | Your application is built and operated securely |
| **Availability** | Free via AWS Artifact | Shared by your company under NDA |
| **Does it satisfy enterprise buyers?** | Only partially — they also need to see your controls | Yes — this is what enterprise buyers are asking for |

**Analogy:** If you rent an office building, the building owner's security audit proves the lobby has cameras and the doors have locks. But your enterprise customer also needs to know that **you** lock your cabinets, control who has keys to your office, and have security training for your staff. That's your own SOC 2.

---

### Q3. If we already have a SOC 2 report, how do I find out which CPA firm issued it, when it was issued, and what was the scope of the audit?

If your company has commissioned its own SOC 2 report, the following information is inside the report document itself:

**How to find the details:**

1. **Open the SOC 2 report PDF**
   - The first 1–2 pages are the **auditor's opinion letter**
   - This letter is signed by the CPA firm — their name, address, and CPA license details are on it
   - The date on the opinion letter tells you when the report was issued

2. **Look at the observation period**
   - The report will state the period of coverage, e.g., "For the period January 1, 2025 to December 31, 2025"

3. **Look at the "Description of the System" section**
   - Written by your management, this section describes what was in scope — which systems, services, and infrastructure were covered

4. **Look at the "Trust Service Criteria Covered" section**
   - This lists which of the 5 TSC (Security, Availability, etc.) were included in scope

**Where to find the report:**
- Your company's legal, compliance, or security team should hold the official copy
- If your company uses a compliance tool like Sprinto, Vanta, or Drata, the report is often stored in the platform
- Check your company's trust portal or security documentation repository (often linked from the company website under "Security" or "Trust")

**If you're not sure whether the report is AWS's or your company's:**
- Open the report and check the name of the "auditee" (the company being audited)
- If it says "Amazon Web Services, Inc." — it's AWS's report
- If it says your company name — it's your company's report

---

## Block 4 — Role of Sprinto in the SOC 2 Process

> Sprinto is described as a compliance automation tool that connects to cloud infrastructure, automates evidence collection, runs continuous monitoring, and manages the end-to-end compliance process.

---

### Q1. Does a tool like Sprinto actually issue or grant the SOC 2 attestation, or is it just a preparation and automation layer?

**Sprinto does not issue or grant the SOC 2 attestation. It is purely a preparation and automation layer.**

The legal authority to issue a SOC 2 report rests exclusively with a licensed CPA firm. No software tool — regardless of how comprehensive it is — can issue an attestation report. That requires a licensed professional who can be held legally accountable for their opinion.

What Sprinto does:
- Connects to your cloud infrastructure, SaaS tools, HR systems, and identity providers
- Automatically collects evidence of controls operating (access logs, training completions, scan results, etc.)
- Monitors for control failures in real time and alerts you to fix them
- Stores and organizes your evidence library for the auditor
- Provides pre-built policy templates
- Manages the audit workflow — which tests are pending, which evidence is collected, what needs action

What Sprinto does NOT do:
- Conduct the audit
- Issue an opinion letter
- Grant you a SOC 2 report or attestation

Think of Sprinto as a highly organized project manager and evidence collector for your SOC 2 journey — but the auditor still has to independently review everything and issue the final report.

---

### Q2. What is the exact relationship between a compliance automation platform (like Sprinto) and the licensed CPA firm — who does what?

They play complementary but distinct roles:

| Role | Compliance Platform (Sprinto) | CPA Firm (Auditor) |
|------|------------------------------|-------------------|
| **Scope** | Preparation, automation, monitoring | Audit, testing, opinion |
| **Evidence collection** | Automated, continuous | Samples and reviews what you provide |
| **Policy templates** | Provides pre-built templates | Reviews policies for adequacy |
| **Control monitoring** | Real-time alerts when controls break | Tests controls at specific points |
| **Audit workflow** | Manages tasks, tracks readiness | Conducts independent testing |
| **Final report** | Cannot issue | Signs and issues the opinion letter |
| **Legal accountability** | None | Legally liable for the opinion |
| **Paid by** | Your company (annual SaaS fee) | Your company (per-engagement fee) |

**How they interact in practice:**
1. You use Sprinto to build and maintain your controls
2. Sprinto generates an evidence package (logs, records, documents)
3. You share Sprinto's evidence portal access with your CPA firm
4. The CPA firm reviews and independently tests the evidence
5. The CPA firm forms their opinion and issues the report

Some compliance platforms (like Thoropass, formerly Laika) have partner CPA firms embedded in their model — so you buy the software and the audit from one vendor. But even then, the audit function is performed by the CPA partner, not the software.

---

### Q3. Can you get a SOC 2 report without a tool like Sprinto, and what does Sprinto specifically replace or simplify?

**Yes, absolutely. SOC 2 existed for over a decade before these tools did. You can get SOC 2 without Sprinto.**

What the manual (no-tool) approach looks like:
- Manually write all security policies in Google Docs or Word
- Manually collect evidence: export logs, take screenshots, save training certificates
- Store everything in spreadsheets or shared drives
- Manually track which controls need attention and when
- Manually coordinate with the auditor via email and shared folders

**What Sprinto replaces or simplifies:**

| Without Sprinto | With Sprinto |
|----------------|-------------|
| Manual screenshot collection from 10+ tools | Automated API integrations pull evidence continuously |
| Spreadsheet tracking of control status | Real-time dashboard showing what's passing, failing, or needs attention |
| Writing policies from scratch | Pre-built, auditor-approved policy templates |
| Discovering control failures after the audit | Alerts when a control breaks (e.g., someone disabled MFA) |
| Back-and-forth email with auditor | Auditor portal where they can directly access evidence |
| Significant internal staff time | Dramatically reduced time per person |

**When NOT using a tool makes sense:**
- You have a dedicated, experienced security/compliance team who can handle this manually
- Your scope is very small and simple
- You want to minimize SaaS spending and have the internal capacity

**When a tool like Sprinto is worth it:**
- Your team is small and everyone wears multiple hats
- You're pursuing SOC 2 for the first time and don't know what evidence to collect
- You want to pursue multiple frameworks (SOC 2 + ISO 27001 + HIPAA) with shared evidence
- You want ongoing, continuous compliance monitoring — not just point-in-time audit preparation

---

## Block 5 — Why "Attestation" and Not "Certification"

> SOC 2 is formally called an "Attestation" report, not a "Certification."

---

### Q1. What is the precise legal and professional difference between an attestation and a certification in the context of compliance?

These two words have distinct meanings in professional standards:

**Attestation:**
- A licensed professional (CPA) expresses a **written conclusion** about a subject matter based on evidence they have examined
- The subject matter is defined and presented by the **responsible party** (the company being audited)
- The scope, criteria, and assertions are **flexible** — the company defines what it's asserting, and the auditor evaluates those specific assertions
- Governed by **SSAE 18** (for US attestations) — the professional standard for CPAs performing attestation engagements
- Legal accountability: The CPA is professionally liable for their opinion; the company is responsible for the accuracy of the system description and its management assertions
- Example: SOC 2, SOC 1

**Certification:**
- An **accredited certification body** verifies that an organization meets a **fixed, universal standard**
- The criteria are defined externally (by the standard) — not by the company
- The result is a **certificate** that can be publicly displayed
- Governed by international standards (e.g., ISO/IEC 17021 for certification bodies)
- The certifying body's accreditation is granted by a national accreditation body (e.g., UKAS in UK, ANAB in US)
- Example: ISO 27001, ISO 9001, PCI DSS

**The core difference in one sentence:**
> In an attestation, *you* define what you're claiming and a CPA verifies it. In a certification, *an external standard* defines what you must achieve and a certification body verifies it.

---

### Q2. Why is SOC 2 specifically structured as an attestation — what does that imply about accountability, scope, and the auditor's responsibility?

SOC 2 is structured as an attestation for deliberate reasons:

**1. Flexibility across diverse business models**
A SaaS company providing HR software, a cloud hosting provider, and a payments processor all have fundamentally different systems and risk profiles. A rigid universal certification standard would either be too strict for some or too loose for others. Attestation allows each company to define its own scope and relevant controls while still having an independent professional verify those claims.

**2. Scope is the company's responsibility**
The company writes the "Description of the System" section — which systems are in scope, what the service commitments are, what the risks are. The auditor then evaluates whether the controls described are sufficient for those commitments. This means:
- If you define scope too narrowly to hide weaknesses, that's your management's responsibility
- The auditor is responsible for their **opinion on what you presented** — not for designing the scope for you

**3. Auditor accountability is bounded**
The CPA's opinion is on:
- Whether the controls are *suitably designed* (Type 1)
- Whether the controls *operated effectively* during the period (Type 2)

The CPA is **not** asserting that your company is perfectly secure or that no breach is possible. They are asserting that the controls you defined, within the scope you defined, appear to be working as described.

**4. Implications:**
- A customer reading a SOC 2 report must also evaluate the scope (not just the clean opinion) — a narrow scope could mean important systems were excluded
- The auditor's professional license is on the line if they issue a false opinion — this creates accountability
- Management's integrity is on the line for the system description and assertions — this creates accountability on the company side

---

### Q3. Are there compliance frameworks that are genuinely called "certifications" (e.g., ISO 27001), and how do they differ from SOC 2?

Yes. Several major frameworks are true certifications. Here is a comparison:

| | SOC 2 (Attestation) | ISO 27001 (Certification) | PCI DSS (Certification-like) |
|--|--------------------|--------------------------|-----------------------------|
| **Type** | Attestation | Certification | Assessment / Certification |
| **Standards body** | AICPA | ISO / IEC | PCI Security Standards Council |
| **Who audits** | Licensed CPA firm | Accredited ISO certification body | Qualified Security Assessor (QSA) |
| **Criteria fixed?** | Flexible — company-defined scope | Fixed — all Annex A controls apply | Fixed — 12 requirements |
| **Output** | SOC 2 report (confidential document) | ISO 27001 certificate (publicly displayable) | Report on Compliance (ROC) or SAQ |
| **Publicly displayable?** | No (confidential, shared under NDA) | Yes (certificate can be published) | Depends on tier |
| **Geographic focus** | Primarily US-market driven | Global, especially Europe/Asia | Global (payment card industry) |
| **Renewal** | Annually | Every 3 years (with annual surveillance audits) | Annually |
| **Scope flexibility** | High — you define what's in scope | Moderate — ISMS scope can be defined, but all Annex A controls apply | Low — all 12 requirements are mandatory for applicable entities |

**Key practical differences:**

- **ISO 27001 certificate** can be posted on your website, included in RFPs without an NDA, and is recognized globally — especially valued in European and APAC markets
- **SOC 2 report** is the preferred trust document in the US B2B market — particularly in SaaS and cloud services
- **They are complementary** — many mature companies pursue both. There is significant control overlap (60–70%), so achieving one makes the other significantly easier
- **For a startup targeting US enterprise customers:** Start with SOC 2
- **For a company targeting European customers or global enterprise:** Add ISO 27001

**Who "stands behind" the claim:**
- In SOC 2: The CPA firm's professional reputation and license
- In ISO 27001: The certification body's accreditation (backed by a national accreditation body)
- Both involve independent third parties with formal accountability — neither is self-attestation

---

*These answers are based on AICPA standards (SSAE 18, TSC 2017), AWS Shared Responsibility Model documentation, ISO/IEC 27001:2022, and publicly available guidance from Vanta, Drata, Secureframe, Sprinto, Linford & Company, and A-LIGN.*
