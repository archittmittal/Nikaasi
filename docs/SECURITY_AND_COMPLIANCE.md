# Nikaasi Security and Compliance Guide

## Document Control

- **Document Version:** 1.0.0
- **Status:** Approved Security Specification
- **Classification:** Security and Data Privacy Standard

---

## 1. Regulatory and Legal Compliance Framework

### 1.1 Digital Personal Data Protection (DPDP) Act, 2023

Nikaasi adheres to the core data minimization, purpose limitation, and consent principles established under India's Digital Personal Data Protection Act, 2023:

1. **Purpose Limitation:** Member identifiers (such as synthetic UAN and PAN) are processed solely for cross-validation and statutory claim generation.
2. **Data Minimization:** No biometric data is collected, stored, or processed. Aadhaar data is strictly limited to synthetic, masked strings.
3. **Storage Limitation (Zero-Retention Runtime):** For the evaluation prototype, all state modifications reside in ephemeral memory or client local storage; no permanent PII store is maintained.

### 1.2 Aadhaar Act (Section 29) Masking Standards

In compliance with UIDAI regulations and Section 29 of the Aadhaar Act:
- Aadhaar numbers must never be displayed in plain text.
- Standard display formatting requires full masking of the first 8 digits (e.g., `XXXX-XXXX-1234`).
- No Aadhaar demographic data is stored in unencrypted format.

---

## 2. Threat Model (STRIDE Methodology)

| Threat Category | Potential Attack Vector | Applied Mitigation Strategy |
| :--- | :--- | :--- |
| **Spoofing Identity** | Actor attempts to claim another member's synthetic UAN | Identity binding through simulated OTP challenge and session token validation. |
| **Tampering with Data** | Modification of self-attestation dates or salary slip metadata | Immediate cryptographic hashing (SHA-256) of uploaded evidence at intake; hashes are immutably appended to the claim audit trail. |
| **Repudiation** | Employer denies receipt of the 15-day SLA verification notice | Immutable event logging with verifiable UTC timestamps and receipt acknowledgments. |
| **Information Disclosure** | Leakage of bank account details or salary data | Client-side data masking, transport layer encryption (TLS 1.3), and zero backend PII persistence. |
| **Denial of Service** | Flooding pre-flight validation endpoints with automated queries | Rate limiting per IP/session and client-side execution of string-matching heuristics. |
| **Elevation of Privilege** | Citizen attempting to sign off on their own administrative field override | Role-Based Access Control (RBAC) separating Citizen, Employer, and Field Commissioner operations. |

---

## 3. Cryptographic Verification of Attestations

To guarantee evidentiary integrity when a claim auto-escalates to an Assistant PF Commissioner:

1. **Evidence Hashing:** Every supporting document (salary slip, Form 16, bank statement) is hashed using SHA-256 upon selection:
   ```
   H = SHA-256(Document_Byte_Stream)
   ```
2. **Dossier Attestation Manifest:** The system generates a JSON manifest binding the member UAN, declared exit date, timestamp, and document hashes.
3. **Tamper Detection:** Any modification to the evidentiary dossier invalidates the verification signature, immediately flagging the claim for manual field investigation.

---

## 4. Ethical Sandbox and Boundary Notice

### Strict Isolation Policy
- **No Live Government Interfaces:** Nikaasi does NOT connect to live production APIs of EPFO, UIDAI (Aadhaar), Income Tax Department (PAN), or NPCI.
- **Synthetic Test Data:** All citizen personas, bank balances, service records, and employer credentials are generated synthetically for demonstration and architectural evaluation.
- **Independent Status:** Nikaasi is an independent software prototype built for the *Build What Moves India* hackathon. It does not claim official affiliation with or endorsement by the Ministry of Labour and Employment.
