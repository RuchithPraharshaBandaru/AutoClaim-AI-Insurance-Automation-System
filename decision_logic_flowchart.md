# Decision Logic Flowchart

The Adjudication Engine enforces a deterministic, priority-ordered pipeline. The system evaluates claims from top to bottom. If a rejection condition is met at any phase, the engine short-circuits and immediately returns a `REJECTED` decision, bypassing any subsequent checks.

## Flowchart

```mermaid
flowchart TD
    Start([Receive Claim Payload]) --> FraudCheck

    %% Step 0: Priority Fraud Check
    FraudCheck{0. Check Basic Fraud Patterns}
    FraudCheck -->|High Claim Vol/Val| ManualReview([MANUAL_REVIEW])
    FraudCheck -->|Safe| EligibilityCheck

    %% Step 1: Eligibility Check
    EligibilityCheck{1. Eligibility & Policy Status}
    EligibilityCheck -->|No Member ID/Name| Reject1([REJECTED: MEMBER_NOT_COVERED])
    EligibilityCheck -->|Late Submission| Reject1([REJECTED: LATE_SUBMISSION])
    EligibilityCheck -->|In Waiting Period| Reject1([REJECTED: WAITING_PERIOD])
    EligibilityCheck -->|Valid| DocumentCheck

    %% Step 2: Document Check
    DocumentCheck{2. Document Validation}
    DocumentCheck -->|No Prescription / Bill| Reject2([REJECTED: MISSING_DOCUMENTS])
    DocumentCheck -->|No Doctor Reg No.| Reject2([REJECTED: DOCTOR_REG_INVALID])
    DocumentCheck -->|Dates Don't Match| Reject2([REJECTED: DATE_MISMATCH])
    DocumentCheck -->|Patient Name Mismatch| Reject2([REJECTED: PATIENT_MISMATCH])
    DocumentCheck -->|Valid| CoverageCheck

    %% Step 3: Coverage Verification
    CoverageCheck{3. Coverage & Exclusions}
    CoverageCheck -->|Condition in Exclusions| Reject3([REJECTED: EXCLUDED_CONDITION])
    CoverageCheck -->|Alternative Med Not Allowed| Reject3([REJECTED: SERVICE_NOT_COVERED])
    CoverageCheck -->|Valid| LimitCheck

    %% Step 4: Limit Validation
    LimitCheck{4. Financial Limits}
    LimitCheck -->|Exceeds Annual Limit| Reject4([REJECTED: ANNUAL_LIMIT_EXCEEDED])
    LimitCheck -->|Exceeds Per-Claim Limit| Reject4([REJECTED: PER_CLAIM_EXCEEDED])
    LimitCheck -->|Exceeds Category Sub-Limit| Reject4([REJECTED: SUB_LIMIT_EXCEEDED])
    LimitCheck -->|Below Min ₹500 Amount| Reject4([REJECTED: BELOW_MIN_AMOUNT])
    LimitCheck -->|Valid| PartialCheck

    %% Partial Approval Check
    PartialCheck{Contains Excluded Items?}
    PartialCheck -->|Yes| SetPartial[Deduct items, Flag as PARTIAL]
    PartialCheck -->|No| KeepApproved[Keep as APPROVED]

    %% Final LLM Step (Only reached if not completely rejected)
    SetPartial --> LLMCheck
    KeepApproved --> LLMCheck

    LLMCheck{5. Medical Necessity (LLM)}
    LLMCheck -->|Not Justified| Reject5([REJECTED: NOT_MEDICALLY_NECESSARY])
    LLMCheck -->|Justified| FinalCalculation

    FinalCalculation[6. Calculate Network Discounts] --> Output([Final Decision Output])
```

## Priority Short-Circuiting
The Express router is optimized to run the deterministic checks (Steps 0-4) *before* making any external API requests. 
If any of those fast mathematical/logic checks trigger a `REJECTED` path, the entire pipeline is instantly aborted and returned to the user, completely avoiding the latency and cost of the LLM execution (Step 5).
