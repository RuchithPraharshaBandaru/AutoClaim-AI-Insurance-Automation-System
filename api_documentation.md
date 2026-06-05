# API Documentation

The Plum Insurance Automation System exposes a set of RESTful API endpoints for the frontend application to interact with the backend services.

## Base URL
```
http://localhost:5000/api
```

---

## 1. Extract Document Data
Extracts structured information (patient details, medicines, procedures, bill amounts) from raw uploaded files using Google Gemini's multimodal vision models.

**Endpoint:** `POST /claims/extract`
**Content-Type:** `multipart/form-data`

### Request Parameters
- `files`: Array of files (images/PDFs) to process. Max 5 files.

### Success Response (200 OK)
```json
{
  "success": true,
  "message": "Extracted details from 2 documents",
  "formData": {
    "memberName": "Sneha Reddy",
    "doctorName": "Dr. Gupta",
    "doctorReg": "KA/45678/2015",
    "diagnosis": "Viral fever",
    "medicines": ["Paracetamol 650mg", "Vitamin C"],
    "consultationFee": 1000,
    "medicinesCost": 500,
    "claimAmount": 1500,
    "warnings": []
  }
}
```

### Error Response (400 / 500)
```json
{
  "error": "Failed to parse document: Images are illegible"
}
```

---

## 2. Submit Claim for Adjudication
Evaluates a normalized claim payload against the deterministic policy rules and performs LLM-driven medical necessity and fraud checks.

**Endpoint:** `POST /claims`
**Content-Type:** `application/json`

### Request Body
```json
{
  "memberId": "EMP001",
  "memberName": "Sneha Reddy",
  "memberJoinDate": "2024-01-01",
  "treatmentDate": "2024-10-15",
  "claimAmount": 1500,
  "hospital": "Apollo Clinic",
  "cashlessRequest": false,
  "previousClaimsSameDay": 0,
  "doctorName": "Dr. Gupta",
  "doctorReg": "KA/45678/2015",
  "diagnosis": "Viral fever",
  "treatment": "Outpatient consultation",
  "medicines": "Paracetamol, Vitamin C",
  "procedures": "",
  "tests": "",
  "consultationFee": 1000,
  "diagnosticTests": 0,
  "medicinesCost": 500,
  "otherCharges": 0,
  "otherChargesLabel": "",
  "rawBillItems": []
}
```

### Success Response (200 OK)
```json
{
  "success": true,
  "claim": {
    "claim_id": "CLM_8A9B1C2D",
    "decision": "APPROVED",
    "approvedAmount": 1500,
    "rejectionReasons": [],
    "rejectedItems": [],
    "confidenceScore": 0.98,
    "notes": "Medical necessity verified. Standard prescription.",
    "nextSteps": "Payment will be processed in 3-5 business days.",
    "flags": []
  }
}
```

---

## 3. Run Pre-Configured Test Case
Executes the adjudication engine against one of the pre-configured test cases (TC001-TC010) and compares the actual output against the expected output.

**Endpoint:** `POST /claims/test/:caseId`

### Path Parameters
- `caseId`: The ID of the test case to run (e.g., `TC001`)

### Success Response (200 OK)
```json
{
  "success": true,
  "match": true,
  "testCase": {
    "case_id": "TC001",
    "case_name": "Clean Claim - Approved",
    "description": "Standard consultation and pharmacy bill"
  },
  "expectedResult": {
    "decision": "APPROVED",
    "approved_amount": 7000
  },
  "actualResult": {
    "decision": "APPROVED",
    "approvedAmount": 7000,
    "rejectionReasons": [],
    "confidenceScore": 0.95
  }
}
```

---

## 4. Get Policy Rules
Fetches the current policy rules and limits for display on the frontend.

**Endpoint:** `GET /policy/rules`

### Success Response (200 OK)
Returns the full contents of `policyTerms.json`.
