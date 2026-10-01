# 🕵️‍♂️ Advanced AI Web Protocols: Reverse Engineering ChatGPT & Claude

A high-level technical deep-dive and reverse-engineering study mapping out the internal production-grade network architectures, streaming protocols, and client-side execution loops behind **OpenAI's ChatGPT** and **Anthropic's Claude**. 

This repository documents how modern enterprise Artificial Intelligence interfaces manage live synchronization, long-context data ingestion, and token generation pipelines using browser Developer Tools (DevTools).

---

## 🛠️ Operational Methodology: How to Intercept the Traces

Although core AI inference occurs remotely inside closed-source cloud infrastructure, web clients rely on persistent, duplex HTTP/3 and HTTP/2 transport bridges to process token generation. To capture these logs in real-time using browser DevTools (Firefox/Chrome):

1. **Environment Setup:** Open the target AI web platform and press `F12` to invoke the developer panel.
2. **Network Filtering:** Navigate to the **Network** tab and activate the **Fetch/XHR** filter to isolate telemetry and system scripts from visual asset buffers.
3. **Capture Execution:** Dispatch a long-context query (e.g., *"Write a 3-paragraph story"*).
4. **Isolate Streams:** Immediately track incoming server connections carrying response payloads while token rendering is active on the screen.

---

## 🔬 1. OpenAI ChatGPT Architecture Analysis

OpenAI implements a highly clean, structured, and developer-friendly REST-adjacent framework that relies heavily on micro-frontends and precise semantic serialization.

### 🧱 Client-Side Call Stack Tracing
When a prompt is initialized, the browser triggers an automated asynchronous execution chain. Rather than loading a monolith script, OpenAI utilizes **code-splitting** to segment logic across independent micro-assets (located under `/unauth-mweb/assets/` paths for mobile/unauthenticated footprints):

* **`conversation-runtime-probe-*.js`:** Orchestrates background environment mapping, health polling, and live channel diagnostics (`u` and `o` runtime parameters).
* **`conversation-followup-dom-controller-*.js`:** Manages immediate DOM mutations, updating the interface frame to print characters and preparing the environment for contextual follow-up query suggestions via localized injection modules (`$n` and `getUpgradeTarget` calls).
* **`a-*.js`:** Serves as the lower-level asynchronous automation engine, handling background micro-tasks and scheduling routines through native `setTimeout` execution wrappers.

### 📡 The `/backend-api` & Event Streaming
* **Target Endpoint:** `POST https://chatgpt.com` (or specialized runtime extensions).
* **The Streaming Mechanism:** ChatGPT leverages native **Server-Sent Events (SSE)** over standard network pipes (`text/event-stream`). Inside the event buffer, data tokens are delivered sequentially inside readable key-value JSON trees:
  ```json
  data: {"message": {"author": {"role": "assistant"}, "content": {"parts": ["O"]}, "status": "in_progress"}}
  data: {"message": {"author": {"role": "assistant"}, "content": {"parts": ["Oi"]}, "status": "in_progress"}}
  data: [DONE]
  ```
* **State Management:** Conversational histories, branches, and regenerated alternative variations are managed as a live node tree directly cached within the client state.

---

## 🧪 2. Anthropic Claude Architecture Analysis

Anthropic's frontend infrastructure for Claude is optimized for enterprise data serialization, enabling heavy payload ingestion (e.g., massive context windows, code bases, and multi-page PDFs) without inducing layout thrashing or browser thread bottlenecks.

### 📡 The `/completion` Pipeline & Event Transport
* **Target Endpoint:** `POST https://claude.ai{org_id}/chat_conversations/{conversation_id}/completion`
* **Transport Driver:** The network connection explicitly binds as an **`eventsource`** over `HTTP/3`, optimizing parallel packet routing to prevent Head-of-Line blocking during dense character streams.

### 👁️ Structural Discoveries from Live Event Traces
Inspecting the raw downstream socket buffer exposes a complex, highly specialized multi-stage state protocol composed of distinct event types:

1. **The Heartbeat (`event: ping`):** The server continuously broadcasts isolated ping blocks containing small JSON bodies: `data: {"type":"ping"}`. This acts as a network keep-alive boundary, preventing firewalls or load balancers from dropping the long-lived streaming connection while the LLM model processes contextual tokens.
2. **The Hidden Reasoner (`type: "thinking"`):** Before rendering user-visible markdown characters, Claude initializes a hidden architectural step inside the network layers. It fires a `content_block_start` mapping out a temporary model layer, streaming internal processing steps before finalizing the logic:
   ```text
   event: content_block_delta
   data: {"type":"content_block_delta","index":0,"delta":{"type":"thinking_summary_delta","summary":{"summary":"Drafting a short three-paragraph story."}}}
   ```
3. **The Content Assembly (`type: "text_delta"`):** Once the thinking boundary reaches a `content_block_stop`, the application swaps active indices and begins pumping rapid token bursts (`"The"`, `" l"`, `"ighthouse ke"`, `"eper's"`) across the network, forcing progressive layout paint cycles on the DOM canvas.

---

## 📊 Structural Infrastructure Comparison

| Architectural Metric | OpenAI ChatGPT | Anthropic Claude |
| :--- | :--- | :--- |
| **Primary Connection Endpoint** | `/backend-api/conversation` | `/api/.../completion` |
| **Network Type Definition** | Standard XHR Fetch | EventSource Stream Connection |
| **Data Payload Transport** | Clean Semantic JSON | High-Density Structured JSON |
| **Streaming Design Pattern** | Character Token SSE Blocks | Multi-Event Type Delta SSE Blocks |
| **Keep-Alive Management** | Passive Socket Duration | Active `event: ping` Heartbeats |
| **Cognitive Trace Rendering** | Direct Output State | Explicit `type: "thinking"` Pipelines |

---

If u inspect Claude AI, you can mark the space XHR or something like this and alternate between the sections of the Ai processement:

<img width="1553" height="672" alt="image" src="https://github.com/user-attachments/assets/2d9af810-907f-4525-956c-91c6c325e173" />

<img width="1278" height="308" alt="Screenshot 2026-10-01 143043" src="https://github.com/user-attachments/assets/c4d35b13-435a-4fc7-8644-98c95d207892" />

<img width="592" height="278" alt="Screenshot 2026-10-01 143435" src="https://github.com/user-attachments/assets/812102af-873a-4fd7-90a6-039a7eeb8aab" />  <img width="585" height="284" alt="Screenshot 2026-10-01 143428" src="https://github.com/user-attachments/assets/222ff9e3-4b13-4344-a0c6-dcbd0651d902" />

<img width="586" height="289" alt="Screenshot 2026-10-01 143423" src="https://github.com/user-attachments/assets/412252ba-bea5-416d-9eb2-795f7ea9ab0e" />  <img width="582" height="265" alt="Screenshot 2026-10-01 143416" src="https://github.com/user-attachments/assets/bb5e4582-83ba-41fd-8f94-9c97d448ab8b" />


---

And, if you do the same on ChatGPT, you can do the same things:

<img width="1365" height="634" alt="Screenshot 2026-10-01 142857" src="https://github.com/user-attachments/assets/d01a486c-7d74-4210-9f6d-d550cc2a9bd7" />

---

Also here's a bonus 

If you do the same on Google Gemini, you can do EXACTLY the same things.

<img width="790" height="452" alt="Screenshot 2026-10-01 141157" src="https://github.com/user-attachments/assets/68bfa3ff-a75b-4b1a-82e7-501b3357b642" />
<img width="1389" height="768" alt="image" src="https://github.com/user-attachments/assets/19931759-39fd-4969-a201-b689ab346e16" />

---

## 📂 Repository Blueprint

* `/chatgpt-analysis/` -> Technical documentation tracking script-splitting dependencies and network initialization stacks.
* `/claude-analysis/` -> Captured live stream traces mapping operational metrics, heartbeats, and raw text assembly models.

---

## 🛑 Security Enforcement & Sanitization Notice
**Critical Safety Protocol:** If you extract raw event blocks or headers from your private DevTools interface to contribute to this archive, **YOU MUST SANITIZE ALL TOKENS**. Ensure strings matching authentication signatures like `Authorization: Bearer ...`, `Cookie: ...`, `__Secure-1PSID`, or specific organizational UUIDs are completely scrubbed. Publishing exposed active session cookies compromises private workspace layers.

---

*Disclaimer: This repository is intended strictly for educational analysis, web systems research, and reverse-engineering comprehension. All endpoints, routing parameters, and internal asset configurations belong exclusively to their respective owners and are subject to production structural updates without prior notice.*
