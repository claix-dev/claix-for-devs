# Claix: the security layer between your database and AI agents

> Connect your SQL database and give each AI agent two controls: it can retrieve only the tables and columns you allow, and every `INSERT`, `UPDATE`, and `DELETE` it proposes is checked against your rules before your code runs it. Your data stays in your database. Call Claix over REST, MCP, and A2A.

[![Claix](https://img.shields.io/badge/Claix-Security%20layer-00C853?style=flat-square)](https://claix.dev)
[![REST](https://img.shields.io/badge/API-REST-0EA5E9?style=flat-square)](https://claix.dev/documentation)
[![MCP](https://img.shields.io/badge/Protocol-MCP-7C3AED?style=flat-square)](https://claix.dev/mcp)
[![A2A](https://img.shields.io/badge/Protocol-A2A-7C3AED?style=flat-square)](https://claix.dev/.well-known/agent.json)
[![Pricing](https://img.shields.io/badge/Pricing-Pay%20per%20use-16A34A?style=flat-square)](https://claix.dev)

Claix is a hosted control layer for AI agents. Retrieve answers from your live database, inside the perimeter you set. Guard decides whether a proposed change is allowed. Your application executes the statement only when the answer is allow.

```text
Your SQL database
        ↓
Data Map: tables, columns, and relationships
        ↓
Routes: which tables and columns a question may use
        ↓
Agent: its endpoint, business prompt, and routes
        ↓
Claix Retrieve: raw rows or typed answers
Claix Guard: allow or deny, with the reason
        ↓
Your application, workflow, or AI agent
```

Claix does not copy your data into a vector index. Retrieve reads the database where it lives. Guard does not run the statement.

🌐 **Website:** [claix.dev](https://claix.dev)
📚 **Documentation:** [claix.dev/documentation](https://claix.dev/documentation)
📄 **OpenAPI:** [claix.dev/openapi.yaml](https://claix.dev/openapi.yaml)
🤖 **A2A Agent Card:** [claix.dev/.well-known/agent.json](https://claix.dev/.well-known/agent.json)
🔌 **MCP server:** [claix.dev/mcp](https://claix.dev/mcp)

---

## What Claix does

| Product | What it does | How you call it |
| --- | --- | --- |
| **Claix Retrieve** | Controlled reads. The agent asks in plain language and gets raw rows or typed answers from the live database. PostgreSQL, MySQL, MariaDB, and Postgres-compatible databases such as Supabase. | `POST /retrieve/{agent_name}`, MCP `claix.retrieve.ask`, A2A |
| **Claix Guard** | Checks every `INSERT`, `UPDATE`, and `DELETE` an agent proposes against rules you wrote in plain English. Returns allow or deny, with the reason. Claix does not execute the statement. | `POST /check-sql` |
| **Claix Ingest** | Turns PDFs, Excel, Word, images, audio, text, HTML, and XML into schema-validated JSON or Markdown. With Save Data on, a mapped row can be inserted into the connected database. | `POST /api/*-json`, `POST /api/*-markdown`, the Python and TypeScript SDKs |

---

## Why Claix?

An agent that needs your database usually gets one of two setups. You build the query tools, permission checks, and logs yourself, or you hand the model a database connection and let it write SQL. The second one works in a demo. In production the model can be steered by text it reads, a query can run cleanly and still answer the wrong question, and a prompt is the only thing between the agent and an `UPDATE` or `DELETE`.

Claix is the service you call instead. You connect a read-only role, choose tables and columns per route, attach routes to an agent, and write Guard rules for the changes that agent may propose. The agent asks. Claix returns typed answers and, when a change is proposed, a verdict. Your code is what writes.

---

## Quick start

### 1. Create a read-only role

The database role is the real barrier. Grant only the tables and columns agents may read. On PostgreSQL:

```sql
CREATE ROLE claix_agent WITH LOGIN PASSWORD 'use-a-secret-manager'
  NOSUPERUSER NOCREATEDB NOCREATEROLE NOINHERIT NOREPLICATION NOBYPASSRLS
  CONNECTION LIMIT 10;

GRANT CONNECT ON DATABASE mydb TO claix_agent;
GRANT USAGE ON SCHEMA public TO claix_agent;

GRANT SELECT (id, name, country) ON public.customers TO claix_agent;
GRANT SELECT (id, customer_id, status, total) ON public.orders TO claix_agent;
```

MySQL and MariaDB use a `claix_agent` user with `REQUIRE SSL` and `SELECT` on the columns you allow. Step-by-step guides:

- [Connect PostgreSQL](https://claix.dev/documentation/connect-postgres)
- [Connect MySQL](https://claix.dev/documentation/connect-mysql)
- [Connect MariaDB](https://claix.dev/documentation/connect-mariadb)

### 2. Connect the database

In the Claix dashboard, open **Database connection** and add the read-only credentials. Claix supports PostgreSQL, MySQL, MariaDB, and Postgres-compatible databases such as Supabase.

### 3. Review the Data Map

Claix reads the structure that role can see and drafts a Data Map: descriptions for each table, the relationships it finds, and the ones it suggests. Review it, correct it, and publish it.

### 4. Create a route

A route is one retrieval capability. It lists the tables and columns that can be used, plus a short guide with your vocabulary and rules. Create it in the dashboard or with `POST /routes`.

### 5. Create an agent

An agent has a name, a slug, a description, a business prompt, the routes it may use, and a response mode.

| `response_mode` | What you get |
| --- | --- |
| `bruto` | The query rows, without a second model call. `format` is optional. |
| `razonada` | A reasoned answer in the type you ask for. `format` is required. |

The slug becomes the public endpoint. A slug of `restaurant-ops` is called at `POST /retrieve/restaurant-ops`.

Create it in the dashboard or with `POST /agents`.

### 6. Ask

Up to 5 questions per call. In `razonada` mode each question needs a `format`: `string`, `int`, `boolean`, `timestamp`, `timestamptz`, `date`, `json`, `array:string`, or `array:int`.

```bash
curl -X POST "https://claix.dev/retrieve/restaurant-ops" \
  -H "x-api-key: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "questions": [
      {
        "question": "How many tables are reserved this Saturday between 20:00 and 22:00?",
        "format": "int"
      }
    ]
  }'
```

```json
{
  "user_ask": [
    {
      "question": "How many tables are reserved this Saturday between 20:00 and 22:00?",
      "format": "int"
    }
  ],
  "ia_response": [14],
  "log_id": "9ff38974-1392-4c1a-83e5-3b2f09bb43dc"
}
```

Answers come back in the same order as the questions. When the answer is outside the tables and columns that agent may use, Claix leaves the value empty. It does not invent one.

`log_id` is the trace: the question, the route, the query, and the result, in the dashboard.

---

## Claix Guard

Guard is the check that runs before your code changes data. You write the rules in plain English. Claix turns them into exact rules with test cases, and you accept, reject, or edit them. At check time a model does not decide: the same statement gets the same verdict.

```bash
curl -X POST "https://claix.dev/check-sql" \
  -H "x-api-key: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "sql": "UPDATE orders SET discount_percent = $1 WHERE id = $2",
    "params": [35, 4821]
  }'
```

```json
{
  "allowed": false,
  "code": "rule_denied",
  "reason": "discount_percent must be 20 or less. Received 35.",
  "decision_id": "9ff38974-1392-4c1a-83e5-3b2f09bb43dc"
}
```

A denial is still HTTP 200. Read `allowed`. Claix does not run the SQL. Your executor runs it only when `allowed` is true.

Guard is €0.01 per check, priced separately from Retrieve. There is no model call on the check. Docs: [POST /check-sql](https://claix.dev/documentation/check-sql) and [claix.dev/guard](https://claix.dev/guard).

---

## API overview

### Authentication

```http
x-api-key: YOUR_API_KEY
```

`Authorization: Bearer YOUR_API_KEY` is also accepted. Prefer `x-api-key`.

API keys are server credentials. Keep them out of browser code, mobile apps, public repositories, and logs.

### Retrieve

```text
POST https://claix.dev/retrieve/{agent_name}
```

| Field | Description |
| --- | --- |
| `questions` | Up to 5 questions. Each one stands alone. |
| `questions[].question` | The question in plain language |
| `questions[].format` | Required in `razonada` mode. Optional in `bruto` mode |

Docs: [Call an agent](https://claix.dev/documentation/call-an-agent).

Routes and agents can also be managed over the API: `POST /routes`, `PUT /routes/edit/{route_id}`, `POST /routes/delete/{route_id}`, `POST /agents`, `PUT /agents/edit/{agent_id}`, `POST /agents/pause/{agent_id}`.

### MCP

Hosted server, version 1.13.0, 32 tools. Retrieve tools are `claix.retrieve.ask`, `claix.routes.*`, and `claix.agents.*`. The rest cover Ingest: schemas, extraction, agent mode, Markdown, multi-document questions, and JSON to Excel.

```json
{
  "mcpServers": {
    "claix": {
      "url": "https://claix.dev/mcp",
      "headers": {
        "x-api-key": "YOUR_API_KEY"
      }
    }
  }
}
```

Repository: [claix-dev/claix-mcp](https://github.com/claix-dev/claix-mcp). Docs: [claix.dev/documentation/mcp](https://claix.dev/documentation/mcp).

### A2A

Other agents discover Claix at [/.well-known/agent.json](https://claix.dev/.well-known/agent.json) and send tasks to `POST https://claix.dev/a2a`. Docs: [claix.dev/documentation/a2a](https://claix.dev/documentation/a2a).

### SDKs

The Python and TypeScript packages are the typed clients for **Claix Ingest**. Retrieve and Guard are on the HTTP API above. Those clients do not wrap those calls yet.

```bash
pip install claix-ai
npm install claix-ai
```

- Python: [Gaelproodoos/claix-python](https://github.com/Gaelproodoos/claix-python)
- TypeScript: [Gaelproodoos/claix-typescript](https://github.com/Gaelproodoos/claix-typescript)
- SDK docs: [claix.dev/documentation/sdks](https://claix.dev/documentation/sdks)

---

## Claix Ingest

Ingest gets documents into structured data. It is separate from Retrieve. Once the rows are in your database, a Retrieve agent can answer questions about them.

| Input | Endpoint |
| --- | --- |
| Excel / CSV → JSON | `POST /api/excel-json` |
| PDF → JSON | `POST /api/pdf-json` |
| Word, text, Markdown, RTF → JSON | `POST /api/doc-json` |
| Image → JSON | `POST /api/img-json` |
| Text, HTML, XML → JSON | `POST /api/txt-json` |
| Audio → JSON | `POST /api/audio-json` |
| File → Markdown | `POST /api/{type}-markdown` |
| Several files, up to 5 questions | `POST /multi-document` |

With your own LLM key, Ingest has no extra Claix processing fee. You pay your provider's tokens. Full reference: [claix.dev/documentation](https://claix.dev/documentation).

---

## Pricing

| | Claix fee | You also pay |
| --- | --- | --- |
| Retrieve, Claix Managed AI | €0.10 per question | — |
| Retrieve, your own LLM key (OpenAI, Google, Anthropic, or xAI) | €0.02 per question | Your provider's tokens |
| Guard | €0.01 per check | — |

Each question in a request counts as one question. With your own key, test the model in the Sandbox before you go live. Current rates are on the [pricing page](https://claix.dev).

---

## Security practices

- Create a dedicated read-only role for Retrieve. Grant `SELECT` only on the columns agents need.
- Keep API keys and database passwords in a secrets manager.
- The business prompt explains the job. It is not the permission. Permissions come from the database role and the routes.
- Treat text stored in your database as untrusted input. A field can contain instructions aimed at the agent.
- Review traces, and keep `log_id` next to important workflow decisions.
- Run Guard before your code executes an `INSERT`, `UPDATE`, or `DELETE`.
- Add your own check for payments, approvals, access changes, and legal decisions.

---

## Limits

- An agent can read everything its routes allow, for every end user. If each person should see only their own rows, enforce that in the database or in your application.
- Claix is hosted. This repository does not ship a self-hosted deployment.
- A model writes the Retrieve queries. A query can be valid, allowed, and still answer the wrong question. Clear table descriptions, explicit relationships, and traces are how you catch that.
- Retrieve does not write. Guard does not execute. Ingest inserts a row only when you turn Save Data on.
- Claix returns data and verdicts to your agent. It does not include dashboards or charts.

---

## Learn more

- [How to let an AI agent query your Postgres database safely](https://claix.dev/blog/how-to-let-an-ai-agent-query-postgres-safely)
- [Read-only Postgres role for AI agents](https://claix.dev/blog/read-only-postgres-role-for-ai-agents)
- [How to document your database for AI agents](https://claix.dev/blog/how-to-document-your-database-for-ai-agents)
- [How to test a text-to-SQL agent](https://claix.dev/blog/how-to-test-a-text-to-sql-agent)
- [Best database access layers for AI agents](https://claix.dev/blog/best-database-access-layer-for-ai-agents)

---

## Contributing

This repository is the public guide for the hosted Claix API.

Useful contributions: examples in Python, JavaScript, Node.js, PHP, and n8n; route and business-prompt examples; A2A and MCP client snippets; corrections when an endpoint in this file drifts from [the documentation](https://claix.dev/documentation).

Do not commit API keys, database credentials, private documents, or customer records.

For the latest endpoints, limits, supported databases, pricing, and terms, use the [Claix documentation](https://claix.dev/documentation).
