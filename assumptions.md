# List of Assumptions

Given the scope of the MVP and the absence of a live relational database, the Plum Insurance Automation System operates under the following key assumptions:

### 1. The React Form Acts as the "Database"
Since there is no live SQL database to store member or policy data, the system treats the React form inputs as the absolute source of truth for the policy record. 
- If a user uploads a prescription but the `Member Name` field is left blank, the system assumes the user does not exist in the database and throws a `MEMBER_NOT_COVERED` error.
- The `PATIENT_MISMATCH` rule compares the name extracted from the physical document against the name typed into the form.

### 2. Generative AI Consistency
- The multimodal document processor relies heavily on Google's Gemini Flash 2.0 API. We assume the API will return JSON in a structured, relatively consistent format. The `documentProcessor.js` utilizes heavy fallback logic to coerce the text if the LLM drops the schema wrapper, but there is still an assumption of general formatting adherence.

### 3. Date Parsing
- The backend assumes that treatment dates and member join dates are passed in standard ISO or `YYYY-MM-DD` formats from the frontend. It calculates time differences (e.g., 30-day submission windows, 90-day waiting periods) assuming UTC or local server time offsets will not create edge-case off-by-one errors.

### 4. Co-Payments & Deductions
- The system currently calculates co-payments (e.g., 20% on certain procedures) and applies them directly to the `approvedAmount`. We assume the user understands that `approvedAmount` is the final payout to the hospital/patient, and the remaining 20% is their out-of-pocket responsibility.

### 5. Deterministic Limits Override Contextual Decisions
- As per the priority rules, hard limits cannot be exceeded. If a perfectly clean, highly-necessary medical procedure costs ₹8000 but the per-claim limit is ₹5000, the system aggressively truncates or rejects the claim regardless of the LLM's medical necessity validation.
