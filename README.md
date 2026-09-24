
```markdown
# Model Context Protocol (MCP) Guide

A standardized, open protocol designed to connect Large Language Models (LLMs) and AI applications to external data sources, tools, and environments securely.

---

## 1. What is MCP?

The **Model Context Protocol (MCP)** is an open standard introduced by Anthropic that standardizes how AI applications provide context to LLMs. 

Think of MCP as the **USB-C of AI integrations**: instead of building and maintaining custom API integrations for every combination of LLM, IDE, and data source, developers build an MCP server once, and any MCP-compatible client can interface with it seamlessly.

---

## 2. MCP Architecture

MCP follows a client-host-server architecture where communication occurs using standardized JSON-RPC 2.0 messages over transports like **stdio** (standard I/O) or **SSE** (Server-Sent Events over HTTP).




```

┌─────────────────────────────────────────────────────────────┐
│                         MCP Host                            │
│  (e.g., Claude Desktop, Cursor, n8n, Custom App)             │
│                                                             │
│   ┌───────────────┐                  ┌──────────────────┐   │
│   │   MCP Client  │                  │    LLM Engine    │   │
│   └───────┬───────┘                  └────────┬─────────┘   │
└───────────┼───────────────────────────────────┼─────────────┘
│                                   │
│ JSON-RPC (stdio / SSE)            │ Prompts &
│                                   │ Completions
▼                                   ▼
┌─────────────────────────┐           ┌───────────────────┐
│       MCP Server        │           │   Target Model    │
│  ┌───────────────────┐  │           └───────────────────┘
│  │ Resources         │  │
│  │ Prompts           │  │
│  │ Tools             │  │
│  └─────────┬─────────┘  │
└────────────┼────────────┘
│ Native APIs / DB Queries
▼
┌─────────────────────────┐
│ Local Files, DBs, APIs  │
└─────────────────────────┘

```



### Core Primitives
* **Host:** The container application initiating the workflow (e.g., n8n, Claude Desktop).
* **Client:** The component inside the host establishing 1:1 connections with servers.
* **Server:** A lightweight program exposing data and functionality via three core primitives:
  * **Resources:** Passive, read-only data (file contents, schema definitions, logs).
  * **Tools:** Executable functions that perform actions or computations.
  * **Prompts:** Pre-defined templates helping users and LLMs interact effectively.

---

## 3. MCP vs. Traditional AI Agents

| Feature | Traditional AI Agent | MCP-Enabled AI Agent |
| :--- | :--- | :--- |
| **Integration Pattern** | Bespoke function-calling schemas per framework (LangChain, LlamaIndex, custom). | Uniform JSON-RPC standard agnostic of agent framework. |
| **Portability** | Tools built for one platform rarely run on another without rewrites. | Write once: runs across Claude Desktop, Cursor, n8n, and custom clients. |
| **Context Isolation** | Agent often requires raw API credentials directly in memory. | MCP Server acts as an abstraction barrier; handles authentication locally. |
| **Scalability** | $M \times N$ complexity (every model needs connectors to every service). | $M + N$ complexity (models integrate with MCP; services expose MCP). |
| **Dynamic Discovery** | Tools must be hard-coded or manually configured per agent run. | Clients discover available tools, prompts, and resources at runtime. |

---

## 4. MCP Workflow


```

1. Client Connects     ──►  Initial handshake & capability exchange
2. Tool Discovery      ──►  Host requests `tools/list`
3. User Query          ──►  "Find pending invoices in Postgres and email a summary"
4. LLM Decision        ──►  Model requests execution of `query_db` via MCP
5. Server Execution    ──►  Server runs query against DB, returns structured output
6. Tool Response       ──►  Output injected into LLM context window
7. Final Output        ──►  LLM composes response / triggers secondary tools

```

---

## 5. Setting Up MCP in n8n

n8n can act as both an **MCP Host** (consuming external tools) and an **MCP Server** (exposing n8n workflows as tools to AI clients).

### Using n8n as an MCP Host (Connecting to MCP Servers)

1. **Deploy your MCP Server:**
   * Run your server with an SSE transport enabled (e.g., exposed on `http://localhost:3000/sse` or a public/internal domain).

2. **Add an AI Agent in n8n:**
   * In a new workflow, add the **AI Agent** node.
   * Attach your preferred chat model (e.g., OpenAI, Anthropic).

3. **Attach the MCP Tool:**
   * In the **Tools** input of the AI Agent node, select the **MCP Tool** node (or **HTTP Request / Custom Tool** configured to communicate with the MCP SSE endpoint).
   * Specify the server connection parameters:
     * **Transport:** `Server-Sent Events (SSE)`
     * **Endpoint URL:** `https://your-mcp-server.example.com/sse`
     * **Authentication:** Bearer token or API key headers if required.

4. **Test the Integration:**
   * Trigger the agent with a chat prompt.
   * Inspect execution data to observe the agent performing tool discovery and invoking actions over MCP.

---

## 6. Key Advantages

* **Plug-and-Play Extensibility:** Instantly add file systems, GitHub, PostgreSQL, Slack, or web search to any agent without custom SDK glue.
* **Separation of Concerns:** Keep API authentication, logic, and data sanitization within the MCP server while the agent focuses purely on reasoning.
* **Security & Control:** Set clear boundary permissions on resources and enforce human-in-the-loop approvals on executable tools.
* **Vendor Independence:** Switch underlying LLMs (OpenAI, Anthropic, local models) without updating your tools or data connectors.

```
