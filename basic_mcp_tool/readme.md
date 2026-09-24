# Model Context Protocol (MCP) Client & Server in n8n

This repository contains the configuration for a distributed AI agent system using the Model Context Protocol (MCP) across two distinct n8n workflows. By splitting the logic into a Client and a Server, the architecture decouples the AI reasoning engine from the execution of specific tools.

## Workflow Overview

The system is composed of two imported JSON files:

* **mcp client.json**: Acts as the user-facing interface and AI reasoning engine.


* **mcp server.json**: Acts as the backend capability provider, exposing a suite of executable tools via an MCP endpoint.



### 1. The MCP Client Workflow (`mcp client.json`)

The client workflow handles direct interaction with the user and houses the main LangChain agent.

* **Trigger:** The workflow is initiated by a "Telegram Trigger" node that listens for incoming user text messages.


* **AI Agent:** The core processing is handled by an AI Agent node using the OpenAI `gpt-4.1-mini` chat model.


* **Memory Management:** A "Simple Memory" (Buffer Window) node retains conversational context up to a length of 3, tracking sessions using the Telegram chat ID as a custom session key.


* **MCP Connection:** The agent is equipped with an "MCP Client" tool. This node connects to the remote server endpoint (`[https://n8n-2prp.srv1981742.hstgr.cloud/mcp/9324e101-193d-46fd-9b3c-4994f04ab48b](https://n8n-2prp.srv1981742.hstgr.cloud/mcp/9324e101-193d-46fd-9b3c-4994f04ab48b)`) using Server-Sent Events (SSE) transport.


* **Output:** The agent's final generated response is sent back to the user via a Telegram "Send a text message" node.



### 2. The MCP Server Workflow (`mcp server.json`)

The server workflow runs independently and waits for incoming requests from the MCP Client. It exposes various automated tools that the client AI can invoke.

* **Trigger:** The workflow is initiated by an "MCP Server Trigger" listening on the specific path `9324e101-193d-46fd-9b3c-4994f04ab48b`.


* **Exposed Capabilities:** The server provides five distinct tools connected directly to the trigger:
* **Calculator:** A basic math evaluation tool.


* **Gmail:** A tool to draft and send emails via Google OAuth2.


* **Google Calendar:** A tool to create calendar events, configured to use the email address `haripriyadesai27@gmail.com`.


* **Google Search (SerpApi):** A standard web search tool.


* **Amazon Search (SerpApi):** A specialized product search tool restricted to the `amazon.in` domain.





## System Execution Flow

When a user interacts with the system, the execution follows this lifecycle:

1. A user sends a message to the connected Telegram bot, which triggers the client workflow.


2. The LangChain AI agent processes the prompt and reviews the conversational memory.


3. The agent communicates with the MCP Client tool, which discovers the capabilities exposed by the remote MCP Server via the SSE connection.


4. If the user's request requires action (e.g., searching Amazon for a product or creating a calendar event), the client sends an execution request to the server workflow.


5. The MCP Server Trigger receives the request, executes the specified tool (such as SerpApi or Google Calendar), and passes the raw data back to the client.


6. The AI agent on the client side parses the returned data, formats a human-readable response, and sends it to the user's Telegram chat.



## Configuration & Prerequisites

To deploy this setup successfully, you will need to authenticate the nodes with the following credentials across both workflows:

* **Telegram:** API credentials for the bot trigger and messaging node.


* **OpenAI:** API key to run the `gpt-4.1-mini` model.


* **Google OAuth2:** Required on the server side for both Gmail and Google Calendar access.


* **SerpApi:** An API key is required on the server side to enable both Google and Amazon web searches.
