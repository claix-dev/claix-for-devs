# Claix API & SDK — AI Document Intelligence & Memory for Agents

Claix is a high-performance, server-to-server Document Intelligence API and Agent Memory substrate. It transforms unstructured documents (PDFs, Excel workbooks, Word docs, images, raw text/HTML) into strictly typed, validated JSON and provides persistent, queryable context for autonomous AI agents.

Built for backend microservices, agentic workflows (LangGraph, CrewAI, LlamaIndex), the **Agent2Agent (A2A) protocol**, and automation engines like n8n, Make, and Zapier.

🌐 **Website:** [claix.dev](https://claix.dev)  
📄 **OpenAPI Spec (v1.8.2):** [claix.dev/openapi.yaml](https://claix.dev/openapi.yaml)  
📦 **Python SDK (PyPI):** `pip install claix`

---

## ⚡ Key Highlights
* **Multimodal Visual Extraction:** Zero-template extraction from scanned PDFs, distorted receipt images, and multi-sheet Excel files.
* **Deterministic Structured Outputs:** 100% schema compliance. Fields with missing document evidence return native `null` instead of hallucinations.
* **Knowledge Spaces (`space_id`):** Group up to 50 active documents under a unified space to perform cross-document reasoning, multi-file totals, and contract-to-invoice reconciliation.
* **Decoupled Document Context (`document_id`):** Ingest once, query on demand via lightweight REST endpoints—reducing input token costs by over 80%.
* **Agent2Agent (A2A) Protocol Ready:** Exposes standard Agent Cards (`/.well-known/agent-card.json`) and task contracts for cross-vendor multi-agent swarms.
* **First-Class Python SDK:** Modular tool adapters for LangChain, LangGraph, CrewAI, and LlamaIndex.

---

## 📑 Table of Contents
* [Authentication](#-authentication)
* [Python SDK & Agent Frameworks](#-python-sdk--agent-frameworks)
* [API Endpoints Overview](#-api-endpoints-overview)
  * [1. Structured Extraction Endpoints](#1-structured-extraction-endpoints)
  * [2. Agent Mode (Semantic Reasoning)](#2-agent-mode-semantic-reasoning)
  * [3. Context Window (Single Document QA)](#3-context-window-single-document-qa)
  * [4. Knowledge Spaces (Cross-Document Multi-File QA)](#4-knowledge-spaces-cross-document-multi-file-qa)
  * [5. Schema Management](#5-schema-management)
* [🤖 Agent2Agent (A2A) Protocol Integration](#-agent2agent-a2a-protocol-integration)
* [Response Structures & Deterministic Nulls](#-response-structures--deterministic-nulls)
* [Error Handling](#-error-handling)
* [Quick Code Examples](#-quick-code-examples)

---

## 🔒 Authentication

All API calls require your private API key provided via one of the following headers:
* **Dedicated Header (Recommended):** `x-api-key: YOUR_API_KEY`
* **Standard Bearer Token:** `Authorization: Bearer YOUR_API_KEY`

---

## 📦 Python SDK & Agent Frameworks

Install the unified official Python package with optional extras:

```bash
# Core SDK (httpx + Pydantic v2)
pip install claix

# With LangChain & LangGraph support
pip install claix[langchain]

# With CrewAI support
pip install claix[crewai]

# With all agent integrations (LangGraph, CrewAI, LlamaIndex)
pip install claix[all]
```

### Quickstart Example (Python)

```python
from claix import ClaixClient

client = ClaixClient(api_key="YOUR_API_KEY")

# 1. Extract typed JSON from a complex PDF
result = client.extract.pdf(
    file="invoice_2026.pdf",
    schema_id="3c7a9f21-4b8e-4d1a-9c6f-2e0d8a5b7c4f",
    space_id="5b9e2c14-7d3a-4f8b-9e1c-6a0d4b8f2e7c"  # Optional: group into a Space
)
print(result.data)

# 2. Query the Knowledge Space (Cross-document reasoning)
space_answer = client.spaces.ask(
    space_id="5b9e2c14-7d3a-4f8b-9e1c-6a0d4b8f2e7c",
    questions=["Which vendor billed the highest total amount across all invoices?"]
)
print(space_answer.ia_response)
```

---

## 🚀 API Endpoints Overview

Base URL: `https://claix.dev/api` *(except where explicit URLs are specified)*.

### 1. Structured Extraction Endpoints

| Endpoint | Method | Input Formats | Max Limits | Description |
|---|---|---|---|---|
| `/excel-json` | `POST` | `.xlsx`, `.xls`, `.csv` | 1st sheet | Transforms spreadsheet rows into an array of typed JSON records conforming to your schema. |
| `/json-excel` | `POST` | JSON array / envelope | — | Generates a binary `.xlsx` spreadsheet matching the schema column definitions. |
| `/pdf-json` | `POST` | `.pdf` (text/scanned) | 15 MB | Multimodal analysis returning a single structured record. Handles visual layouts and tables. |
| `/doc-json` | `POST` | `.docx`, `.txt`, `.md`, `.rtf` | 10 MB / 300k chars | Deterministic text extraction before AI inference. *(Legacy `.doc` is not supported).* |
| `/img-json` | `POST` | `.jpeg`, `.jpg`, `.png`, `.webp`, `.heic`, `.heif` | 15 MB | Multimodal OCR and visual comprehension. Returns `422` if blurry or unreadable. |
| `/txt-json` | `POST` | Raw text, HTML, XML | 300k chars | Schema-based structured extraction from raw text passed directly in the payload. |

---

### 2. Agent Mode (Semantic Reasoning)

Agent Mode executes structured extraction **followed by a semantic reasoning phase** powered by your custom business rules.

* **Endpoints:**
  * `POST https://claix.dev/agent/excel-json`
  * `POST https://claix.dev/agent/pdf-json`
  * `POST https://claix.dev/agent/doc-json`
  * `POST https://claix.dev/agent/img-json`
  * `POST https://claix.dev/agent/txt-json`
* **Requirements:** Schema must have `is_agent_mode: true` and a configured `agent_definition`.
* **Output:** Returns the standard `data` array **plus** an `agent_data` object with typed boolean flags, integer evaluations, enums, or textual summaries.

---

### 3. Context Window (Single Document QA)

When an extraction is executed with context window enabled, Claix stores the processed Markdown representation, returning a persistent `document_id`.

* **Query Document:** `POST https://claix.dev/document-context/{document_id}`
  * Body: `{ "questions": ["Question 1", "Question 2"] }` (Max 5 questions, max 400 chars each).
  * Returns: `{ "user_ask": [...], "ia_response": [...] }`.
* **Get Raw Content:** `GET https://claix.dev/get-document/{document_id}` *(Free)*.
* **Delete Context:** `DELETE https://claix.dev/delete-document/{document_id}` *(Free)*.

---

### 4. Knowledge Spaces (Cross-Document Multi-File QA)

Knowledge Spaces allow multiple active documents to be grouped under a single `space_id` for cross-file comparison, reconciliation, and aggregations.

* **Create Space:** `POST https://claix.dev/create-space`  
  * Body: `{ "name": "Vendor Operations 2026" }` ➔ Returns `space_id`. *(Free)*.
* **Assign Docs to Space:** Pass `space_id` as an optional parameter when calling any `/api/*-json` or `/agent/*-json` endpoint.
* **Query Space (Cross-Document QA):** `POST https://claix.dev/space-context/{space_id}`
  * Evaluates up to **50 active documents** and **200,000 characters of context** simultaneously.
  * Answers multi-file questions: *"Do the totals in invoice_march.pdf match the contracted budget in contract_omega.pdf?"*.
* **Delete Space:** `DELETE https://claix.dev/delete-space/{space_id}` *(Free)*.

---

### 5. Schema Management

* **List Schemas:** `GET https://claix.dev/api/schemas` *(Free)*.
* **Create Schema:** `POST https://claix.dev/api/create-schema` *(Free)*.
* **Delete Schema:** `POST https://claix.dev/api/delete-schema` or `DELETE https://claix.dev/api/delete-schema?schema_id={uuid}` *(Free)*.

---

## 🤖 Agent2Agent (A2A) Protocol Integration

Claix natively implements the open **Agent2Agent (A2A)** specification, allowing orchestrators (such as LangGraph, CrewAI, AutoGen, and Microsoft Agent Framework) to discover skills and delegate document extraction tasks via standard JSON-RPC 2.0 / HTTPS task lifecycles.

* **Public Agent Card:** `https://claix.dev/.well-known/agent-card.json`
* **Exposed Skills:**
  * `extract_document`: Schema-constrained extraction from binary files.
  * `query_document_context`: Targeted single-file interrogation (`document_id`).
  * `cross_document_reasoning`: Multi-file cross-referencing and validation (`space_id`).

---

## 📋 Response Structures & Deterministic Nulls

### Standard Extraction Payload

```json
{
  "success": true,
  "schema_utilizado": "Facturas de Proveedores",
  "total_registros": 1,
  "data": [
    {
      "numero_factura": "F-2026-00456",
      "fecha_emision": "2026-03-14",
      "fecha_vencimiento": null,
      "proveedor": "Suministros Industriales S.L.",
      "importe_total": 1284.50
    }
  ]
}
```

### Knowledge Space Cross-Document Response

```json
{
  "user_ask": [
    "Which supplier billed the highest total amount?",
    "Is there any contract whose price does not match its invoice?"
  ],
  "ia_response": [
    "Suministros Omega S.A., with 48,320 € across three invoices.",
    "Yes: contract_omega.pdf specifies 12,000 € but invoice_march.pdf charges 13,450 €."
  ]
}
```
> **Deterministic Nulls:** If a requested field or question has no direct textual evidence in the documents, Claix returns a native `null` rather than hallucinating plausible values.

---

## 🚨 Error Handling

Claix uses standard HTTP status codes:

| Status Code | Reason & Troubleshooting |
|---|---|
| `400 Bad Request` | Missing file, invalid JSON, exceeding 5 questions, or schema without `is_agent_mode`. |
| `401 Unauthorized` | Missing, inactive, or suspended API key. |
| `404 Not Found` | Schema, Document ID, or Space ID does not exist or belongs to another account. |
| `405 Method Not Allowed` | Incorrect HTTP method (e.g., using `GET` instead of `POST`). |
| `413 Payload Too Large` | Exceeds file limits (10 MB for Docs, 15 MB for PDF/Img) or 300k characters. |
| `422 Unprocessable Entity` | Illegible/blurry image, or no matching data found for the given schema. |
| `500 Internal Server Error` | Claix platform server error. |
| `502 Bad Gateway` | AI vision engine or LLM reasoning provider timeout. |

---

## 💻 Quick Code Examples

### LangGraph / LangChain Tool Integration

```python
from claix.integrations.langchain import ClaixExtractTool, ClaixSpaceContextTool
from langgraph.prebuilt import create_react_agent
from langchain_openai import ChatOpenAI

tools = [
    ClaixExtractTool(api_key="YOUR_API_KEY"),
    ClaixSpaceContextTool(api_key="YOUR_API_KEY")
]

model = ChatOpenAI(model="gpt-4o")
agent = create_react_agent(model, tools)
```

### CrewAI Multi-Agent Setup

```python
from crewai import Agent, Task, Crew
from claix.integrations.crewai import ClaixKnowledgeSpaceTool

space_tool = ClaixKnowledgeSpaceTool(
    api_key="YOUR_API_KEY",
    space_id="5b9e2c14-7d3a-4f8b-9e1c-6a0d4b8f2e7c"
)

auditor = Agent(
    role="Financial Auditor",
    goal="Reconcile invoices against supplier agreements",
    backstory="Senior auditor utilizing Claix Knowledge Spaces for zero-hallucination document cross-referencing.",
    tools=[space_tool]
)
```

### Node.js (cURL / Fetch)

```javascript
const formData = new FormData();
formData.append('schema_id', '3c7a9f21-4b8e-4d1a-9c6f-2e0d8a5b7c4f');
formData.append('file', new Blob([fs.readFileSync('./invoice.pdf')]), 'invoice.pdf');
formData.append('space_id', '5b9e2c14-7d3a-4f8b-9e1c-6a0d4b8f2e7c'); // Optional

const response = await fetch('[https://claix.dev/api/pdf-json](https://claix.dev/api/pdf-json)', {
  method: 'POST',
  headers: { 'x-api-key': process.env.CLAIX_API_KEY },
  body: formData
});

const result = await response.json();
console.log(result.data);
```

---

## 🛡️ Security & Privacy
* **Isolated Multi-Tenancy:** Knowledge spaces and document context windows are partitioned cryptographically by account API keys.
* **No Model Training:** Customer document data is never used to train third-party foundation models.
* **Controlled Retention:** Temporary context windows auto-expire after workflow completion; persistent documents and spaces can be purged immediately via `DELETE` endpoints.
