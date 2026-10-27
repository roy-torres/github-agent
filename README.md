# github-agent

An agent project that works with GitHub through the **Model Context Protocol (MCP)**.

## What is the Model Context Protocol?

The Model Context Protocol (MCP) is an open standard, introduced by Anthropic in November 2024, that defines how AI applications connect to external tools, data sources, and services. It is often described as a "USB-C port for AI": one common interface instead of a custom integration for every model/tool pair.

Before MCP, connecting M applications to N tools meant building M x N bespoke integrations. With MCP, each application implements the client side once and each tool implements the server side once.

## Architecture

MCP uses a client-server model built on JSON-RPC 2.0:

| Role | Description |
| --- | --- |
| **Host** | The AI application the user interacts with (e.g. Cursor, Claude Desktop, an IDE or custom agent). |
| **Client** | A connector inside the host that maintains a 1:1 connection with one MCP server. |
| **Server** | A lightweight program that exposes capabilities (tools, data, prompts) from a system such as GitHub, a database, or the filesystem. |

## Core Primitives

Servers can expose:

- **Tools** - Functions the model can call to take actions (e.g. create an issue, open a pull request, push a file).
- **Resources** - Read-only data that provides context (e.g. file contents, documents, database records).
- **Prompts** - Reusable prompt templates and workflows that users can invoke.

Clients can offer servers:

- **Sampling** - Lets a server request an LLM completion through the host.
- **Roots** - Lets a server learn which filesystem/URI boundaries it may operate within.
- **Elicitation** - Lets a server ask the user for additional input.

## Transports

- **stdio** - The host launches the server as a local subprocess and communicates over standard input/output. Best for local tools.
- **Streamable HTTP** - The server runs remotely and communicates over HTTP (with optional streaming via Server-Sent Events). Best for hosted services; supports authorization such as OAuth.

## Typical Flow

1. The host starts a client, which connects to a server and performs an **initialize** handshake to negotiate capabilities.
2. The client lists available tools/resources/prompts (e.g. `tools/list`).
3. The model decides to use a tool and the client sends a `tools/call` request.
4. The server executes the action (such as calling the GitHub API) and returns the result to the model.

## Example: GitHub MCP Server

The GitHub MCP server exposes GitHub functionality as MCP tools, such as:

- Creating and managing repositories, branches, and files
- Reading and writing issues and pull requests
- Searching code, commits, and repositories
- Inspecting GitHub Actions workflows and security alerts

This lets an agent perform GitHub tasks (like creating this README) through natural-language requests.

## Security Considerations

- Grant servers the **least privilege** they need (scoped tokens, limited permissions).
- Review tool calls that perform destructive or irreversible actions.
- Only install MCP servers from sources you trust; treat tool output as untrusted input (prompt injection).

## Resources

- Official site: https://modelcontextprotocol.io
- Specification: https://modelcontextprotocol.io/specification
- GitHub organization: https://github.com/modelcontextprotocol
- GitHub MCP server: https://github.com/github/github-mcp-server
