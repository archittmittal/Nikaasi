# Product Requirements Document (PRD): Nikaasi

## Document Control

- **Document Version:** 1.0.0
- **Project Name:** Nikaasi (Provident Fund Dispute and Claim Resolution Engine)
- **Track:** Build What Moves India (Public Digital Infrastructure Track)
- **Classification:** Public Digital Infrastructure Specification (Independent Prototype)

---

## 1. Executive Summary and Problem Statement

In India, Provident Fund (PF) deposits represent compulsory, salary-deducted life savings managed by the Employees' Provident Fund Organisation (EPFO). Over 796 lakh claims were submitted in FY 2024-25, out of which **174 lakh claims (21.8%) were rejected**.

Current public portal infrastructure notifies citizens of claim rejections weeks after filing through unhelpful numeric reason codes (such as "Code 104-B" or "KYC Incomplete"). The member is left with no actionable guidance on corrective steps and frequently enters a loop of blind resubmissions. Furthermore, when an ex-employer fails to record the member's Date of Exit (DoE), the citizen is rendered completely powerless, as the portal provides no statutory mechanism for self-attestation or administrative escalation.

Nikaasi re-architects this process by shifting from retrospective failure notification to **pre-emptive pre-flight cross-validation**, **plain-language intake**, **exit-date self-attestation**, and **transparent, time-bound statutory escalation**.

---

## 2. Target Personas

### Persona 1: Rajesh Kumar (First-Time Job Switcher)
- **Profile:** 24-year-old delivery executive transitioning to a warehouse supervisory role.
- **Pain Point:** Has an unmerged UAN from his previous employer. His name is recorded as "Rajesh Kumar" on Aadhaar and "Rajesh Kr" on EPFO.
- **Goal:** Transfer or withdraw PF without his claim bouncing after three weeks.

### Persona 2: Sunita Devi (Defunct Employer Worker)
- **Profile:** 38-year-old garment worker whose previous factory closed operations without filing member exit dates.
- **Pain Point:** Cannot file Form 19/10C because Date of Exit is blank; HR is unreachable.
- **Goal:** Establish her exit date using salary slips and bank statements and withdraw her accumulated PF.

### Persona 3: Amit Verma (Medical Emergency Advance Claimant)
- **Profile:** 45-year-old factory technician requiring immediate funds for an emergency hospitalization.
- **Pain Point:** Confused by regulatory jargon (Form 31, Para 68J, Rule 68B) and does not know how much he is legally entitled to withdraw.
- **Goal:** State his emergency in simple Hindi/English and receive an auto-filled, eligible advance claim.

---

## 3. Regulatory and Statutory Framework

Nikaasi aligns with the Employees' Provident Funds and Miscellaneous Provisions Act, 1952, and its subordinate schemes:

| Statutory Instrument | Regulatory Scope | Traditional Requirement | Nikaasi Transformation |
| :--- | :--- | :--- | :--- |
| **Form 19** | Final Settlement of Provident Fund | Requires minimum 2 months of unemployment and marked Date of Exit (DoE). | Auto-computed upon unemployment intent; triggers Self-Attestation flow if DoE is missing. |
| **Form 10C** | Pension Withdrawal Benefit / Scheme Certificate | Eligible for members with service between 6 months and 10 years. | Automatically bundled with Form 19 based on service tenure calculation. |
| **Form 31 (Para 68J)** | Non-Refundable Advance for Medical Treatment | Requires medical certificate and minimum balance eligibility. | Auto-mapped from natural language input; automatically calculates eligible ceiling. |
| **Form 31 (Para 68B)** | Advance for Purchase / Construction of House | Requires minimum 5 years of continuous service. | Service history automatically validated across all active and legacy UANs. |

---

## 4. Functional Requirements

### FR-01: Plain-Language Conversational Intent Intake
- **Description:** The system must accept natural language descriptions of the user's circumstances in English or Hindi and automatically identify the correct statutory form and paragraph.
- **Inputs:** Free-form text input or guided scenario selections (e.g., job resignation, medical emergency, home purchase).
- **Output:** Selected claim package (Form 19, Form 10C, or Form 31 with specific paragraph mapping) along with calculated maximum withdrawal eligibility.

