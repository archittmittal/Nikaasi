# Nikaasi API Reference

## API Specification

- **API Version:** v1.0.0
- **Base URI:** `/api/v1`
- **Protocol:** HTTPS / JSON REST
- **Authentication:** Simulated Session Bearer Token / Sandbox Mock Persona Header

---

## Standard Headers

| Header Name | Type | Required | Description |
| :--- | :--- | :--- | :--- |
| `Content-Type` | String | Yes | Must be `application/json` |
| `X-Sandbox-Persona-Id` | String | No | Target mock persona identifier (e.g., `PERSONA_01`, `PERSONA_03`) |
| `Idempotency-Key` | String | Optional | UUIDv4 string to prevent duplicate state submissions |

---

## Endpoints

### 1. Pre-Flight Cross-Validation

#### `POST /api/v1/preflight/validate`
Executes pre-flight cross-referencing across simulated Aadhaar, PAN, and Banking records.

##### Request Payload
```json
{
  "uan": "100928374821",
  "fullName": "Archit Mittal",
  "aadhaarMasked": "XXXX-XXXX-8921",
  "pan": "ABCDE1234F",
  "bankIfsc": "HDFC0001234",
  "bankAccountNumber": "50100293847581"
}
```

##### Response (`200 OK`)
```json
{
  "status": "SUCCESS",
  "diagnosticResult": {
    "overallStatus": "WARNING",
    "scores": {
      "identityConfidence": 0.94,
      "bankSeedingConfidence": 1.0,
      "serviceContinuityConfidence": 0.75
    },
    "discrepancies": [
      {
        "code": "DISC_CLASS_B_DOE_MISSING",
        "category": "CLASS_B",
        "field": "employments[0].exitDate",
        "severity": "BLOCKING",
        "description": "Previous employer (InfraCorp India) has not updated the Date of Exit.",
        "remediationSteps": [
          "Initiate Exit-Date Self-Attestation with salary slip / Form 16.",
          "Trigger 15-day employer SLA clock."
        ]
      }
    ],
    "eligibleClaims": [
      {
        "formType": "FORM_19",
        "maxEligibleAmount": 348500,
        "turnaroundEstimateDays": 3
      }
    ]
  }
}
```

---

### 2. Conversational Intent Classification

#### `POST /api/v1/intake/classify-intent`
Translates natural language statements into statutory EPFO form and paragraph selections.

##### Request Payload
```json
{
  "memberStatement": "I resigned two months ago from my company and need to withdraw my entire provident fund and pension savings.",
  "language": "en"
}
```

##### Response (`200 OK`)
```json
{
  "status": "SUCCESS",
  "classification": {
    "intentType": "FULL_FINAL_SETTLEMENT",
    "primaryForm": "FORM_19",
    "secondaryForms": ["FORM_10C"],
    "statutoryParagraph": null,
    "confidenceScore": 0.98,
    "statutoryConditionsMet": {
      "twoMonthsUnemployed": true,
      "serviceOver6Months": true,
      "serviceUnder10Years": true
    }
  }
}
```

---

### 3. Exit-Date Self-Attestation Submission

#### `POST /api/v1/attestation/submit`
Submits evidentiary documentation to establish the member's exit date when omitted by an employer.

##### Request Payload
```json
{
  "claimId": "clm_del_2026_09182",
  "uan": "100928374821",
  "establishmentId": "DLCPM0019283000",
  "declaredExitDate": "2026-05-31",
  "evidenceDocuments": [
    {
      "documentType": "FORM_16",
      "fileName": "Form16_FY2025_26.pdf",
      "sha256Checksum": "e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855"
    },
    {
      "documentType": "SALARY_SLIP",
      "fileName": "SalarySlip_May2026.pdf",
      "sha256Checksum": "f2ca1bb6c7e907d06dafe4687e579fce76b37e4e93b7605022da52e6ccc26fd2"
    }
  ]
}
```

##### Response (`201 Created`)
```json
{
  "status": "SUCCESS",
  "dossierId": "dos_att_2026_48291",
  "state": "EMPLOYER_SLA_ACTIVE",
  "slaClock": {
    "startedAt": "2026-08-27T08:15:30.000Z",
    "expiresAt": "2026-09-11T08:15:30.000Z",
    "totalDays": 15,
    "daysRemaining": 15
  },
  "holdingEntity": "InfraCorp India Private Limited (HR Operations)",
  "escalationDestination": "EPFO Regional Office Delhi (South)"
}
```

---

### 4. Claim Tracking and Observability

#### `GET /api/v1/claims/{claimId}/status`
Retrieves granular lifecycle tracking data for a specific claim.

##### Response (`200 OK`)
```json
{
  "claimId": "clm_del_2026_09182",
  "currentState": "EMPLOYER_SLA_ACTIVE",
  "holdingDesk": {
    "entityName": "InfraCorp India Private Limited",
    "deskType": "EMPLOYER_HR_VERIFICATION",
    "contactEmail": "hr-nodal@infracorp.mock",
    "slaRemainingDays": 6
  },
  "escalationSchedule": {
    "autoEscalateAt": "2026-09-11T08:15:30.000Z",
    "targetOffice": "RPFC Delhi South - Sector 4",
    "statutoryRule": "Section 26B Admin Override Framework"
  },
  "eventHistory": [
    {
      "sequence": 1,
      "timestamp": "2026-08-27T08:00:00.000Z",
      "event": "PREFLIGHT_CROSS_VALIDATION_COMPLETED",
      "actor": "CITIZEN"
    },
    {
      "sequence": 2,
      "timestamp": "2026-08-27T08:15:30.000Z",
      "event": "SELF_ATTESTATION_SUBMITTED",
      "actor": "CITIZEN"
    },
    {
      "sequence": 3,
      "timestamp": "2026-08-27T08:16:00.000Z",
      "event": "EMPLOYER_NOTICE_DISPATCHED",
      "actor": "SYSTEM"
    }
  ]
}
```

---

### 5. Sandbox Time-Travel Simulator

#### `POST /api/v1/sandbox/sla/advance-time`
Advances the virtual sandbox timeline to test SLA expiration and automatic field office escalation.

##### Request Payload
```json
{
  "claimId": "clm_del_2026_09182",
  "advanceDays": 16
}
```

##### Response (`200 OK`)
```json
{
  "status": "SUCCESS",
  "previousState": "EMPLOYER_SLA_ACTIVE",
  "newState": "SLA_BREACHED_ESCALATED",
  "daysAdvanced": 16,
  "message": "SLA expired without employer rebuttal. Claim dossier auto-escalated to EPFO Regional Office Commissioner."
}
```

---

## Standard Error Response Model

```json
{
  "error": {
    "code": "ERR_INVALID_UAN_FORMAT",
    "message": "The provided UAN must be exactly 12 numeric digits.",
    "timestamp": "2026-08-27T08:15:30.000Z",
    "requestId": "req_88192301"
  }
}
```

| HTTP Code | Error Code | Description |
| :--- | :--- | :--- |
| `400 Bad Request` | `ERR_SCHEMA_VALIDATION` | Payload fields failed schema validation |
| `404 Not Found` | `ERR_CLAIM_NOT_FOUND` | Target claim or UAN record does not exist |
| `409 Conflict` | `ERR_DUPLICATE_CLAIM` | An active claim for this UAN is already in progress |
| `422 Unprocessable` | `ERR_INELIGIBLE_CLAIM` | Statutory eligibility criteria not satisfied |
