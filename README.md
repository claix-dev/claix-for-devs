# Claix: Controlled Database Retrieval for AI Agents

> Connect your Postgres or Supabase database and give each AI agent controlled, read-only access to the tables and columns you choose. Agents ask in plain language and get validated results or typed answers over REST, MCP and A2A.

[![Claix](https://img.shields.io/badge/Claix-Database%20Retrieval-00C853?style=flat-square)](https://claix.dev)
[![REST](https://img.shields.io/badge/API-REST-0EA5E9?style=flat-square)](https://claix.dev/documentation)
[![A2A](https://img.shields.io/badge/Protocol-A2A-7C3AED?style=flat-square)](https://claix.dev/.well-known/agent-card.json)
[![MCP](https://img.shields.io/badge/Protocol-MCP-7C3AED?style=flat-square)](https://claix.dev/mcp)
[![Pricing](https://img.shields.io/badge/Pricing-Pay%20per%20question-16A34A?style=flat-square)](https://claix.dev)

Claix is a hosted **controlled database retrieval layer for AI agents**. It sits between your agent and your relational database, so the agent never has to write SQL against your tables or hold a database connection itself.

```text
Your Postgres or Supabase database   (read-only role)
        ↓
Data Map: tables, columns and relationships
        ↓
Routes: which tables and columns can be used, and how
        ↓
Agent: its own endpoint, business prompt and routes
        ↓
Validated results or typed answers
        ↓
Your application, workflow or AI agent
```

Claix does not copy your data or build an index over it. Agents read from your database where it lives, only from what you allow.

🌐 **Website:** [claix.dev](https://claix.dev)  
📚 **Documentation:** [claix.dev/documentation](https://claix.dev/documentation)  
📄 **OpenAPI specification:** [claix.dev/openapi.yaml](https://claix.dev/openapi.yaml)  
🤖 **A2A Agent Card:** [claix.dev/.well-known/agent-card.json](https://claix.dev/.well-known/agent-card.json)  
🔌 **MCP server:** [claix.dev/mcp](https://claix.dev/mcp)  

---

## Why Claix?

An agent that needs real data from your database has two common options, and both have problems:

- **Build the data access yourself** with a framework: query tools, permission checks and logging, written, hosted and maintained for every agent.
- **Give the model a database connection** and let it write SQL. That works in a demo and is risky in production: a model can be manipulated by instructions hidden in text it reads, it can write a query that runs fine and returns the wrong answer without any error, and nobody can easily see what it did.

Claix replaces both. You connect your database with a read-only role, choose which tables and columns each agent can use, and write a short note in plain language about your business rules. There is no code to write and nothing to host. Your agent sends questions to its endpoint and gets typed answers back.

The difference from a framework is that Claix is not something you build with. It is a service you call.

An agent should reason about the task it was given, not about your schema, your SQL and your permissions. Keeping those jobs apart means the agent only sees what you allow, a manipulated prompt can't give it more access than you configured, and you can see and test every answer.

---

## Key capabilities

| Capability | What it does |
|---|---|
| **Read-only access** | Connects with a database role you create and limit |
| **Data Map** | Describes your tables, columns and relationships in plain language, drafted for you to review |
| **Routes** | Define which tables and columns can be used, with a short guide for your rules |
| **Agents** | Each agent has its own endpoint, business prompt and routes |
| **Typed answers** | Ask for a string, integer, boolean, array or object and get that type back |
| **Several questions per request** | Send a list of questions and get the answers in the same order |
| **Traces** | Every request is recorded: the question, the route, the query and the result |
| **Sandbox** | Test your agents with real questions before connecting them to your application |
| **REST, MCP and A2A** | Call agents from code, MCP clients, or other agents |
| **Managed AI or BYOK** | Use the Claix model stack, or bring your own supported provider key |

---

## Table of contents

- [Quick start](#quick-start)
- [Concepts](#concepts)
- [API overview](#api-overview)
- [Response modes](#response-modes)
- [Integrations](#integrations)
- [Claix Ingest](#claix-ingest)
- [Error handling](#error-handling)
- [LLM keys and pricing](#llm-keys-and-pricing)
- [Security practices](#security-practices)
- [Limits and what Claix does not do](#limits-and-what-claix-does-not-do)
- [Learn more](#learn-more)

---

# Quick start

## 1. Create a read-only role in your database

Create a dedicated role limited to the tables and columns you want agents to use. Grant only what is needed.

```sql
CREATE ROLE claix_agent WITH LOGIN PASSWORD 'use-a-secret-manager'
  NOSUPERUSER NOCREATEDB NOCREATEROLE NOINHERIT NOREPLICATION NOBYPASSRLS
  CONNECTION LIMIT 10;

GRANT CONNECT ON DATABASE mydb TO claix_agent;
GRANT USAGE ON SCHEMA public TO claix_agent;

-- Grant only the columns agents may use
GRANT SELECT (id, name, country) ON public.customers TO claix_agent;
GRANT SELECT (id, customer_id, status, total) ON public.orders TO claix_agent;
```

> [!IMPORTANT]
> The role is the real barrier. Anything the role can read, an agent can ask for. Grant as little as you can, and never use a superuser or your application's main database user.

## 2. Connect your database

In the Claix dashboard, open **Database connection** and add the credentials of the `claix_agent` role. Claix supports **Postgres and Supabase** today.

## 3. Review the Data Map

Claix reads the structure your role can see and drafts a **Data Map**: descriptions for each table, the relationships it finds and the ones it suggests. Review it, correct it and publish it.

## 4. Create a route

A route lists the tables and columns that can be used for one purpose, and a short guide with your rules and vocabulary. For example, a route for order status might allow `orders.status` and `orders.total`, with a note saying that `cancelled` orders stay in the table.

## 5. Create an agent

An agent has:

- A **name**, which becomes its endpoint: `/agent/{name}`.
- A short **description**.
- A **business prompt** that says what the agent does and when to use each route.
- The **routes** it can use.
- A **response mode**: raw results or reasoned, typed answers.

## 6. Ask a question

```bash
curl -X POST "https://claix.dev/agent/hr-assistant" \
  -H "x-api-key: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "questions": [
      { "question": "What is the phone number of the employee named Carlos?", "format": "string" },
      { "question": "Has the customer Javier Perez accepted marketing consent?", "format": "boolean" },
      { "question": "What is the job title of Marta?", "format": "string" }
    ]
  }'
```

<!-- VERIFY: request body shape, field names and the endpoint URL against the live API before publishing. -->

```json
[
  {
    "user_ask": [
      { "question": "What is the phone number of the employee named Carlos?", "format": "string" },
      { "question": "Has the customer Javier Perez accepted marketing consent?", "format": "boolean" },
      { "question": "What is the job title of Marta?", "format": "string" }
    ],
    "ia_response": [null, true, "Waiter"],
    "log_id": "9ff38974-1392-4c1a-83e5-3b2f09bb43dc"
  }
]
```

Answers come back in the same order as the questions. A `null` means the answer is not in the data the agent is allowed to use. Claix doesn't invent a value.

<!-- VERIFY: null semantics and the response shape. -->

---

# Concepts

| Concept | What it is |
|---|---|
| **Connection** | Your database, linked with a read-only role |
| **Data Map** | A description of your tables, columns and relationships. Published as a version, so changes are deliberate |
| **Route** | A set of tables and columns that can be used for one purpose, plus a guide with your rules |
| **Agent** | The identity that asks questions: its own endpoint, business prompt and routes |
| **Response mode** | Whether the agent returns raw results or reasoned, typed answers |
| **Trace** | A record of each request: the question, the route chosen, the query that ran and the result |
| **Sandbox** | A place to test an agent with real questions |

A route defines what can be retrieved, and an agent defines who retrieves it and with which routes. Routes can be shared between agents, so a support bot and an internal assistant can use the same database and still see different data.

### Writing a good business prompt

The business prompt is guidance, not security. Use it to explain the agent's job, when to use each route, what your terms mean and what the agent should not answer. What an agent can actually access is defined by its routes and by your database role.

```text
You are the reservations assistant for a restaurant group.
Use the "Bookings" route for questions about reservations and tables.
Use the "Staff" route for questions about shifts and roles.
Do not answer questions about payments. Say that information is not available.
"Confirmed" means status C; "pending" means P; "cancelled" means X.
```

---

# API overview

## Authentication

All Claix API calls require an API key. Use one of these headers:

```http
x-api-key: YOUR_API_KEY
```

or:

```http
Authorization: Bearer YOUR_API_KEY
```

The dedicated `x-api-key` header is recommended.

> [!IMPORTANT]
> Claix API keys are server credentials. Do not expose them in browser code, mobile apps, public repositories, screenshots, client-side workflows or logs. Store them in server-side environment variables or a secrets manager. A key gives access to everything the agents it can call are allowed to see.

## Agent endpoint

```text
POST https://claix.dev/agent/{name}
```

| Field | Description |
|---|---|
| `questions` | A list of questions. Up to 5 per request |
| `questions[].question` | The question in plain language |
| `questions[].format` | The type you expect: `string`, `integer`, `boolean`, `array` or `object` |

<!-- VERIFY: limits, supported format values and per-question length limit. -->

Each question is answered independently. Questions are not aware of each other, so write each one so it makes sense on its own.

## Response fields

| Field | Description |
|---|---|
| `user_ask` | The questions you sent |
| `ia_response` | The answers, in the same order, typed as requested |
| `log_id` | The identifier of the request. Use it to find the full trace in the dashboard |

## `log_id`

Every request includes a `log_id`. Use it to:

- Find the exact request in your Claix traces.
- See which route answered and what query ran.
- Correlate Claix with your own workflow run.
- Provide a support reference and build an audit trail.

---

# Response modes

Each agent has a response mode.

| Mode | What you get | Best for |
|---|---|---|
| **Data** | The results of the query as they are, without a second model call | Code and other agents that process the result |
| **Answer** | A reasoned answer in the type you ask for | Results that go straight to a person, or workflows that need a single value |

In **Answer** mode, the query results are sent to the model to write the answer. That adds a model call, which means more latency and cost. It also means your data passes through the model provider, so consider that for sensitive data. **Data** is the default.

<!-- VERIFY: the exact shape of the Data mode response. -->

---

# Integrations

Every agent is reachable over three interfaces.

## REST

Call the agent endpoint from your backend, scripts, n8n, Make, Zapier or any tool that can send an HTTP request.

## MCP

Claix exposes a hosted MCP server for AI assistants, coding tools and MCP-capable clients.

```text
https://claix.dev/mcp
```

Server metadata:

```text
https://claix.dev/.well-known/mcp/server-card.json
```

Example client configuration:

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

<!-- VERIFY: which tools the MCP server exposes for agents. See the MCP documentation: https://claix.dev/documentation/mcp -->

## A2A

Claix is a native **Agent-to-Agent (A2A)** agent. A2A-compatible agents and orchestrators can discover Claix through its public Agent Card and delegate data questions to it.

```text
https://claix.dev/.well-known/agent-card.json
```

A2A JSON-RPC endpoint:

```text
https://claix.dev/a2a
```

For complete A2A documentation, see [claix.dev/documentation/a2a](https://claix.dev/documentation/a2a).

## Python SDK

```bash
pip install claix
```

See the [SDK documentation](https://claix.dev/documentation/sdks) for the methods currently available.

---

# Claix Ingest

**Claix Ingest** is a separate product for getting data into your database. It turns documents, spreadsheets, images, text, HTML, XML and audio into schema-defined, source-traceable JSON, which you can then load into your own tables. Once the data is in your database, Claix Retrieve can answer questions about it.

| Input type | Endpoint |
|---|---|
| Excel / CSV | `POST /api/excel-json` |
| PDF | `POST /api/pdf-json` |
| Documents (Word, text, markdown, RTF) | `POST /api/doc-json` |
| Images | `POST /api/img-json` |
| Text / HTML / XML | `POST /api/txt-json` |
| Audio | `POST /api/audio-json` |

Ingest supports source tracing, persistent document context and Knowledge Spaces. Existing integrations continue to work. See the [documentation](https://claix.dev/documentation) for the full reference.

---

# Error handling

| Status | Meaning | Retry? |
|---:|---|---|
| `200` | Request completed successfully | — |
| `400` | Invalid request, missing fields or malformed payload | No; correct the request |
| `401` | Missing, invalid, inactive or suspended API key | No; correct credentials |
| `404` | The agent does not exist or belongs to a different account | No; correct the name |
| `405` | Incorrect HTTP method | No; use the documented method |
| `413` | Request exceeds limits | No; reduce the input |
| `429` | Rate limit exceeded | Yes; follow `Retry-After` where available |
| `500` | Internal Claix error | Retry carefully |
| `502` | Temporary AI-provider or processing failure | Retry with exponential backoff |

<!-- VERIFY: status codes and meanings for the agent endpoint, including paused agents. -->

Error responses use a consistent shape:

```json
{
  "error": "Human-readable error message.",
  "detalle": "Optional technical detail.",
  "log_id": "9ff38974-1392-4c1a-83e5-3b2f09bb43dc"
}
```

Recommended retry policy:

```text
Retry:
429, 500, 502

Do not retry without changing the request:
400, 401, 404, 405, 413
```

---

# LLM keys and pricing

You choose which model runs your agents.

| Option | Best for | Billing |
|---|---|---|
| **Claix Managed AI** | Teams that want a managed model with no provider setup | €0.10 per question |
| **Bring Your Own Key (BYOK)** | Teams that want to choose their model and control provider costs | €0.02 per question, plus your provider's usage |

<!-- VERIFY: prices and the free-question allowance against the current pricing page. -->

- Pricing is per question. Each question in a request counts as one question.
- BYOK does not mean AI inference is free. Your provider bills you for tokens and any other usage.
- With your own key, results depend on the model you choose, so test it in the Sandbox with your own routes before going live.
- Free questions are included to start. See the [pricing page](https://claix.dev) for current details.

Add and manage keys in **LLM Keys** in the dashboard. Keys are stored encrypted and used only to run your agents' requests.

---

# Security practices

- Create a dedicated **read-only** database role for Claix, with `SELECT` on only the tables and columns agents need. Prefer views for anything sensitive.
- Do not use a superuser or your application's main database user.
- Keep API keys and database passwords server-side, in a secrets manager. Rotate them regularly.
- Use separate API keys for development, staging and production when possible.
- Treat everything in your database as untrusted input. A text field can contain instructions aimed at an agent, so do not let query results override your application's rules.
- Don't put secrets or personal data in table comments, routes or the business prompt.
- Do not rely on the business prompt for permissions. Permissions come from your database role and your routes.
- Review traces regularly, and keep the `log_id` alongside important workflow decisions.
- Add your own validation for high-impact actions, such as payments, approvals, access changes or legal decisions.
- Rotate the database password if someone who knew it leaves your team.

---

# Limits and what Claix does not do

- **Per-agent access, not per end user.** An agent can read everything its routes allow, for every user. If each person should only see their own rows, enforce that in your database or your application.
- **Postgres and Supabase today.** Other databases are not supported yet.
- **A hosted service.** Claix is not self-hosted.
- **A model writes the queries.** A query can be valid, allowed and still answer the wrong question. Clear table descriptions, explicit relationships and traces help you catch that.
- **Read-only.** Claix reads from your database. It does not change your data. Loading data is a separate step, for example with Claix Ingest.
- **Not an analytics tool.** Claix returns data and answers to your agent. It does not include dashboards or charts.

---

# Learn more

- [How to let an AI agent query your Postgres database safely](https://claix.dev/blog/how-to-let-an-ai-agent-query-postgres-safely)
- [Read-only Postgres role for AI agents](https://claix.dev/blog/read-only-postgres-role-for-ai-agents)
- [How to document your database for AI agents](https://claix.dev/blog/how-to-document-your-database-for-ai-agents)
- [How to test a text-to-SQL agent](https://claix.dev/blog/how-to-test-a-text-to-sql-agent)
- [Best database access layers for AI agents](https://claix.dev/blog/best-database-access-layer-for-ai-agents)

<!-- VERIFY: update these links to match the published URLs. -->

---

# Contributing

This repository contains examples and integration guidance for the hosted Claix API.

Contributions are welcome for:

- Examples in Python, JavaScript, Node.js, PHP and n8n.
- Route and business-prompt examples for common use cases.
- A2A and MCP client integrations.
- Documentation improvements.
- Security and reliability guidance.

Please do not commit real API keys, database credentials, private documents or customer records.

---

# License

Unless otherwise stated, code examples and documentation in this repository are licensed under the [MIT License](LICENSE).

---

# Disclaimer

Claix is a hosted service. This repository provides API examples and integration guidance; it does not provide a self-hosted Claix deployment.

For the latest endpoint details, limits, supported databases, pricing and product terms, refer to the official [Claix documentation](https://claix.dev/documentation).
