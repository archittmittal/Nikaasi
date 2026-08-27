# Product Requirements Document (PRD): Nikaasi

## Document Control

- **Document Version:** 1.1.0
- **Project Name:** Nikaasi (Provident Fund Dispute and Claim Resolution Engine)
- **Track:** Build What Moves India (Public Digital Infrastructure Track)
- **AI Core:** OpenAI GPT-4o / Codex Structured Intent Extraction Layer
- **Classification:** Public Digital Infrastructure Specification (Independent Prototype)

---

## 1. Builder Brief Alignment and Six Core Evaluation Questions

This specification directly operationalizes the evaluation criteria defined in the **Build What Moves India** builder brief:

### 1.1 Who is facing the problem?
The target population is India's salaried workforce governed by the Employees' Provident Fund Organisation (EPFO), representing over 70 million actively contributing members and 300 million total registered UANs. The failure modes disproportionately penalize:
- First-time job switchers who have unmerged, fragmented UANs.
- Contractual and blue-collar workers in high-turnover sectors (construction, security, logistics, manufacturing).
- Citizens whose vernacular names have been transliterated with minor spelling inconsistencies between Aadhaar, PAN, and EPFO records.

### 1.2 What is difficult about the current experience?
The legacy EPFO member portal (UAN Member e-Sewa) and grievance portal (EPFiGMS) operate on a punitive, asynchronous model:
- Rejections arrive 15 to 25 days after submission as terse, non-actionable numeric error codes (e.g., "Rejected: 104-B").
- Citizens are not provided with an explanation of the root cause or a structured sequence of corrective actions.
- When an ex-employer fails to record the member's Date of Exit (DoE), the citizen has zero portal-based recourse and is locked in an indefinite administrative freeze.

### 1.3 What did you change?
1. **Pre-emptive Pre-Flight Validation:** Inspects member data against simulated Aadhaar, PAN, and banking databases *before* claim generation.
2. **OpenAI-Powered Conversational Intake:** Replaces complex statutory forms (Forms 19, 10C, 31) with natural language comprehension in Hindi, English, and Hinglish.
3. **Exit-Date Self-Attestation with 15-Day SLA Clock:** Introduces alternative documentary evidence submission and an automated statutory escalation clock.
4. **Radical Honest Tracking:** Replaces vague "Under Process" messages with explicit holding desk assignments, live countdown timers, and escalation points.

### 1.4 Why is your version better?
The current system judges the citizen retrospectively after weeks of silence. Nikaasi inverts this relationship by acting as an active citizen advocate: preventing clerical errors before filing and placing an enforceable deadline on third parties.

### 1.5 What works today, and what is still mocked?
- **Operational Prototype:** End-to-end interactive citizen journey, OpenAI conversational intent classification, multi-field identity diffing, cryptographic evidence hashing, 15-day SLA state machine with time-travel evaluation controls, and transparent tracking dashboard.
- **Mocked Sandbox:** Authoritative government registries (Aadhaar, PAN, NPCI penny-drop, EPFO UAN master) are strictly simulated with synthetic personas to maintain complete privacy and zero live PII exposure.

### 1.6 How could the idea work safely at scale?
- **Account Aggregator (AA) Ecosystem:** Pull authenticated bank statements directly via RBI-regulated AA frameworks to verify salary credit cessation without manual file uploads.
- **DigiLocker Integration:** Direct retrieval of digitally signed Form 16 and employment separation records.
- **Aadhaar e-Sign:** Legally binding self-attestation declarations under the IT Act, 2000.
- **Automated Regional Office Dispatch:** Auto-route breached claims to the Assistant PF Commissioner queue under Section 26B administrative override powers.

---

## 2. Target Personas

### Persona 1: Rajesh Kumar (First-Time Job Switcher)
- **Profile:** 24-year-old delivery executive transitioning to a warehouse supervisory role.
- **Pain Point:** Has an unmerged UAN from his previous employer. Name is "Rajesh Kumar" on Aadhaar and "Rajesh Kr" on EPFO.
- **Journey:** Pre-flight engine flags transliteration difference with non-blocking confidence, merges UAN records, and prepares Form 19/10C package.

