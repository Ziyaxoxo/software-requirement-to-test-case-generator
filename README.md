# AI Requirement-to-Test Generator

An AI-assisted system that automatically generates software test cases from requirements and related project artifacts using **Retrieval-Augmented Generation (RAG)** and **Large Language Models (LLMs)**.

Instead of relying on requirement text alone, the system retrieves relevant source code, API documentation, and historical context to generate **unit, integration, security, boundary, and edge-case tests** with full requirement-to-test traceability.

---

## Problem Statement

Manual test design is repetitive, time-consuming, and prone to missing edge cases or security scenarios. Existing AI tools often ignore project context, producing generic, low-value tests.

This project solves that by grounding test generation in **actual project context** source code, API specs, and requirement history , rather than requirement text in isolation.

## Objectives

- Generate context-aware test cases from requirements.
- Improve requirement-to-test traceability.
- Recommend edge, negative, integration, and security tests.
- Reduce QA effort and improve coverage.

---

## Demo / Screenshots

<img width="923" height="512" alt="image" src="https://github.com/user-attachments/assets/1d7f98bc-5b39-4b5e-902f-959664b48187" />
<br>
<img width="925" height="505" alt="image" src="https://github.com/user-attachments/assets/a46cbed2-6a88-413d-b4f2-1c33491390c1" />
<br>
<img width="920" height="500" alt="image" src="https://github.com/user-attachments/assets/1a0ec52e-eb44-4737-83df-ca23598f4fdf" />

<br>

---

## Software Requirement Specification

### Functional Requirements

| ID | Requirement |
|----|-------------|
| FR1 | Upload SRS / user stories |
| FR2 | Upload source code and API documentation |
| FR3 | Retrieve relevant project artifacts using semantic search |
| FR4 | Generate unit, integration, security, boundary, and edge-case tests |
| FR5 | Export generated tests (PDF / CSV / JSON) |
| FR6 | Display traceability between requirement and generated tests |

### Non-Functional Requirements

| Category | Requirement |
|----------|-------------|
| Performance | Response time under 10 seconds for normal projects |
| Security | Secure local storage of project artifacts |
| Scalability | Scalable retrieval using a vector database |
| Maintainability | Modular and maintainable architecture |

---

## System Design & Architecture

### Layered Architecture

```
┌─────────────────────────────────────────────────────────┐
│                   Presentation Layer                    │
│                  React UI / Gradio UI                   │
└───────────────────────────┬─────────────────────────────┘
                             │ REST / Function calls
┌───────────────────────────▼───────────────────────────────┐
│                   Application Layer                       │
│                  FastAPI Backend                          │
└───────────────────────────┬───────────────────────────────┘
                             │
┌───────────────────────────▼───────────────────────────────┐
│                       AI Layer                            │
│      Embedding Model  →  RAG Retrieval  →  LLM Generator  │
└───────────────────────────┬───────────────────────────────┘
                             │
┌───────────────────────────▼───────────────────────────────┐
│                       Data Layer                          │
│         FAISS Vector Database  +  SQLite Metadata         │
└───────────────────────────────────────────────────────────┘
```

### Data Flow / Pipeline

```
 ┌──────────────┐     ┌──────────────┐     ┌──────────────┐     ┌──────────────┐     ┌────────────────────┐
 │ Requirements │ ->  |  Embedding   │ ->  │  RAG / FAISS │ ->  │     LLM      │ ->  │ Generated Tests    │
 │  (SRS input) │     │  Generation  │     │   Retrieval  │     │  Generation  │     │ (Unit/Integration/ │
 └──────────────┘     └──────────────┘     └──────────────┘     └──────────────┘     │  Edge/Boundary/    │
        ▲                                          ▲                                 │  Security)         │
        │                                          │                                 └────────────────────┘
 ┌──────┴───────┐                          ┌───────┴─────-───┐
 │ Source Code  │                          │   API Docs      │
 │ (.zip / repo)│                          │(OpenAPI/Swagger)│
 └──────────────┘                          └────────────-────┘
```

**Workflow:** Requirements → Embedding Generation → Retrieval (FAISS) → LLM → Test Suite → Traceability Report