### FR-02: Pre-Flight Cross-Validation Engine
- **Description:** The system must validate member profile data across simulated Aadhaar, PAN, and Bank databases prior to formal claim generation.
- **Validation Rules:**
  - `RULE-01 (Identity Match):` Aadhaar Name vs. EPFO Name fuzzy similarity must score >= 0.88 (Jaro-Winkler). Transliteration differences must be flagged with a non-blocking normalization suggestion.
  - `RULE-02 (Date of Birth):` Date of birth across Aadhaar and EPFO must match exactly.
  - `RULE-03 (Bank Seeding):` Bank account must be active, have completed penny-drop verification, and be linked to the member's Aadhaar.
  - `RULE-04 (Service History):` Check for fragmented UANs and overlapping service dates.

### FR-03: Prescriptive Class A Remediation Guide
- **Description:** If a Class A data mismatch is detected during pre-flight validation, the system must generate a step-by-step resolution roadmap.
- **Requirements:** 
  - Provide clear, sequenced instructions on which record to update first (e.g., Joint Declaration vs. Bank Seeding).
  - Estimate the resolution time for each corrective step.

### FR-04: Exit-Date Self-Attestation Portal (Class B Resolution)
- **Description:** When a member's Date of Exit (DoE) is absent, the system must allow the member to self-declare their exit date by submitting evidentiary documents.
- **Accepted Proof Types:** Last salary slip, Form 16 Part A/B, or bank statement showing the cessation of salary credits.
- **Integrity Guarantee:** Generate a SHA-256 hash of all uploaded documents and create a tamper-evident self-attestation package.

### FR-05: 15-Day Employer SLA Escalation State Machine
- **Description:** Upon submission of an exit-date self-attestation, an automated 15-calendar-day SLA clock is initiated for the ex-employer.
- **Workflow:**
  - Day 0: Formal verification notice dispatched to employer HR.
  - Days 1-14: Active countdown displayed to both citizen and employer.
  - Day 15: If employer fails to confirm or contest, claim is automatically transitioned to `ESCALATED_TO_REGIONAL_OFFICE`.
  - Field Office Action: EPFO Assistant Commissioner receives dossier with documentary proofs for administrative override.

### FR-06: Radical Honest Tracking and Observability Dashboard
- **Description:** Provide a transparent, real-time tracking interface that displays the exact holding desk, active SLA timer, and escalation hierarchy.
- **Display Fields:** Current Stage, Assigned Entity (e.g., Employer HR Desk or Field Office), Days Elapsed / Remaining, Next Automatic Action, and Direct Contact Information.

### FR-07: Interactive Evaluation Sandbox Controls
- **Description:** For evaluation and demonstration purposes, provide controls to switch between mock personas and advance time (simulate Day 0 to Day 15 SLA transitions).

---

## 5. Non-Functional Requirements

### NFR-01: Performance and Latency
- Pre-flight cross-validation results must be rendered within 500 milliseconds.
- Intent classification must complete within 300 milliseconds.

### NFR-02: Accessibility and Usability
- Interface must conform to WCAG 2.1 AA guidelines.
- Responsive mobile-first design supporting viewports from 360px width upwards.
- Full bilingual language parity between Hindi and English.

### NFR-03: Data Privacy and Sandboxing
- Strict adherence to the Digital Personal Data Protection (DPDP) Act, 2023.
- No live government API calls; 100% synthetic mock data.
- Aadhaar numbers must be masked, displaying only the last 4 digits (e.g., `XXXX-XXXX-1234`).

### NFR-04: Deterministic Auditability
- Every state change in the claim lifecycle must generate an immutable, timestamped audit log.

---

## 6. Acceptance Criteria

1. **Intake Accuracy:** Entering a scenario description indicating job departure 3 months ago correctly resolves to Form 19 + Form 10C.
2. **Pre-Flight Diagnostic:** A name discrepancy (e.g., "Archit Mittal" vs "Architt Mittal") is surfaced with phonetic match confidence and does not result in an opaque rejection.
3. **Attestation and SLA:** Uploading supporting proof initiates a 15-day SLA. Simulating time-travel to Day 15 automatically moves the claim to the Field Office queue.
4. **Zero Live Data Leakage:** All data models operate entirely in a sandbox environment without contacting external services.
