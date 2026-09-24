# Understanding the Model Context Protocol (MCP)

This document provides a comprehensive, easy-to-understand breakdown of the **Model Context Protocol (MCP)**. Whether you are a developer looking to build integrations or an AI enthusiast wanting to understand how AI assistants interact with external data, this guide covers the core concepts, architecture, and practical use cases of MCP.

---

## 1. What is MCP? (The "Elevator Pitch")

Historically, giving an AI assistant access to your local files, databases, or SaaS tools required writing custom, one-off integrations for every single application.

**The Model Context Protocol (MCP) is an open standard that solves this.** Think of it as the "USB-C plug for AI." It provides a universal, standardized way for AI applications to connect to external data sources and tools securely.

Instead of an AI company building separate integrations for Google Drive, GitHub, local file systems, and Slack, developers can build a single **MCP Server** for their data. Any AI application that supports the MCP standard can then instantly connect to it.

---

## 2. Core Architecture: The "Who's Who"

MCP uses a standard client-server architecture, but with specific terminology tailored for AI interactions. There are three main components you need to understand:

### A. The MCP Host

The Host is the application the user is directly interacting with. This is usually an AI-powered interface.

* **Examples:** Claude Desktop, Cursor (Code Editor), Windsurf, or a custom AI chat application.
* **Role:** The host takes your input, communicates with the Large Language Model (LLM), and decides when to ask the MCP Client to fetch data or run a tool.

### B. The MCP Client

The Client is the engine running *inside* the Host application.

* **Role:** It maintains 1:1 connections with various MCP Servers. When the AI model says, "I need to read this file," the MCP Client formats that request according to the protocol and sends it to the appropriate server.

### C. The MCP Server

The Server is a lightweight program that acts as a bridge between the AI and your actual data or tools.

* **Examples:** A local SQLite database server, a GitHub server, or a server that reads your local file system.
* **Role:** It exposes specific capabilities (Resources, Tools, and Prompts) to the client. It handles the actual execution—reading the file, running the SQL query, or calling the external API—and returns the result to the Client.

---

## 3. The Three Pillars of MCP

An MCP Server can expose three distinct types of capabilities to an AI model. Understanding these three primitives is the key to mastering MCP.

### Pillar 1: Resources (Data the AI can READ)

Resources are pieces of data that the server exposes to the AI. They are meant to be **read-only** and provide context.

* **How they work:** They are identified by URIs (Uniform Resource Identifiers), similar to website URLs.
* **Examples:**
* `file:///users/documents/notes.txt` (A local file)
* `postgres://database/customers/schema` (A database schema)
* `api://github/issues/123` (An API response)


* **When to use:** When the AI needs background information or data to analyze before answering a user's question.

### Pillar 2: Tools (Actions the AI can TAKE)

Tools are executable functions that allow the AI to perform actions or fetch highly dynamic data that requires parameters.

* **How they work:** The server defines a tool (e.g., `execute_sql_query`) and specifies the required arguments (e.g., `query: string`). The AI decides to call the tool, passes the arguments, and the server executes it.
* **Examples:**
* `write_file(path, content)`
* `search_web(query)`
* `restart_server()`


* **When to use:** When the AI needs to create, update, or delete something, or when it needs to filter data based on user input (like running a specific search query). **Note:** Because tools take action, they always require a "Human in the Loop" approval step in most Host applications to ensure security.

### Pillar 3: Prompts (Instructions the server PROVIDES)

Prompts are reusable, server-defined templates that help users instruct the AI on how to interact with the server's data.

* **How they work:** Instead of the user typing a long prompt every time, the server can provide a pre-packaged instruction set.
* **Examples:** A GitHub MCP server might expose a prompt called `review_code`. When a user invokes this prompt, the server automatically attaches the code from the current pull request and instructs the AI, "Review this code for security vulnerabilities and style guide violations."
* **When to use:** To streamline complex workflows and ensure the AI gets the exact instructions it needs to work with a specific domain's data.

---

## 4. How They Communicate (Transports)

MCP defines standardized ways for the Client and Server to send messages back and forth. These are called Transports. Currently, there are two primary transport mechanisms:

1. **Stdio (Standard Input/Output):**
* **Use case:** Local connections.
* **How it works:** The MCP Client launches the MCP Server as a local sub-process on your computer. They communicate directly through standard command-line streams. This is highly secure because no data leaves your machine; it's perfect for local file access or local databases.


2. **SSE (Server-Sent Events) over HTTP:**
* **Use case:** Remote connections.
* **How it works:** The MCP Server is hosted in the cloud (like a standard web API). The Client connects to it over the internet using SSE to receive real-time updates and standard HTTP POST requests to send commands.



## 5. A Typical MCP Workflow Example

To tie it all together, here is what happens when you ask an MCP-enabled AI (like Claude Desktop) a question:

1. **User Request:** You type, "Summarize the errors in my local web server log."
2. **Capability Check:** Claude knows it is connected to a "Local File System MCP Server."
3. **Tool/Resource Call:** Claude decides it needs to read the log file. It asks the MCP Client to fetch the resource `file:///var/logs/apache/error.log`.
4. **Server Execution:** The MCP Client uses Stdio to ask the local MCP Server for that file. The server reads the file from your hard drive and sends the text back.
5. **AI Processing:** Claude receives the log text, analyzes it, and generates a human-readable summary.
6. **Response:** Claude displays the summary to you on the screen.
