# MCP + LangGraph Demo (Multi-Tool Agent)

A minimal, production-style example showing how to:

- run **multiple MCP servers** (FastMCP)
- connect them via **`MultiServerMCPClient`**
- let a **LangGraph ReAct agent** call tools from those servers

This repository includes:

- `mathserver.py` — tools: `add`, `multiply` (MCP over **stdio**)
- `weather.py` — tool: `get_weather` (MCP over **streamable HTTP**)
- `client.py` — loads tools from both servers and runs an agent with Groq (`ChatGroq`)

> Note: `main.py` is currently a placeholder and not used by the demo.

---

## Features

- Multi-server MCP tool discovery via `MultiServerMCPClient`
- Tool calling via `langgraph.prebuilt.create_react_agent`
- Includes two transports:
  - `stdio` (math)
  - `streamable_http` (weather)

---

## Architecture

```text
+-------------------+          MCP tools (stdio)
|  client.py        | <------------------------------------+
|  ReAct agent      |                                       |
|  (LangGraph)      |                                       |
+-------------------+                                       |
          |                                                   |
          | MCP tools (streamable_http)                     |
          v                                                   |
+-------------------+                                       |
|  weather.py       |  FastMCP tool: get_weather            |
|  MCP over HTTP    |---------------------------------------+
+-------------------+
        ^
        |
        | FastMCP tool: add, multiply
        +-------------------+
                            mathserver.py
                            MCP over stdio
```

---

## Prerequisites

- Python **3.12**+
- A Groq API key for `ChatGroq`

---

## Setup

### 1) Create and activate a virtual environment

```bash
python -m venv .venv
source .venv/bin/activate
```

### 2) Install dependencies

```bash
pip install -r requirements.txt
```

> The project’s canonical dependencies are also declared in `pyproject.toml`.

### 3) Set your Groq API key

```bash
export GROQ_API_KEY="<your_key>"
```

---

## How to Run

### Terminal A: Start the math MCP server (stdio)

```bash
python mathserver.py
```

This server exposes:

- `add(a: int, b: int) -> int`
- `multiply(a: int, b: int) -> int`

### Terminal B: Start the weather MCP server (streamable HTTP)

```bash
python weather.py
```

The client expects it at:

- `http://127.0.0.1:8000/mcp`

### Terminal C: Run the agent

```bash
python client.py
```

The agent will:

- ask the math server to compute `(3+5)*12`
- ask the weather server for a response about the California

---

## Example Prompts

`client.py` uses these prompts by default:

- **Math:** `What is (3+5)* 12?`
- **Weather:** `What is the weather of the California?`

---

## Configuration Notes (Transports)

- **Math (`mathserver.py`) uses `stdio`**
  - `client.py` launches it with:
    - `command: "python"`
    - `args: ["mathserver.py"]`
    - `transport: "stdio"`

- **Weather (`weather.py`) uses `streamable_http`**
  - `client.py` connects to:
    - `url: "http://127.0.0.1:8000/mcp"`
    - `transport: "streamable_http"`

If you change either transport or port/endpoint, update `client.py` accordingly.

---

## Troubleshooting

### Tool discovery returns empty or the agent can’t call tools
- Ensure both servers are running before starting `client.py`.

### Weather server connection errors (HTTP)
- Verify the weather server is listening on `8000`.
- Confirm the endpoint matches `http://127.0.0.1:8000/mcp`.

### Groq errors
- Confirm `GROQ_API_KEY` is set in your shell.
- If you want to use a different model, update `client.py`:
  - `model=ChatGroq(model="qwen/qwen3-32b")`

---

## Project Structure

- `client.py` — Multi-server MCP client + LangGraph agent runner
- `mathserver.py` — MCP FastMCP server exposing arithmetic tools
- `weather.py` — MCP FastMCP server exposing a weather tool
- `main.py` — placeholder entry point (not used by the demo)

---

## License

MIT (add license text if you plan to publish)

