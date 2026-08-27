# Nikaasi Technical Specification

## Document Control

- **Document Version:** 1.0.0
- **Status:** Approved Technical Architecture
- **Classification:** Engineering & Algorithmic Specification

---

## 1. Algorithmic Models and Heuristics

### 1.1 Transliteration and Identity Matching Algorithm

Name mismatches between Aadhaar (often issued in English transliterated from regional scripts like Devanagari, Tamil, or Bengali) and EPFO records account for the largest share of Class A rejections.

Nikaasi uses a hybrid matching pipeline combining **Phonetic Normalization (Double Metaphone / Indian Phonetic Rule Adaptations)** and **Jaro-Winkler String Distance**:

```
Input:
  s1 = Aadhaar Name (e.g., "ARCHITT MITTAL")
  s2 = EPFO UAN Name (e.g., "ARCHIT MITTAL")

Pipeline:
  1. Pre-processing:
     - Upper-case conversion
     - Removal of honorifics (Shri, Smt, Mr, Dr, Late)
     - Strip excess whitespace and punctuation
     - Standardize common abbreviation expansions (e.g., "KUMAR" <-> "KR")

  2. Phonetic Hashing:
     - Code1 = DoubleMetaphone(s1)  // "ARKT MTL"
     - Code2 = DoubleMetaphone(s2)  // "ARKT MTL"

  3. Distance Metric:
     - d_jw = JaroWinklerDistance(s1, s2)
     
  4. Classification Matrix:
     - If s1 == s2:
         Status = EXACT_MATCH (Confidence = 1.0)
     - Else if Code1 == Code2 and d_jw >= 0.88:
         Status = PHONETIC_MATCH (Confidence = d_jw) -> Class A Auto-Remediation Recommended
     - Else if d_jw >= 0.82:
         Status = PARTIAL_MISMATCH (Confidence = d_jw) -> Step-by-Step Joint Declaration Flow
     - Else:
         Status = HARD_MISMATCH -> Formal Correction Mandatory
```

### 1.2 Multi-UAN Timeline Merging Algorithm

When a member has accumulated multiple UANs across different employers, the system checks for overlapping service periods and generates a consolidation sequence:

```
Algorithm: ValidateAndMergeUANHistory(MemberRecord)
  Input: List of Employment Records E = [e_1, e_2, ..., e_n]
  Sort E by JoiningDate ascending
  
  For i from 1 to n-1:
    If E[i].ExitDate is NULL and i < n:
      Flag ANOMALY_MISSING_EXIT_DATE on E[i]
    Else if E[i].ExitDate > E[i+1].JoiningDate:
      Flag ANOMALY_OVERLAPPING_SERVICE between E[i] and E[i+1]
      
  Generate UAN Merge Package (Form 13 / Transfer Request)
```

---

## 2. SLA State Machine Engine

The SLA engine tracks the 15-day employer countdown and coordinates automatic jurisdictional escalation.

### 2.1 State Definitions

| State Name | Description | Active Actor |
| :--- | :--- | :--- |
| `DRAFT` | Claim intake initiated by member | Citizen |
| `PREFLIGHT_PASSED` | Cross-validation verified with zero blocking errors | System |
| `ATTESTATION_REQUIRED` | Date of Exit absent; evidence submission requested | Citizen |
| `ATTESTATION_SUBMITTED` | Evidence uploaded and cryptographically hashed | Citizen / System |
| `EMPLOYER_SLA_ACTIVE` | 15-day statutory verification notice dispatched | Employer HR |
| `EMPLOYER_ACKNOWLEDGED` | Employer confirms exit date within SLA | Employer HR |
| `SLA_BREACHED_ESCALATED` | 15 days elapsed without employer response; routed to Field Office | System / EPFO RPFC |
| `COMMISSIONER_APPROVED` | Regional Field Office approves via statutory override | Assistant PF Commissioner |
| `SETTLEMENT_DISBURSED` | Direct credit issued to member's verified bank account | Payment Gateway / NPCI |

### 2.2 Transition Table

