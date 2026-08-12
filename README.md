# claix-for-devs
Official JS/TS SDK and React Widget for Claix — Type-safe document-to-JSON API
# Claix API — Document & Data to Type-Safe JSON

Claix is a powerful server-to-server API designed to transform unstructured documents, images, tabular data, and JSON into structured, type-safe formats. Built for backend integrations, automation scripts, and workflow tools like n8n, Zapier, or Make.

Powered by AI, Claix uses **automatic semantic recognition** to handle synonyms, abbreviations, translations, and name variants between your input files and your predefined JSON schemas.

🌐 **Website:** [claix.dev](https://www.claix.dev)  
📄 **OpenAPI Spec (v1.4.0):** [claix.dev/openapi.yaml](https://www.claix.dev/openapi.yaml)  

---

## ⚡ Quick Links
- [Authentication](#-authentication)
- [Base URL](#-base-url)
- [Endpoints Overview](#-endpoints)
- [Supported Formats & Limits](#-supported-formats--limits)
- [Usage Examples](#-usage-examples)

---

## 🔒 Authentication

All API endpoints require authentication via an API Key. You can provide this key in one of two ways:

1. **Custom Header (Recommended):** `x-api-key: YOUR_API_KEY`
2. **Bearer Token:** `Authorization: Bearer YOUR_API_KEY`

---

## 🌍 Base URL

All endpoints are relative to the production base URL:
```text
[https://claix.dev/api](https://claix.dev/api)
```

---

## 🚀 Endpoints

### 1. Excel/CSV to JSON
`POST /excel-json`

Converts a `.xlsx` or `.csv` file into JSON based on a specific schema. 
- **Behavior:** The API processes the first sheet of the file and returns an array of records where the columns exactly match the schema properties.
- **Request:** `multipart/form-data` containing `file` and `schema_id`.

### 2. JSON to Excel
`POST /json-excel`

Converts one or multiple JSON documents into a binary Excel (`.xlsx`) file.
- **Behavior:** Generates a spreadsheet with columns ordered and named exactly as defined in the schema.
- **Request:** Accepts `multipart/form-data` or raw `application/json` (direct array, single object, or envelope with `schema_id` + `data`).

### 3. PDF to JSON
`POST /pdf-json`

Extracts structured data from a PDF document (selectable text or scanned/OCR).
- **Behavior:** The entire PDF is treated as a single data source. The response always contains exactly one record in the `data` array.
- **Limits:** Max file size is **15 MB**.
- **Request:** `multipart/form-data` containing `file` and `schema_id`.

### 4. Document to JSON
`POST /doc-json`

Extracts structured data from text documents.
- **Behavior:** Text is deterministically extracted on the server before being sent to the AI model. The document is treated as a single data source (returns one record).
- **Supported Formats:** `.docx`, `.txt`, `.md`, `.rtf`. *(Note: Legacy `.doc` Word 97-2003 is NOT supported).*
- **Limits:** Max file size **10 MB**, max extracted text **300,000 characters**.
- **Request:** `multipart/form-data` containing `file` and `schema_id`.

### 5. Image to JSON
`POST /img-json`

Extracts structured data from the visible content of an image using multimodal AI.
- **Behavior:** The model first evaluates image legibility (sharpness, focus, lighting). If illegible, it safely fails with a `422` error. Returns exactly one record.
- **Supported Formats:** `.jpeg`, `.jpg`, `.png`, `.webp`, `.heic`, `.heif`.
- **Limits:** Max file size **15 MB**.
- **Request:** `multipart/form-data` containing `file` and `schema_id`.

### 6. List Schemas
`GET /schemas`

Read-only endpoint that returns all schemas created under your account.
- **Behavior:** Returns full schema definitions, types, and IDs ordered newest to oldest. 
- **Billing:** This endpoint does not generate usage logs and is entirely free to query.

---

## 📋 Response Structures

### Success Response (Parsing Endpoints)
All successful extraction/conversion requests return a standardized JSON structure:

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
*Note: `excel-json` responses also include a `mapa_columnas` object detailing how original headers were mapped to schema properties.*

### Error Codes
Claix provides semantic HTTP status codes for easy debugging:
- **`400`**: Bad Request (missing file, unsupported format, invalid UUID).
- **`401`**: Unauthorized (Invalid, inactive, or suspended API key).
- **`404`**: Schema ID not found or doesn't belong to the account.
- **`413`**: Payload Too Large (exceeds 10MB/15MB limits or 300k char limit).
- **`422`**: Unprocessable Entity (illegible image, no matching columns found, or no data extracted).
- **`502`**: Bad Gateway (AI service timeout or failure).

---

## 💻 Usage Examples

### Node.js (cURL equivalent) - PDF to JSON

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

### Python - List Schemas

```python
import requests

url = "[https://claix.dev/api/schemas](https://claix.dev/api/schemas)"
headers = {
    "x-api-key": "YOUR_API_KEY"
}

response = requests.get(url, headers=headers)
print(response.json())
```

---

## 🛡️ Security & Privacy
We process data in memory. Your documents are never stored, saved, or used to train external models. Read our full [Data Processing Agreement (DPA)](https://www.claix.dev/dpa).
