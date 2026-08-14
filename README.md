# Claix API — AI Data Processing for Systems & Agents

Claix is a powerful server-to-server API designed to transform unstructured documents, images, tabular data, and raw text into strictly typed JSON formats. Built for backend integrations, automation scripts, and workflow tools like n8n, Zapier, or Make.

Powered by AI, Claix acts as a **schema-driven inference engine**. It handles direct data extraction and complex semantic reasoning (Agent Mode) in a single pass—eliminating fragile OCR templates and unpredictable LLM hallucinations.

🌐 **Website:** [claix.dev](https://claix.dev)  
📄 **OpenAPI Spec (v1.5.0):** [claix.dev/openapi.yaml](https://claix.dev/openapi.yaml)

---

## ⚡ Quick Links
* [Authentication](#-authentication)
* [Base URL](#-base-url)
* [Standard Extraction Endpoints](#-standard-extraction-endpoints)
* [🤖 Agent Mode (Semantic Reasoning)](#-agent-mode-semantic-reasoning)
* [Response Structures](#-response-structures)
* [Error Codes](#-error-codes)
* [Usage Examples](#-usage-examples)

---

## 🔒 Authentication

All API endpoints require authentication via an API Key. You can provide this key in one of two ways:
* **Custom Header (Recommended):** `x-api-key: YOUR_API_KEY`
* **Bearer Token:** `Authorization: Bearer YOUR_API_KEY`

---

## 🌍 Base URL

All standard extraction endpoints are relative to the production base URL:  
`https://claix.dev/api`

*(Note: Agent Mode endpoints use a different path structure. See the Agent Mode section below).*

---

## 🚀 Standard Extraction Endpoints

These endpoints perform direct data mapping based on your schema definition.

### 1. Excel/CSV to JSON
`POST /excel-json`
* **Behavior:** Processes the first sheet of the file and returns an array of records where columns exactly match the schema properties.
* **Request:** `multipart/form-data` containing `file` and `schema_id`.

### 2. JSON to Excel
`POST /json-excel`
* **Behavior:** Generates a binary spreadsheet (`.xlsx`) with columns ordered and named exactly as defined in the schema.
* **Request:** Accepts `multipart/form-data` or raw `application/json` (direct array, single object, or envelope with `schema_id` + `data`).

### 3. PDF to JSON
`POST /pdf-json`
* **Behavior:** Extracts structured data from a PDF (selectable text or scanned/OCR). Treated as a single data source (returns exactly one record).
* **Limits:** Max file size 15 MB.
* **Request:** `multipart/form-data` containing `file` and `schema_id`.

### 4. Document to JSON
`POST /doc-json`
* **Behavior:** Extracts data from text documents. Text is deterministically extracted on the server before AI inference. 
* **Supported Formats:** `.docx`, `.txt`, `.md`, `.rtf`. *(Legacy `.doc` Word 97-2003 is NOT supported).*
* **Limits:** Max file size 10 MB, max extracted text 300,000 characters.
* **Request:** `multipart/form-data` containing `file` and `schema_id`.

### 5. Image to JSON
`POST /img-json`
* **Behavior:** Extracts data from the visible content of an image. If the image is illegible (blurry, bad lighting), it safely fails with a `422` error.
* **Supported Formats:** `.jpeg`, `.jpg`, `.png`, `.webp`, `.heic`, `.heif`.
* **Limits:** Max file size 15 MB.
* **Request:** `multipart/form-data` containing `file` and `schema_id`.

### 6. List Schemas
`GET /schemas`
* **Behavior:** Read-only endpoint returning all schemas created under your account (including Agent definitions). Free to query.

---

## 🤖 Agent Mode (Semantic Reasoning)

Agent Mode allows you to embed custom logic and business rules directly into your schema. Instead of just extracting explicit data, Claix evaluates semantic questions and returns strictly typed answers (booleans, integers, exact strings) ready for your autonomous agents or backend logic.

### Agent Endpoints
To use Agent Mode, replace `/api/` with `/agent/` in your request path:
* `POST https://claix.dev/agent/excel-json`
* `POST https://claix.dev/agent/pdf-json`
* `POST https://claix.dev/agent/doc-json`
* `POST https://claix.dev/agent/img-json`

### Requirements
1. Your schema must have `is_agent_mode: true`.
2. Your schema must contain an `agent_definition` (the rules the AI needs to evaluate).
3. The request format (`multipart/form-data` with `file` and `schema_id`) remains exactly the same as standard endpoints.

---

## 📋 Response Structures

### Standard Extraction Response
All successful standard extraction requests return a standardized JSON structure:

```json
{
  "success": true,
  "schema_utilizado": "Facturas de Proveedores",
  "total_registros": 1,
  "data": [
    {
      "numero_factura": "F-2026-00456",
      "fecha_emision": "2026-03-14",
      "proveedor": "Suministros S.L.",
      "importe_total": 1284.50
    }
  ]
}
```
*(Note: `excel-json` responses also include a `mapa_columnas` object detailing header mappings).*

### Agent Mode Response
When calling an `/agent/*` endpoint, the response includes the standard `data` array **PLUS** an `agent_data` object containing your inferred, type-safe answers:

```json
{
  "success": true,
  "schema_utilizado": "Revisión de Contratos Legales",
  "total_registros": 1,
  "data": [
    {
      "nombre_arrendatario": "Laura Fernández",
      "renta_mensual": 950.00
    }
  ],
  "agent_data": {
    "clausula_penalizacion": true,
    "es_renovacion_automatica": false,
    "tipo_contrato": "indefinido",
    "resumen_agent": "Contrato de alquiler estándar sin cláusulas abusivas detectadas."
  }
}
```

---

## 🚨 Error Codes

Claix provides semantic HTTP status codes for easy debugging:

* **400: Bad Request:** Missing file, unsupported format, invalid UUID, or calling an `/agent/` endpoint with a schema that does not have `is_agent_mode` activated.
* **401: Unauthorized:** Invalid, inactive, or suspended API key.
* **404: Not Found:** Schema ID does not exist or doesn't belong to the account.
* **405: Method Not Allowed:** Usually means you are using GET instead of POST.
* **413: Payload Too Large:** Exceeds file size limits (10MB/15MB) or 300k char limit.
* **422: Unprocessable Entity:** Illegible image, no matching columns found, or no data extracted.
* **500: Internal Server Error:** Claix platform error.
* **502: Bad Gateway:** AI service timeout, extraction failure, or Gemini agent phase failure.

---

## 💻 Usage Examples

### Node.js (Standard PDF to JSON)

```javascript
const fs = require('fs');

async function parsePdf() {
  const formData = new FormData();
  formData.append('schema_id', '3c7a9f21-4b8e-4d1a-9c6f-2e0d8a5b7c4f');
  formData.append('file', new Blob([fs.readFileSync('./invoice.pdf')]), 'invoice.pdf');

  const response = await fetch('[https://claix.dev/api/pdf-json](https://claix.dev/api/pdf-json)', {
    method: 'POST',
    headers: {
      'x-api-key': process.env.CLAIX_API_KEY
    },
    body: formData
  });

  const result = await response.json();
  console.log(result.data);
}
```

### Python (Agent Mode PDF to JSON)

```python
import requests

url = "[https://claix.dev/agent/pdf-json](https://claix.dev/agent/pdf-json)"
headers = {
    "x-api-key": "YOUR_API_KEY"
}
payload = {
    "schema_id": "b980cfe7-61ef-4a5a-9724-881c8a5541e2"
}
files = {
    "file": open("./contract.pdf", "rb")
}

response = requests.post(url, headers=headers, data=payload, files=files)
result = response.json()

# Extracted structured data
print(result["data"]) 

# Type-safe semantic reasoning responses
print(result["agent_data"]) 
```

---

## 🛡️ Security & Privacy

We process data in memory. Your documents are never stored, saved, or used to train external models. Read our full Data Processing Agreement (DPA) on our website.
