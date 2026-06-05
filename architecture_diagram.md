# Architecture Diagram

The Plum Insurance Automation System is built as a monolithic client-server application, optimized for rapid inference and reliable rule-based processing.

## System Architecture

```mermaid
graph TD
    %% Entities
    User((User / Agent))
    
    %% Frontend Components
    subgraph Frontend [React Frontend (Vite)]
        UI[Web UI]
        State[React State]
        DocUpload[Document Uploader]
        Form[Claim Form]
    end
    
    %% Backend Components
    subgraph Backend [Node.js / Express Backend]
        Router[API Router]
        
        subgraph Services
            DocProc[Document Processor]
            LLM[Gemini Vision & LLM Service]
            Engine[Adjudication Rules Engine]
        end
        
        Data[(Local JSON Mock DB)]
    end
    
    %% External Services
    GoogleAI{{Google Gemini API}}

    %% Flow
    User -->|Interacts with| UI
    UI --> DocUpload
    DocUpload -->|Sends File| Router
    
    Router --> DocProc
    DocProc -->|Image/PDF| LLM
    LLM <-->|API Calls| GoogleAI
    LLM -->|Extracted JSON| Router
    Router -->|Auto-fills| Form
    
    User -->|Submits| Form
    Form -->|Claim Payload| Router
    Router -->|1. Basic Rules| Engine
    Engine -->|Reads Limits/Terms| Data
    
    Router -->|2. Medical Necessity & Fraud| LLM
    LLM <-->|API Calls| GoogleAI
    
    Router -->|3. Final Adjudication| Engine
    Engine -->|Decision JSON| UI
    UI -->|Displays Result| User
```

## Component Breakdown

### 1. React Frontend
- **Framework**: Vite + React + Vanilla CSS
- **Purpose**: Provides a dynamic, glassmorphism-styled UI for claim submission and dashboard viewing.
- **Key Modules**:
  - `SubmitClaim.jsx`: Handles file uploads, form state management, and API orchestration.
  - `ClaimForm.jsx`: Dynamic form that syncs with AI-extracted pre-fill data.
  - `DocumentUpload.jsx`: Drag-and-drop interface.

### 2. Express Backend
- **Framework**: Node.js + Express
- **Purpose**: Acts as the central orchestrator between the frontend, the rules engine, and external AI services.
- **Key Modules**:
  - `claims.js` (Router): Exposes REST endpoints (`/api/claims`, `/api/claims/extract`).
  - `adjudicationEngine.js`: 100% deterministic, step-by-step mathematical rules engine enforcing policy terms.
  - `llmService.js`: Wrapper around the `@google/genai` SDK for multimodal document reading and clinical analysis.

### 3. Data Layer
- **Format**: Local JSON files (`policyTerms.json`, `testCases.json`).
- **Purpose**: Simulates a database to allow the system to run statelessly.

## Performance Optimizations
- **Short-Circuit Execution**: The Express router performs a "dry run" of the deterministic `adjudicationEngine`. If basic eligibility rules fail (e.g., missing doctor registration), the system instantly rejects the claim and entirely skips the expensive/slow Google AI API calls.
