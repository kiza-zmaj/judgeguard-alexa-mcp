# 🛡️ JudgeGuard — Alexa+ MCP Governance Server

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](https://opensource.org/licenses/MIT)
[![Python: 3.12+](https://img.shields.io/badge/Python-3.12%2B-brightgreen.svg)](https://www.python.org/)
[![MCP Spec: 2025-11-25](https://img.shields.io/badge/MCP-2025--11--25-orange.svg)](https://modelcontextprotocol.io/)
[![Track: Alexa+](https://img.shields.io/badge/Track-Alexa%2B-blueviolet.svg)](https://amazonappdev2026.devpost.com/)
[![AWS: Bedrock](https://img.shields.io/badge/AWS-Bedrock%20Claude%203.5%20%7C%20Titan-FF9900.svg)](https://aws.amazon.com/bedrock/)
[![RAG: NotebookLM](https://img.shields.io/badge/RAG-Google%20NotebookLM-4285F4.svg)](https://notebook.google.com/)

**JudgeGuard** is an autonomous AI governance gatekeeper and safety bridge designed for the **Amazon Developer Hackathon 2026** ([amazonappdev2026.devpost.com](https://amazonappdev2026.devpost.com/)).

It implements a self-hosted **Model Context Protocol (MCP)** server over **Streamable HTTP (MCP Spec 2025-11-25+)** to safeguard Alexa+ agentic interactions, prevent unauthorized high-risk operations (e.g. unlocking doors, financial transactions, destructive commands), and provide verifiable RAG grounding via **Google NotebookLM** and deep semantic reasoning via **AWS Bedrock**.

---

## 🏆 Hackathon Tracks & Challenges

| Track / Challenge | Implementation Path | Implementation Evidence |
| :--- | :--- | :--- |
| **Primary Track: Alexa+** | **Plan A (Core):** Self-hosted MCP Server implementing MCP spec **2025-11-25+** over **Streamable HTTP** with runtime tool invocation.<br>**Plan B (Visual Demo):** Custom **Alexa+ Experience Web Simulator** (`static/index.html`). | [`server.py`](server.py), [`static/index.html`](static/index.html) |
| **AWS Builder Mini-Challenge** | Direct runtime integration with **AWS Bedrock Runtime** using **Claude 3.5 Sonnet** and **Amazon Titan Text Express**. | [`bedrock_client.py`](bedrock_client.py), [`AWS_PRODUCT_FEEDBACK.md`](AWS_PRODUCT_FEEDBACK.md) |
| **Open Source Mini-Challenge** | Independent, standalone open-source repository under the **MIT License**, created and developed entirely within the hackathon window. | [`LICENSE`](LICENSE), [`pyproject.toml`](pyproject.toml) |

> **Note for Judges Regarding Alexa+:**  
> The included Web Simulator (`http://127.0.0.1:8765/simulator`) is **our custom demonstration web application** created to showcase real-time agentic workflows, SSE event streaming, and the JudgeGuard HUD. It is not Amazon's internal Alexa+ simulator. The server itself is a compliant, self-hosted MCP server communicating via Streamable HTTP.

---

## 🏗️ Architecture & Verification Pipeline

```
User Voice / Action Request
            ↓
   Alexa+ Agent / Host
            ↓ (MCP Streamable HTTP POST /mcp: tools/call)
┌──────────────────────────────────────────────────────────────────┐
│                   JudgeGuard Governance Gateway                  │
│                                                                  │
│  Layer 00: Destructive & Dangerous Shell/Command Filter          │
│  Layer 01: Boundary & Scope Check (Door locks, payments, etc.)  │
│  Layer 02: Google NotebookLM RAG Policy Grounding (Cited facts) │
│  Layer 03: AWS Bedrock Runtime Reasoning (Claude 3.5 / Titan)    │
└──────────────────────────────────────────────────────────────────┘
            ↓
         VERDICT
   ├── 🟢 PASSED  → Tool executes safely; audit event broadcast over SSE
   └── 🛑 BLOCKED → Action halted safely; incident logged to audit trail
```

---

## ⚡ Quickstart: Running & Testing

### 1. Clone & Setup Environment
```bash
git clone https://github.com/kiza-zmaj/judgeguard-alexa-mcp.git
cd judgeguard-alexa-mcp

python3 -m venv .venv
source .venv/bin/activate

pip install -e .
```

### 2. Start the MCP Server & Web Simulator
```bash
python3 server.py
```
The server binds to `http://127.0.0.1:8765` and serves both the Streamable HTTP MCP endpoint and the interactive Web Simulator.

### 3. Run Automated Verification Tests
```bash
python3 -m unittest test_server.py
```
**Test Baseline:**
- **Command:** `python3 -m unittest test_server.py`
- **Result:** `Ran 10 tests in 0.051s - OK` (10 passed, 0 failures, 0 errors)
- **Environment:** Python 3.12.3 on Linux x86_64 (`Linux 6.8.0-101-generic`)

---

## 🔌 MCP Streamable HTTP Protocol (Spec 2025-11-25+)

The server supports standard JSON-RPC 2.0 requests over HTTP POST (`/mcp`) and live Server-Sent Events (SSE) streaming.

### 1. Initialize Handshake
```bash
curl -X POST http://127.0.0.1:8765/mcp \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "id": 1,
    "method": "initialize",
    "params": {
      "protocolVersion": "2025-11-25",
      "clientInfo": {"name": "AlexaPlusHost", "version": "1.0.0"}
    }
  }'
```

### 2. List Available Tools (`tools/list`)
```bash
curl -X POST http://127.0.0.1:8765/mcp \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc": "2.0", "id": 2, "method": "tools/list"}'
```

Available governance tools:
- `judgeguard_verify_action`: Pre-action verification gate for agent actions.
- `judgeguard_notebooklm_rag`: Authoritative RAG query with source citation.
- `judgeguard_bedrock_evaluate`: Multi-model risk analysis via AWS Bedrock.
- `judgeguard_audit_context`: Hallucination and factual consistency check.
- `judgeguard_record_friction`: Structured engineering friction logger.

### 3. Execute Pre-Action Verification (`tools/call`)
```bash
curl -X POST http://127.0.0.1:8765/mcp \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "id": 3,
    "method": "tools/call",
    "params": {
      "name": "judgeguard_verify_action",
      "arguments": {"action": "Unlock front door for delivery driver"}
    }
  }'
```

### 4. Connect to SSE Event Stream
```bash
curl -N http://127.0.0.1:8765/mcp
```

---

## 🌐 Interactive Alexa+ Web Simulator

Navigate to:
```
http://127.0.0.1:8765/
```
Or:
```
http://127.0.0.1:8765/simulator
```

**Simulator Highlights:**
- **Voice / Chat Interface:** Send test Alexa+ commands (e.g. *"Alexa, check energy consumption"*, *"Alexa, unlock the front door"*).
- **Real-Time JudgeGuard HUD:** Visual feedback showing pre-check evaluation, policy compliance, and approval status.
- **NotebookLM RAG Panel:** Inspect citations and authoritative knowledge grounding.
- **Friction Logger:** Record and review developer friction points in real time.

---

## 📑 Hackathon Documentation & Evidence

- **Master Submission Checklist:** [`MASTER_SUBMISSION_CHECKLIST.md`](MASTER_SUBMISSION_CHECKLIST.md)
- **AWS Product Feedback:** [`AWS_PRODUCT_FEEDBACK.md`](AWS_PRODUCT_FEEDBACK.md) — Bedrock integration telemetry, latency benchmarks, and feature requests.
- **Engineering Friction Logs:** [`HACKATHON_FRICTION_LOG.md`](HACKATHON_FRICTION_LOG.md) — 3 detailed technical logs documenting real challenges and solutions (for the 10% judging bonus).
- **Demo Video Script:** [`DEMO_VIDEO_SCRIPT.md`](DEMO_VIDEO_SCRIPT.md) — Timed 2:45 walkthrough script for YouTube/Vimeo public submission.

---

## 📄 License

This project is licensed under the [MIT License](LICENSE).
