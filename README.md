# Claix API & SDK — Unstructured Data to Structured JSON

> Turn documents, spreadsheets, images, text, HTML, XML, and audio into structured, source-traceable JSON for applications, workflows, and AI agents.

[![Claix](https://img.shields.io/badge/Claix-Data%20Intelligence-00C853?style=flat-square)](https://claix.dev)
[![REST](https://img.shields.io/badge/API-REST-0EA5E9?style=flat-square)](https://claix.dev/documentation)
[![A2A](https://img.shields.io/badge/Protocol-A2A-7C3AED?style=flat-square)](https://claix.dev/.well-known/agent-card.json)
[![MCP](https://img.shields.io/badge/Protocol-MCP-7C3AED?style=flat-square)](https://claix.dev/mcp)
[![BYOK](https://img.shields.io/badge/BYOK-No%20Claix%20processing%20fee-16A34A?style=flat-square)](https://claix.dev)

Claix is a developer-first **unstructured data intelligence API**. It transforms business content—including PDFs, Excel and CSV files, Word and text documents, images, HTML, XML, and audio—into schema-defined, structured JSON.

Claix is built for applications, backends, workflows, data pipelines, internal tools, automation platforms, and AI agents. It can be accessed through REST, native Agent-to-Agent (A2A), and Model Context Protocol (MCP) interfaces.

```text
PDF / Excel / CSV / image / text / HTML / XML / audio
        ↓
Claix schema-guided extraction
        ↓
Structured JSON
        ↓
Source tracing and review signals
        ↓
Application, database, CRM, ERP, workflow, or AI agent
```

With optional **Source Tracing**, every extracted field can include evidence showing where the value came from: a spreadsheet column and row, a PDF page and paragraph, an image region, a text fragment, or an audio timestamp. When Claix cannot establish precise evidence, it returns:

```text
requires_human_revision
```

This lets your system distinguish between source-supported data and values that should be reviewed before a consequential action.

🌐 **Website:** [claix.dev](https://claix.dev)  
📚 **Documentation:** [claix.dev/documentation](https://claix.dev/documentation)  
📄 **OpenAPI specification:** [claix.dev/openapi.yaml](https://claix.dev/openapi.yaml)  
📦 **Python SDK:** `pip install claix`  
🤖 **A2A Agent Card:** [claix.dev/.well-known/agent-card.json](https://claix.dev/.well-known/agent-card.json)  
🔌 **MCP server:** [claix.dev/mcp](https://claix.dev/mcp)  

---

## Why Claix?

Raw OCR, transcripts, text chunks, and generic summaries are not enough for most production workflows.

Applications need data they can store, validate, compare, route, and act on.

```text
Unstructured business content
        ↓
Claix
        ↓
Schema-matched JSON
        ↓
Evidence and traceability
        ↓
Reliable automation
```

Claix helps developers:

- Extract structured data from business documents and multimodal inputs.
- Define the exact fields their application or workflow expects.
- Normalize inconsistent inputs into stable JSON contracts.
- Trace extracted values to their source location.
- Flag unsupported values for human review.
- Persist extracted content for later queries.
- Compare information across multiple related files.
- Build workflows through REST, MCP, A2A, n8n, Make, Zapier, or custom backends.
- Use Claix Managed AI or Bring Your Own Key (BYOK).

---

## Key capabilities

| Capability | What it does |
|---|---|
| **Schema-guided extraction** | Returns JSON matching the fields and types defined by your schema |
| **Multimodal inputs** | Processes spreadsheets, PDFs, documents, images, text, HTML, XML, and audio |
| **Source Tracing** | Returns evidence per field when enabled |
| **Human review signals** | Uses `requires_human_revision` when precise evidence cannot be established |
| **Agent Mode** | Adds typed, schema-defined reasoning after extraction |
| **Persistent context** | Stores processed content for follow-up questions through `document_id` |
| **Knowledge Spaces** | Groups related content under `space_id` for cross-document or cross-source queries |
| **REST API** | Integrates with applications, backends, scripts, and automation platforms |
| **Native A2A** | Lets A2A-compatible agents discover Claix and delegate tasks |
| **MCP server** | Lets MCP-capable assistants and tools call Claix without custom REST wiring |
| **BYOK** | Use your own supported AI provider key with no additional Claix processing fee |

---

## Supported inputs

| Input type | Endpoint | Typical use cases |
|---|---|---|
| Excel / CSV | `POST /api/excel-json` | CRM imports, supplier lists, lead data, product catalogs, financial exports |
| PDF | `POST /api/pdf-json` | Invoices, contracts, reports, forms, purchase orders, scanned PDFs |
| Documents | `POST /api/doc-json` | Word documents, markdown, RTF files, policies, proposals, notes |
| Images | `POST /api/img-json` | Receipts, screenshots, photographed forms, visual records |
| Text / HTML / XML | `POST /api/txt-json` | Email bodies, raw text, web forms, HTML exports, XML feeds |
| Audio | `POST /api/audio-json` | Calls, meetings, voice notes, interviews, dictation, operational reports |

Claix accepts the following input formats:

```text
.xlsx · .csv
.pdf
.docx · .txt · .md · .rtf
.jpeg · .jpg · .png · .webp · .heic · .heif
raw text · HTML · XML
.mp3 · .wav · .m4a · .ogg
```

---

## Table of contents

- [Authentication](#authentication)
- [Quick start](#quick-start)
- [Source Tracing](#source-tracing)
- [API overview](#api-overview)
  - [Structured extraction](#structured-extraction)
  - [Audio extraction](#audio-extraction)
  - [Agent Mode](#agent-mode)
  - [Persistent context](#persistent-context)
  - [Knowledge Spaces](#knowledge-spaces)
  - [Schema management](#schema-management)
- [Python SDK](#python-sdk)
- [A2A integration](#a2a-integration)
- [MCP server](#mcp-server)
- [Response formats](#response-formats)
- [Error handling](#error-handling)
- [BYOK and pricing](#byok-and-pricing)
- [Security practices](#security-practices)

---

# Authentication

All Claix API calls require an API key.

Use one of the following headers:

```http
x-api-key: YOUR_API_KEY
```

or:

```http
Authorization: Bearer YOUR_API_KEY
```

The dedicated `x-api-key` header is recommended.

> [!IMPORTANT]
> Claix API keys are server credentials. Do not expose them in browser code, mobile apps, public repositories, screenshots, client-side workflows, or logs. Store them in server-side environment variables or a secrets manager.

---

# Quick start

## 1. Create a schema

Create a schema in the Claix dashboard for the input type you want to process.

For example, an invoice extraction schema may define:

```json
{
  "invoice_number": {
    "type": "string",
    "description": "The invoice identifier. Return null if not explicitly present."
  },
  "supplier_name": {
    "type": "string",
    "description": "The supplier or issuer of the invoice."
  },
  "invoice_date": {
    "type": "string",
    "description": "Invoice issue date in ISO 8601 format when possible."
  },
  "due_date": {
    "type": "string",
    "description": "Payment due date in ISO 8601 format when explicitly present."
  },
  "total_amount": {
    "type": "number",
    "description": "Final total amount due. Do not return a subtotal, tax value, or line-item amount."
  },
  "currency": {
    "type": "string",
    "description": "Currency code or explicitly stated currency."
  }
}
```

## 2. Send a file and schema ID

```bash
curl -X POST "[https://claix.dev/api/pdf-json](https://claix.dev/api/pdf-json)" \
  -H "x-api-key: YOUR_API_KEY" \
  -F "file=@./invoice.pdf" \
  -F "schema_id=YOUR_SCHEMA_UUID"
```

## 3. Read the structured result

```json
{
  "success": true,
  "schema_utilizado": "Supplier invoices",
  "total_registros": 1,
  "data": [
    {
      "invoice_number": "INV-2026-0841",
      "supplier_name": "Northwind Supplies",
      "invoice_date": "2026-09-17",
      "due_date": "2026-10-17",
      "total_amount": 1284.5,
      "currency": "EUR"
    }
  ],
  "log_id": "7c2e1a90-4b3d-4f8a-9e21-6d5c8b0a1f34"
}
```

---

# Source Tracing

## Extract data with evidence

Claix can return field-level evidence when Source Tracing is enabled in a schema.

Without Source Tracing, fields remain simple values:

```json
{
  "invoice_number": "INV-2026-0841",
  "total_amount": 1284.5,
  "currency": "EUR"
}
```

With Source Tracing enabled, each field becomes:

```json
{
  "value": "...",
  "source": "..."
}
```

Example:

```json
{
  "success": true,
  "data": [
    {
      "invoice_number": {
        "value": "INV-2026-0841",
        "source": "page 1, \"Invoice No. INV-2026-0841\""
      },
      "total_amount": {
        "value": 1284.5,
        "source": "page 1, \"Total Due: EUR 1,284.50\""
      },
      "currency": {
        "value": "EUR",
        "source": "page 1, \"Total Due: EUR 1,284.50\""
      },
      "due_date": {
        "value": null,
        "source": "requires_human_revision"
      }
    }
  ],
  "log_id": "7c2e1a90-4b3d-4f8a-9e21-6d5c8b0a1f34"
}
```

## Evidence by input type

| Input | Example `source` |
|---|---|
| Excel / CSV | `column "Customer", row 2` |
| PDF | `page 1, paragraph 3` or quoted text |
| Word / text | Section, paragraph, or text fragment |
| Image | Image area, visual label, or recognized text fragment |
| HTML / XML | Relevant element or content fragment |
| Audio | `second 72` or `seconds 220-235` |
| Document Context | Relevant persisted document source |
| Knowledge Space | Document IDs, file names, and the cross-source relationship |

## Human review signal

When Claix cannot establish a precise source, it uses:

```text
requires_human_revision
```

Treat this as a workflow control signal:

```text
If source is available:
    continue validation or automation

If source is requires_human_revision:
    route to review
    request clarification
    avoid irreversible action
```

> [!NOTE]
> Source Tracing improves traceability and reviewability. It does not replace deterministic validation, authorization checks, business rules, or human review for high-impact decisions.

---

# API overview

Base REST API URL:

```text
[https://claix.dev/api](https://claix.dev/api)
```

Some endpoints, including Agent Mode, Context, Knowledge Spaces, A2A, and MCP, use explicit root-level URLs described below.

## Structured extraction

| Endpoint | Method | Input | Description |
|---|---|---|---|
| `/api/excel-json` | `POST` | `.xlsx`, `.csv` | Converts spreadsheet rows into schema-defined JSON records |
| `/api/pdf-json` | `POST` | `.pdf` | Extracts a schema-defined JSON record from a text or scanned PDF |
| `/api/doc-json` | `POST` | `.docx`, `.txt`, `.md`, `.rtf` | Extracts structured data from text-based documents |
| `/api/img-json` | `POST` | `.jpeg`, `.jpg`, `.png`, `.webp`, `.heic`, `.heif` | Extracts structured data from images |
| `/api/txt-json` | `POST` | Text, HTML, XML | Extracts structured data from raw content |
| `/api/audio-json` | `POST` | `.mp3`, `.wav`, `.m4a`, `.ogg` | Extracts structured data from spoken audio |

All extraction endpoints require:

```text
file
schema_id
```

except text extraction, which accepts:

```text
content
schema_id
```

Most extraction endpoints also accept an optional:

```text
space_id
```

Use `space_id` when you want to associate persisted content with a Knowledge Space.

## Extraction behavior

| Input | Output shape | Important behavior |
|---|---|---|
| Excel / CSV | `data[]` with one object per processed row | Only the first worksheet is processed for Excel files |
| PDF | `data[0]` | One PDF returns one structured record |
| Document | `data[0]` | One document returns one structured record |
| Image | `data[0]` | One image returns one structured record |
| Text / HTML / XML | `data[0]` | Raw content returns one structured record |
| Audio | `data[0]` | One audio file returns one structured record |

## File limits

| Input | Limit |
|---|---|
| PDF | 15 MB |
| Image | 15 MB |
| Audio | 12 MB and 10 minutes |
| Document | 10 MB or 300,000 characters |
| Text / HTML / XML | 300,000 characters |
| Excel / CSV | First worksheet only |

---

# Audio extraction

Claix Audio-to-JSON turns calls, meetings, voice notes, interviews, dictation, and spoken operational updates into schema-defined JSON.

```text
Call or voice note
        ↓
Claix Audio-to-JSON
        ↓
Structured business data
        ↓
CRM, ticket, workflow, database, or agent action
```

Example:

```bash
curl -X POST "[https://claix.dev/api/audio-json](https://claix.dev/api/audio-json)" \
  -H "x-api-key: YOUR_API_KEY" \
  -F "file=@./supplier-voice-note.mp3" \
  -F "schema_id=YOUR_AUDIO_SCHEMA_UUID"
```

Example response with Source Tracing:

```json
{
  "success": true,
  "schema_utilizado": "Invoice voice notes",
  "total_registros": 1,
  "data": [
    {
      "invoice_number": {
        "value": "F-2026-00456",
        "source": "second 72, \"invoice F-2026-00456\""
      },
      "total_amount": {
        "value": 1284.5,
        "source": "seconds 220-235, total amount"
      },
      "currency": {
        "value": null,
        "source": "requires_human_revision"
      }
    }
  ],
  "log_id": "7c2e1a90-4b3d-4f8a-9e21-6d5c8b0a1f34"
}
```

Audio files that are silent, unintelligible, excessively noisy, distorted, or contain no meaningful speech return:

```text
422 Unprocessable Entity
```

---

# Agent Mode

Agent Mode performs two stages:

```text
1. Schema-guided extraction
2. Typed reasoning based on your agent_definition
```

Use it when your workflow needs both factual fields and structured operational interpretation.

Examples:

```text
Is the request urgent?

Does the call require finance follow-up?

Does a contract renew automatically?

Should this support issue be escalated?

Does an invoice conflict with a purchase order?
```

## Agent Mode endpoints

| Endpoint | Input |
|---|---|
| `POST /agent/excel-json` | Excel / CSV |
| `POST /agent/pdf-json` | PDF |
| `POST /agent/doc-json` | Document |
| `POST /agent/img-json` | Image |
| `POST /agent/txt-json` | Text / HTML / XML |
| `POST /agent/audio-json` | Audio |

Requirements:

```text
Schema type must match the input.
is_agent_mode must be enabled.
agent_definition must be configured.
```

Example audio Agent Mode request:

```bash
curl -X POST "[https://claix.dev/agent/audio-json](https://claix.dev/agent/audio-json)" \
  -H "x-api-key: YOUR_API_KEY" \
  -F "file=@./customer-call.m4a" \
  -F "schema_id=YOUR_AGENT_AUDIO_SCHEMA_UUID"
```

Example response:

```json
{
  "success": true,
  "schema_utilizado": "Customer call notes",
  "total_registros": 1,
  "data": [
    {
      "customer_name": {
        "value": "Suministros Industriales del Ebro S.L.",
        "source": "seconds 0-6"
      },
      "invoice_number": {
        "value": "F-2026-00456",
        "source": "seconds 7-12"
      }
    }
  ],
  "agent_data": {
    "urgency": {
      "value": true,
      "source": "seconds 26-28"
    },
    "requires_finance_follow_up": {
      "value": true,
      "source": "seconds 20-25"
    },
    "call_summary": {
      "value": "The customer requested urgent payment confirmation for invoice F-2026-00456.",
      "source": "seconds 7-28"
    }
  },
  "log_id": "7c2e1a90-4b3d-4f8a-9e21-6d5c8b0a1f34"
}
```

---

# Persistent context

When Context Window is enabled for a schema, Claix persists the processed representation and returns a `document_id`.

```text
Process once
        ↓
Receive document_id
        ↓
Ask follow-up questions later
        ↓
Avoid repeatedly uploading the same source
```

This works for persisted content from documents and supported multimodal inputs, including audio transcription stored as Markdown.

## Ask a persisted document

```text
POST [https://claix.dev/document-context/{document_id}](https://claix.dev/document-context/{document_id})
```

Request body:

```json
{
  "questions": [
    "What payment terms are listed?",
    "Does the agreement renew automatically?"
  ]
}
```

Limits:

```text
Maximum questions per request: 5
Maximum characters per question: 400
```

## Get persisted content

```text
GET [https://claix.dev/get-document/{document_id}](https://claix.dev/get-document/{document_id})
```

## Delete a persisted document

```text
DELETE [https://claix.dev/delete-document/{document_id}](https://claix.dev/delete-document/{document_id})
```

> [!CAUTION]
> Deleting a persisted document is irreversible.

Context retrieval and document deletion do not consume extraction credits.

---

# Knowledge Spaces

Knowledge Spaces group related persisted inputs under a `space_id`.

```text
Contract
    + invoice
    + spreadsheet
    + voice note
    + purchase order
        ↓
Knowledge Space
        ↓
Cross-source question
```

This is useful for tasks such as:

```text
Does the amount mentioned in the supplier call match the invoice?

Which supplier billed the highest total amount?

Do any invoices exceed the agreed contract price?

What commitments were made across calls and written documents?

Which contracts renew in the next 90 days?
```

## Create a Knowledge Space

```text
POST [https://claix.dev/create-space](https://claix.dev/create-space)
```

Request body:

```json
{
  "name": "Supplier operations 2026"
}
```

The response includes a `space_id`.

## Associate content with a Knowledge Space

Pass `space_id` during extraction or Agent Mode processing:

```text
space_id=YOUR_SPACE_UUID
```

This works when Context Window is enabled and the input is persisted.

## Ask questions across a Knowledge Space

```text
POST [https://claix.dev/space-context/{space_id}](https://claix.dev/space-context/{space_id})
```

Request body:

```json
{
  "questions": [
    "Which invoice does not match the contracted price?",
    "What supplier billed the highest total amount?"
  ]
}
```

Knowledge Space limits:

```text
Up to 50 active documents per query
Up to 200,000 characters of total context
Up to 5 questions per request
Up to 400 characters per question
```

When Knowledge Space source verification is enabled, each answer can include the relevant source documents, file names, document IDs, and cross-source relationship.

Example:

```json
{
  "user_ask": [
    "What supplier billed the highest total amount?"
  ],
  "ia_response": [
    {
      "value": "Suministros Omega S.A., with €48,320 across three invoices.",
      "source": "cross-reference of document_id a1b2c3d4-1111-2222-3333-444444444444 (invoice-feb.pdf) and document_id b2c3d4e5-5555-6666-7777-888888888888 (invoice-mar.pdf): sum of invoice amounts"
    }
  ],
  "log_id": "7c2e1a90-4b3d-4f8a-9e21-6d5c8b0a1f34"
}
```

If Claix cannot establish precise supporting evidence:

```json
{
  "value": "A possible discrepancy was detected.",
  "source": "requires_human_revision"
}
```

## Delete a Knowledge Space

```text
DELETE [https://claix.dev/delete-space/{space_id}](https://claix.dev/delete-space/{space_id})
```

> [!CAUTION]
> Deleting a Knowledge Space is irreversible.

---

# Schema management

| Operation | Endpoint | Method |
|---|---|---|
| List schemas | `/api/schemas` | `GET` |
| Create schema | `/api/create-schema` | `POST` |
| Delete schema | `/api/delete-schema` | `POST` or `DELETE` |

Schemas define:

- Input type.
- Expected output fields.
- Field types.
- Field descriptions.
- Whether Context Window is enabled.
- Whether Source Tracing is enabled.
- Whether Agent Mode is enabled.
- Agent Mode typed reasoning definitions.

Use precise field descriptions. Good descriptions improve extraction quality when a source contains multiple similar values.

Example:

```text
Bad:
amount

Better:
Final amount payable by the customer. Do not return subtotal, taxes,
discounts, account balance, or individual line-item values.
```

---

# Python SDK

Install the official Python SDK:

```bash
pip install claix
```

Optional agent-framework extras:

```bash
pip install claix[langchain]
pip install claix[crewai]
pip install claix[all]
```

## Python quick start

```python
from claix import ClaixClient

client = ClaixClient(api_key="YOUR_API_KEY")

result = client.extract.pdf(
    file="invoice_2026.pdf",
    schema_id="YOUR_SCHEMA_UUID",
    space_id="YOUR_SPACE_UUID"
)

print(result.data)
print(result.log_id)
```

## Query a Knowledge Space

```python
space_answer = client.spaces.ask(
    space_id="YOUR_SPACE_UUID",
    questions=[
        "Which supplier billed the highest total amount across all invoices?"
    ]
)

print(space_answer.ia_response)
print(space_answer.log_id)
```

> [!NOTE]
> Consult the [Claix SDK documentation](https://claix.dev/documentation/sdks) for the current Python SDK methods, version support, and integration examples.

---

# A2A integration

Claix is a native **Agent-to-Agent (A2A)** agent.

A2A-compatible agents, orchestrators, and scripts can discover Claix through its public Agent Card:

```text
[https://claix.dev/.well-known/agent.json](https://claix.dev/.well-known/agent.json)
```

Alternative Agent Card path:

```text
[https://claix.dev/.well-known/agent-card.json](https://claix.dev/.well-known/agent-card.json)
```

A2A JSON-RPC endpoint:

```text
[https://claix.dev/a2a](https://claix.dev/a2a)
```

Claix supports A2A tasks for:

- Schema discovery and management.
- Structured extraction from Excel/CSV, PDF, documents, images, text, HTML, and XML.
- Agent Mode extraction.
- Persistent document retrieval and queries.
- Knowledge Space creation and queries.
- Document and Knowledge Space deletion.

Example discovery request:

```bash
curl -sS "[https://claix.dev/.well-known/agent.json](https://claix.dev/.well-known/agent.json)"
```

For complete A2A documentation, see:

```text
[https://claix.dev/documentation/a2a](https://claix.dev/documentation/a2a)
```

---

# MCP server

Claix exposes a hosted MCP server for AI assistants, coding tools, automation platforms, and MCP-capable clients.

MCP endpoint:

```text
[https://claix.dev/mcp](https://claix.dev/mcp)
```

MCP server metadata:

```text
[https://claix.dev/.well-known/mcp/server-card.json](https://claix.dev/.well-known/mcp/server-card.json)
```

The MCP server exposes tools for:

- Schema management.
- Structured extraction.
- Agent Mode.
- Persistent document context.
- Knowledge Spaces.
- Document deletion.
- Knowledge Space deletion.

Example Cursor configuration:

```json
{
  "mcpServers": {
    "claix": {
      "url": "[https://claix.dev/mcp](https://claix.dev/mcp)",
      "headers": {
        "x-api-key": "YOUR_API_KEY"
      }
    }
  }
}
```

See the dedicated [Claix MCP repository](https://github.com/claix-ai/claix-mcp) or [MCP documentation](https://claix.dev/documentation/mcp) for the full tool catalog and client setup guides.

---

# Response formats

## Standard extraction

```json
{
  "success": true,
  "schema_utilizado": "Supplier invoices",
  "total_registros": 1,
  "data": [
    {
      "invoice_number": "INV-2026-0841",
      "supplier_name": "Northwind Supplies",
      "total_amount": 1284.5,
      "currency": "EUR"
    }
  ],
  "log_id": "7c2e1a90-4b3d-4f8a-9e21-6d5c8b0a1f34"
}
```

## Extraction with Source Tracing

```json
{
  "success": true,
  "data": [
    {
      "invoice_number": {
        "value": "INV-2026-0841",
        "source": "page 1, \"Invoice No. INV-2026-0841\""
      },
      "total_amount": {
        "value": 1284.5,
        "source": "page 1, \"Total Due: EUR 1,284.50\""
      },
      "currency": {
        "value": "EUR",
        "source": "page 1, \"Total Due: EUR 1,284.50\""
      }
    }
  ],
  "log_id": "7c2e1a90-4b3d-4f8a-9e21-6d5c8b0a1f34"
}
```

## Error response

```json
{
  "error": "Human-readable error message.",
  "detalle": "Optional technical detail.",
  "log_id": "7c2e1a90-4b3d-4f8a-9e21-6d5c8b0a1f34"
}
```

## `log_id`

Every successful request includes a `log_id` when the usage record is saved. Most authenticated errors also include one.

Use it to:

- Find the exact request in Claix logs.
- Correlate Claix processing with your own workflow run.
- Debug unexpected extraction behavior.
- Provide a support reference.
- Build an audit trail.

---

# Error handling

| Status | Meaning | Retry? |
|---:|---|---|
| `200` | Request completed successfully | — |
| `400` | Invalid request, missing fields, malformed payload, wrong schema type, or invalid input | No; correct the request |
| `401` | Missing, invalid, inactive, or suspended API key | No; correct credentials |
| `404` | Schema, document, or Knowledge Space does not exist or belongs to a different account | No; correct the identifier |
| `405` | Incorrect HTTP method | No; use the documented method |
| `413` | File is too large, audio is too long, or content exceeds limits | No; reduce the input |
| `422` | Input is unreadable, unintelligible, or contains no extractable data for the schema | No; review the source or schema |
| `429` | Rate limit exceeded | Yes; follow `Retry-After` where available |
| `500` | Internal Claix error | Retry carefully |
| `502` | Temporary AI-provider or processing failure | Retry with exponential backoff |

Recommended retry policy:

```text
Retry:
429, 500, 502

Do not retry without changing the request:
400, 401, 404, 405, 413, 422
```

---

# BYOK and pricing

Claix supports:

| Option | Best for | Billing |
|---|---|---|
| Claix Managed AI | Teams that want a managed model and provider setup | Claix processing pricing |
| Bring Your Own Key | Teams that want provider and model control | No additional Claix processing fee; provider usage applies |

With BYOK:

```text
Claix processing fee: €0
AI provider usage: billed by your selected provider
```

BYOK does not mean AI inference is free. Your provider may charge for tokens, audio processing, image or vision processing, long context, caching, premium models, or other services.

---

# Security practices

- Keep API keys server-side.
- Use separate API keys for development, staging, demos, and production when possible.
- Do not process content without an appropriate legal basis, notice, or authorization.
- Treat uploaded files, source text, HTML, XML, and audio as untrusted input.
- Validate extracted values before using them in critical workflows.
- Require Source Tracing for fields that drive payments, approvals, access changes, legal actions, compliance decisions, or destructive operations.
- Route `requires_human_revision` fields to a review workflow.
- Do not allow extracted content to override application policy, authorization, or system instructions.
- Store `log_id` alongside important workflow decisions.
- Delete persisted documents and Knowledge Spaces when they are no longer needed.
- Restrict downstream permissions so an agent or automation can perform only the minimum actions required.

---

# Use cases

## Invoice and finance operations

```text
Invoice PDF or supplier voice note
        ↓
Claix extracts invoice number, supplier, amount, currency, and due date
        ↓
Source evidence is returned per field
        ↓
Compare against purchase order
        ↓
Approve, reject, or route to finance review
```

## Contract intelligence

```text
Contract PDF or recorded negotiation
        ↓
Claix extracts parties, dates, renewal terms, payment clauses, and obligations
        ↓
Persist context and source evidence
        ↓
Query the contract later or compare it with related documents
```

## Customer support and sales calls

```text
Customer call recording
        ↓
Claix extracts account, issue, invoice, urgency, requested action, and deadline
        ↓
Agent Mode identifies escalation or follow-up requirements
        ↓
Create CRM record, support ticket, or workflow task
```

## Spreadsheet normalization

```text
Excel or CSV upload
        ↓
Claix maps inconsistent source columns to your schema
        ↓
Returns normalized JSON with source row and column evidence
        ↓
Import records into CRM, database, or internal tooling
```

## Multisource Knowledge Spaces

```text
Contract + invoices + spreadsheet + customer call
        ↓
Claix Knowledge Space
        ↓
Cross-source questions, reconciliation, and discrepancy detection
        ↓
Evidence-backed answer or requires_human_revision
```

---

# Contributing

This repository contains examples and integration guidance for the hosted Claix API.

Contributions are welcome for:

- Python, JavaScript, Node.js, PHP, and n8n examples.
- Input-specific schemas.
- Invoice, contract, support, sales, and audio workflow examples.
- A2A and MCP client integrations.
- Documentation improvements.
- Security and reliability guidance.

Please do not commit real API keys, private documents, customer records, audio recordings, or credentials.

---

# License

Unless otherwise stated, code examples and documentation in this repository are licensed under the [MIT License](LICENSE).

---

# Disclaimer

Claix is a hosted service. This repository provides API examples and integration guidance; it does not provide a self-hosted Claix deployment.

For the latest endpoint details, schemas, limits, supported providers, pricing, and product terms, refer to the official [Claix documentation](https://claix.dev/documentation).