---

## Tech Stack

| Component | Technology |
|-----------|------------|
| Frontend | React (or Gradio for the Colab/prototype build) |
| Backend | FastAPI (Python) |
| LLM | GPT / Llama / Mistral (swappable, OpenAI-compatible) |
| Embeddings | `bge-large-en-v1.5` or `e5-large-v2` (Sentence-Transformers) |
| Vector DB | FAISS |
| Metadata DB | SQLite |
| Code Parsing | Tree-sitter / Python `ast` / JavaParser |
| API Docs Parsing | OpenAPI / Swagger |
| Export | fpdf2 (PDF), pandas (CSV/JSON) |

---

## Methodology / Workflow

1. **Requirement Preprocessing** : parse SRS into discrete, numbered requirement statements.
2. **Embedding Generation** : embed requirements, code symbols, and API endpoints into a shared vector space.
3. **RAG Retrieval** : retrieve the most relevant artifacts per requirement from the FAISS vector database.
4. **LLM-Based Test Generation** : prompt the LLM per requirement/category to produce structured test cases.
5. **Validation & Formatting** : parse and validate LLM output against a fixed JSON schema.
6. **Export & Review** : display results with traceability, then export to PDF/CSV/JSON.

### Use Case Workflow

1. User uploads requirements and project artifacts.
2. System indexes artifacts.
3. User selects **Generate Tests**.
4. RAG retrieves relevant context.
5. LLM generates categorized test cases.
6. User reviews, edits, and exports results.

---

## Entity Relationship (Textual)

```
Requirement (1..N)        →  Retrieved Artifact
Retrieved Artifact (N..1) →  Test Case
Test Case (N..1)          →  Export Report
```

---

## Getting Started

### Prerequisites

- Python 3.10+
- An OpenAI API key (or a self-hosted OpenAI-compatible endpoint for Llama/Mistral)
- Google Colab (for the notebook build) **or** a local Python environment

### Installation

```bash
git clone https://github.com/<your-username>/ai-requirement-to-test-generator.git
cd ai-requirement-to-test-generator
pip install -r requirements.txt
```

### Environment Variables

Create a `.env` file (or set as environment variables):

```bash
OPENAI_API_KEY=your_api_key_here
```

---

## Usage

### Option A : Google Colab (Notebook)

1. Open `AI_Requirement_to_Test_Generator.ipynb` in Google Colab.
2. Run all cells top to bottom.
3. Use the Gradio interface to:
   - Upload your SRS (PDF/TXT)
   - Upload source code (.zip) and/or OpenAPI spec (optional)
   - Build the FAISS index
   - Generate tests
   - Export as PDF / CSV / JSON

### Option B : Local / FastAPI + React

```bash
# Backend
uvicorn app.main:app --reload

# Frontend
cd frontend
npm install
npm start
```

---

## Project Structure

```
ai-requirement-to-test-generator/
├── app/
│   ├── ingestion/          # SRS, source code, API doc parsers
│   ├── embeddings/         # Embedding + FAISS index management
│   ├── retrieval/          # RAG retrieval logic
│   ├── generation/         # Prompt templates + LLM orchestration
│   ├── export/             # PDF / CSV / JSON export utilities
│   └── main.py             # FastAPI entry point
├── frontend/                # React UI
├── notebooks/
│   └── AI_Requirement_to_Test_Generator.ipynb
├── docs/
│   └── screenshots/         # Add UI screenshots here
├── requirements.txt
└── README.md
```

---

## Expected Outputs

-  Unit test scenarios
-  Integration test scenarios
-  Security test scenarios
-  Boundary and edge-case tests
-  Requirement-to-test traceability report

---

## Future Enhancements

-  GitHub integration (pull code directly from a repo URL)
-  CI/CD pipeline support
-  Automatic test script generation (JUnit / PyTest)
-  Test coverage estimation
-  Learning from accepted/rejected test cases (feedback loop)

---

## Contributing

Contributions are welcome. Please open an issue to discuss proposed changes before submitting a pull request.

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/your-feature`)
3. Commit your changes
4. Open a pull request

---