```
+------------------------+--------------------------+----------------------------+
| Current State          | Trigger Event            | Next State                 |
+------------------------+--------------------------+----------------------------+
| DRAFT                  | PREFLIGHT_VALIDATE_OK    | PREFLIGHT_PASSED           |
| PREFLIGHT_PASSED       | DOE_MISSING_DETECTED     | ATTESTATION_REQUIRED       |
| ATTESTATION_REQUIRED   | SUBMIT_EVIDENCE_BUNDLE   | ATTESTATION_SUBMITTED      |
| ATTESTATION_SUBMITTED  | DISPATCH_EMPLOYER_NOTICE | EMPLOYER_SLA_ACTIVE        |
| EMPLOYER_SLA_ACTIVE    | EMPLOYER_CONFIRMS        | EMPLOYER_ACKNOWLEDGED      |
| EMPLOYER_SLA_ACTIVE    | SLA_CLOCK_EXPIRES (D+15) | SLA_BREACHED_ESCALATED     |
| SLA_BREACHED_ESCALATED | COMMISSIONER_OVERRIDE    | COMMISSIONER_APPROVED      |
| COMMISSIONER_APPROVED  | INITIATE_PAYOUT          | SETTLEMENT_DISBURSED       |
| EMPLOYER_ACKNOWLEDGED  | INITIATE_PAYOUT          | SETTLEMENT_DISBURSED       |
+------------------------+--------------------------+----------------------------+
```

---

## 3. Data Schemas and Interfaces (TypeScript / JSON Schema)

### 3.1 Member Profile Schema

```typescript
export interface MemberProfile {
  uan: string; // 12-digit synthetic UAN
  maskedAadhaar: string; // "XXXX-XXXX-1234"
  pan: string; // "ABCDE1234F"
  fullNameAadhaar: string;
  fullNameEPFO: string;
  dateOfBirth: string; // ISO 8601 YYYY-MM-DD
  gender: 'MALE' | 'FEMALE' | 'OTHER';
  bankAccount: {
    accountNumberMasked: string; // "XXXXXXXX5678"
    ifsc: string;
    bankName: string;
    isNpciSeeded: boolean;
    isPennyDropVerified: boolean;
    accountHolderName: string;
  };
  employments: Array<{
    establishmentId: string;
    establishmentName: string;
    joiningDate: string;
    exitDate: string | null;
    isExitDateVerified: boolean;
    totalContributionMonths: number;
    epfBalance: number;
    epsBalance: number;
  }>;
}
```

### 3.2 Pre-Flight Diagnostic Result Schema

```typescript
export interface DiagnosticResult {
  overallStatus: 'PASSED' | 'WARNING' | 'ACTION_REQUIRED';
  timestamp: string;
  scores: {
    identityConfidence: number; // 0.0 to 1.0
    bankSeedingConfidence: number;
    serviceContinuityConfidence: number;
  };
  discrepancies: Array<{
    code: string;
    category: 'CLASS_A' | 'CLASS_B';
    field: string;
    sourceValue: string;
    targetValue: string;
    severity: 'BLOCKING' | 'NON_BLOCKING';
    description: string;
    remediationSteps: string[];
  }>;
  eligibleClaims: Array<{
    formType: 'FORM_19' | 'FORM_10C' | 'FORM_31';
    paragraph?: string;
    maxEligibleAmount: number;
    turnaroundEstimateDays: number;
  }>;
}
```

### 3.3 Attestation and SLA Dossier Schema

```typescript
export interface AttestationDossier {
  dossierId: string;
  claimId: string;
  memberUAN: string;
  establishmentId: string;
  declaredExitDate: string;
  submittedAt: string;
  slaDeadline: string;
  supportingDocuments: Array<{
    documentId: string;
    documentType: 'SALARY_SLIP' | 'FORM_16' | 'BANK_STATEMENT' | 'RESIGNATION_ACCEPTANCE';
    fileName: string;
    sha256Checksum: string;
    uploadTimestamp: string;
  }>;
  auditTrail: Array<{
    stepIndex: number;
    timestamp: string;
    actor: 'MEMBER' | 'SYSTEM' | 'EMPLOYER' | 'FIELD_OFFICE';
    action: string;
    holdingEntity: string;
    notes?: string;
  }>;
}
```

---

## 4. Sandbox Personas for Demonstration

| Persona ID | Name | Case Classification | Scenario Parameters |
| :--- | :--- | :--- | :--- |
| `PERSONA_01` | Rajesh Kumar | Clean Record (Happy Path) | Full match across Aadhaar, PAN, Bank. Exit date marked. Eligible for Form 19 & 10C. |
| `PERSONA_02` | Sunita Devi | Class A: Transliteration Gap | Aadhaar: "SUNITA DEVI", EPFO: "SUNITHA DEVI". Jaro-Winkler = 0.93. Bank seeding verified. |
| `PERSONA_03` | Amit Verma | Class B: Missing Date of Exit | Exit Date is NULL; Company "TechnoSoft Ltd" unresponsive. Triggers Self-Attestation & 15-Day SLA clock. |
| `PERSONA_04` | Priya Sharma | Fragmented Multi-UAN | 2 separate UANs across previous jobs. Service gap detected. Form 13 consolidation workflow triggered. |
