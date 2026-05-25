# n8n Healthcare LLM Workflow – HL7/FHIR with Guardrails & MCP

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![n8n](https://img.shields.io/badge/n8n-%23EA4B71.svg?style=flat&logo=n8n&logoColor=white)](https://n8n.io)
[![OpenAI](https://img.shields.io/badge/OpenAI-LLM-412991?logo=openai)](https://openai.com)
[![HL7](https://img.shields.io/badge/HL7-v2.x-005A9C)](https://www.hl7.org)
[![FHIR](https://img.shields.io/badge/FHIR-R4-7A1FA5)](https://hl7.org/fhir)

**A ready-to-import n8n automation workflow that safely ingests HL7 v2.x and FHIR R4 data, enforces strict LLM guardrails, and integrates with Model Context Protocol (MCP) for clinically responsible AI responses.**

> 🏥 **Purpose**: Enable secure, auditable LLM-assisted decision support without ever allowing the model to diagnose, prescribe, or guarantee outcomes.

---


---

## ✨ Features

| Feature | Description |
|---------|-------------|
| **Auto‑detect HL7 / FHIR** | Accepts raw HL7 pipe‑separated strings or FHIR JSON (resource/bundle) |
| **Smart Extraction** | Pulls patient ID, condition/problem, and clinical context from both formats |
| **MCP Integration** | Calls your Model Context Protocol server to inject evidence‑based guidelines |
| **LLM with Hard Guardrails** | System prompt enforces safety rules (no diagnoses, no prescriptions, always recommend physician) |
| **Post‑processing Guardrail** | Regex‑based filter blocks forbidden patterns (e.g., “guarantee”, “prescribe”, “100% cure”) |
| **Webhook Ready** | Easy to call from any EMR, API gateway, or external system |
| **Fully Customizable** | Modify the Code nodes, swap LLM provider, or add your own clinical rules |

---

## 🧱 Architecture (how it works)

```mermaid
graph LR
    A[HL7 / FHIR Input] --> B[Webhook Trigger]
    B --> C{Parser Node<br/>Auto‑detect format}
    C -->|Extracts patient + condition| D[MCP Node<br/>Fetch clinical context]
    D --> E[LLM with Guardrail Prompt<br/>OpenAI GPT-4]
    E --> F[Output Guardrail Check<br/>Regex filter]
    F --> G[Safe Response]
    G --> H[Webhook Reply]


  ##   Installation & Setup
1. Import the workflow into n8n
Download workflow.json from this repository.

In n8n, go to Workflows → Add Workflow → Import from File.

Upload workflow.json.

2. Configure credentials
Open the LLM with Guardrail Prompt node.

Under Credentials, select or create your OpenAI API credential.

(Optional) If you use a different LLM provider, replace the node with an equivalent HTTP Request node.


 Configure the MCP endpoint
Open the MCP - Model Context Protocol node.

Replace the url with your actual MCP server endpoint, e.g.
https://your-mcp.internal/clinical-context

Ensure the node sends the required payload (patientId, clinicalContext).
The workflow already injects {{ $json.patientId }} and {{ $json.clinicalContext }}.

Example 1 – HL7 v2.x message
 
curl -X POST https://your-n8n.com/webhook/health-llm-guardrail \
  -H "Content-Type: text/plain" \
  -d 'MSH|^~\&|SendingApp|SendingFac|||202501231200||ADT^A01|MSGID|P|2.5
PID|||P12345||Doe^John||19800101|M
OBX|1|TX|CONDITION||Diabetes Type 2 with neuropathy'


Example 2 – FHIR Condition resource

curl -X POST https://your-n8n.com/webhook/health-llm-guardrail \
  -H "Content-Type: application/json" \
  -d '{
    "resourceType": "Condition",
    "id": "cond-567",
    "subject": {"reference": "Patient/67890"},
    "code": {"text": "Uncontrolled hypertension, stage 2"}
  }'


  Example Response (guardrail‑compliant)

{
  "originalLlmResponse": "Based on the provided information about uncontrolled hypertension, I recommend consulting a licensed physician for a complete treatment plan...",
  "guardrailPassed": true,
  "finalResponse": "Based on the provided information about uncontrolled hypertension, I recommend consulting a licensed physician for a complete treatment plan...",
  "violationsDetected": []
}

Guardrails – How We Keep the LLM Safe
Layer	Mechanism
1. System prompt	Explicitly forbids diagnoses, treatment plans, guarantees, and prescriptions. Always recommends physician consultation.
2. Output regex filter	Scans for patterns like /guarantee/i, /prescribe/i, /100% cure/i. If any match, the response is replaced with a safety message.
3. MCP context injection	Guides the LLM with evidence‑based constraints from your own clinical server.
You can extend the regex patterns in the Output Guardrail Check node (Code node) by editing the forbiddenPatterns array.