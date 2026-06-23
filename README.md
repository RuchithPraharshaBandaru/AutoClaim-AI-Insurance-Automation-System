# AutoClaim AI - Intelligent OPD Adjudication System

An enterprise-grade, AI-powered full-stack application designed to completely automate the adjudication (approval, rejection, or partial payment) of Outpatient Department (OPD) health insurance claims. 

## The Problem
Traditional health insurance claim processing is highly manual, slow, and error-prone. Claims adjusters spend countless hours cross-referencing messy handwritten doctor prescriptions against complex 100-page policy PDFs to verify coverage limits, exclusions, and waiting periods.

## The Solution: AutoClaim AI
AutoClaim AI acts as a digital "first-pass" claims adjuster. By combining a deterministic, rule-based engine for hard policy constraints with Generative AI (Google Gemini) for contextual analysis, the system can instantly process a claim from document upload to final decision in seconds.

## Core Capabilities

- **Multi-stage Adjudication Engine**
  Evaluates claims through a rigorous 5-step pipeline: Patient Eligibility -> Document Validation -> Coverage & Limits -> Waiting Periods -> Medical Necessity.
- **Multimodal Document Extraction** 
  Users can upload raw images or PDFs of prescriptions/bills. The system uses Vision LLMs to extract structured JSON data (Diagnosis, Medicines, Doctor Reg No.) directly from the documents, ignoring blurry or illegible submissions to prevent AI hallucinations.
- **AI Fraud & Anomaly Detection**
  The LLM cross-references the prescribed medications against the stated diagnosis. If a patient claims Rs.5,000 for "Vitamin C" under a "Viral Fever" diagnosis, the AI automatically flags it as a billing anomaly for MANUAL_REVIEW.
- **RAG-Powered Policy Chatbot**
  A persistent, floating chat widget that uses Retrieval-Augmented Generation to let users ask natural-language questions about their exact coverage limits, waiting periods, and exclusions based on the raw policy JSON.
- **Explainable AI Confidence Scores**
  The system doesn't just output a decision; it provides a human-readable "Confidence Reasoning" paragraph explaining exactly why the AI made that decision, building trust for human auditors.

## Technology Stack

- **Frontend**: React (Vite) with custom glassmorphism CSS, responsive layouts, and interactive dashboards.
- **Backend**: Node.js & Express.js REST API.
- **Artificial Intelligence**: Google Gemini 2.0 Flash (Multimodal Vision & Contextual Analysis).
- **Architecture**: Stateless MVP architecture relying on robust local JSON mock data for rapid deployment and testing without heavy database overhead.

### Adjudication Outcomes

| Decision Type | Trigger Condition |
|----------|-------------|
| **APPROVED** | All rules passed, medications align with diagnosis, claim within annual limits. |
| **REJECTED** | Hard rule failure (e.g., policy expired, unregistered doctor, condition in 30-day waiting period). |
| **PARTIAL** | Core treatment covered, but specific items (e.g., cosmetic supplements) were automatically excluded. |
| **MANUAL_REVIEW** | Fraud detected, blurry documents, or low AI confidence score. Escalated to human adjuster. |

## Deep Dive Documentation

For recruiters or engineers looking to dive into the architecture, please refer to the **[TECHNICAL_DOCUMENTATION.md](./TECHNICAL_DOCUMENTATION.md)** file, which contains:
- Complete System Architecture & Data Flow Diagrams
- Comprehensive API Endpoint Documentation
- The 6-Step Decision Logic Flowchart
- Foundational Assumptions and Policy Rules

## Prerequisites

- **Node.js** 18+
- **Google Gemini API Key** (free tier works) — [Get one here](https://aistudio.google.com/app/apikey)

## Setup & Run

### 1. Clone and install dependencies

```bash
# Install backend dependencies
cd server
npm install

# Install frontend dependencies
cd ../client
npm install
```

### 2. Configure environment variables

```bash
cd server
cp .env.example .env
```

Edit `server/.env` with your credentials:

```env
GEMINI_API_KEY=your_gemini_api_key_here
PORT=5000
```

> **Note:** The application works without the Gemini API key (rule-based only mode), but AI features like fraud detection, confidence reasoning, document extraction, and the policy chatbot require it.

### 3. Start the application

```bash
# Terminal 1 — Start backend
cd server
npm run dev

# Terminal 2 — Start frontend
cd client
npm run dev
```

- **Frontend**: http://localhost:5173
- **Backend API**: http://localhost:5000/api
- **Health Check**: http://localhost:5000/api/health

## Pages

| Page | URL | Description |
|------|-----|-------------|
| Submit Claim | `/` | Submit new claims via form or run test cases |
| Dashboard | `/dashboard` | View all claims with status filters and expandable details |
| Policy | `/policy` | Browse coverage limits, exclusions, and waiting periods |