### Persona 2: Sunita Devi (Defunct Employer Worker)
- **Profile:** 38-year-old garment worker whose previous employer shuttered without recording employee exit dates.
- **Pain Point:** Cannot submit final settlement because Date of Exit is blank; HR is defunct.
- **Journey:** Enters self-attestation flow, uploads Form 16 and last pay slip, triggers 15-day SLA clock, and auto-escalates to Regional Commissioner.

### Persona 3: Amit Verma (Medical Emergency Advance Claimant)
- **Profile:** 45-year-old technician requiring emergency funds for immediate hospitalization.
- **Pain Point:** Confused by statutory jargon (Para 68J, Form 31) and maximum ceiling limits.
- **Journey:** Types "Need 50000 rupees for my father's surgery"; OpenAI parser identifies Form 31 (Para 68J) and calculates instant eligibility.

---

## 3. Regulatory and Statutory Framework

| Statutory Instrument | Scope | Traditional Rule | Nikaasi Transformation |
| :--- | :--- | :--- | :--- |
| **Form 19** | Full & Final PF Settlement | Requires 2 months unemployment and verified Date of Exit. | Auto-selected upon unemployment intent; triggers Self-Attestation if DoE is missing. |
| **Form 10C** | Pension Withdrawal Benefit | Requires between 6 months and 10 years of total service. | Automatically bundled with Form 19 based on service history calculation. |
| **Form 31 (Para 68J)** | Non-Refundable Advance for Illness | Requires medical emergency intent and minimum balance. | OpenAI intent parser maps illness statements directly to Para 68J with ceiling calculations. |
| **Form 31 (Para 68B)** | Advance for Housing / Construction | Requires minimum 5 years continuous service. | Service history automatically verified across legacy and active UANs. |

---

## 4. Functional Requirements

### FR-01: OpenAI-Powered Natural Language Intake
- System must accept unstructured natural language input in English, Hindi, and transliterated Hinglish.
- OpenAI model extracts: `employmentStatus`, `intentCategory`, `requestedAmount`, `monthsSinceExit`, and `serviceTenureMonths`.
- Outputs structured JSON mapped to statutory form requirements with deterministic fallback validation.

### FR-02: Pre-Flight Algorithmic Cross-Validation
- Validates member profile data across simulated Aadhaar, PAN, and Banking records.
- Executes Jaro-Winkler string similarity on full names (threshold >= 0.88 for phonetic equivalence).
- Checks active NPCI Aadhaar-bank account seeding and IFSC validity.

### FR-03: Prescriptive Guided Remediation (Class A)
- If identity or bank discrepancies are detected, generates an ordered, step-by-step remediation roadmap.

### FR-04: Exit-Date Self-Attestation (Class B)
- When Date of Exit (DoE) is absent, enables worker to declare exit date and attach supporting evidence (Form 16, salary slips, bank statement).
- Generates SHA-256 cryptographic hashes of uploaded evidence.

### FR-05: 15-Day SLA Escalation State Machine
- Initiates an enforceable 15-calendar-day countdown on the ex-employer.
- Auto-escalates the claim dossier to the EPFO Regional Office (RPFC) on Day 15 if unacknowledged.

### FR-06: Radical Honest Tracking Dashboard
- Displays live holding desk assignment, remaining SLA days, and escalation nodal contacts.

### FR-07: Evaluator Sandbox Controls
- Provides instant persona switching and time-travel controls to test Day 0 to Day 15 SLA state transitions.

---

## 5. Non-Functional Requirements

- **NFR-01 (Performance):** Pre-flight validation response < 500ms; OpenAI intent parsing < 1500ms.
- **NFR-02 (Accessibility):** WCAG 2.1 AA compliant, mobile-first responsive layout (down to 360px width), bilingual Hindi/English.
- **NFR-03 (Data Privacy):** 100% synthetic sandbox data; no real citizen PII; zero backend PII persistence.
- **NFR-04 (Auditability):** Deterministic audit trail for every state transition with ISO 8601 UTC timestamps.

---

## 6. Hackathon Deliverables Specification

1. **Public Live Link:** Deployed on modern edge infrastructure, accessible directly in any web browser without authentication barriers or application downloads.
2. **Video Demonstration (≤ 2 Minutes):**
   - Minute 1: End-to-end citizen journey demonstrating deliberate Class B failure and self-attestation recovery.
   - Minute 2: Architectural "how and why", OpenAI integration, and India Stack scale-up model.
3. **Written Summary:** Structured executive summary under 250 words.
