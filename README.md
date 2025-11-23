# 🧠 Blinksbuy MCP Adapter (Canonical JSON v1.1)

### AI Automation Platform – Model Context Protocol Adapter  
*Version: 1.1 · Deployment ready for Railway*

---

## 📌 Overview

The **MCP Adapter** is the entry point for all external channels (web, mobile, voice, chat, WhatsApp, ElevenLabs, etc.) into the **Model Context Protocol (MCP) Platform**.

It converts **raw client messages** into a **Canonical JSON Envelope (v1.1)** that the MCP Orchestrator understands.  
It also converts the MCP’s canonical response back into a channel-agnostic JSON format that upstream systems can use (TTS, chat clients, UIs, etc.)

This adapter ensures that **all channels speak one unified protocol**, enabling consistent:

- Observability (trace/span/message IDs)  
- LLM prompt context  
- Session continuity  
- Microservice-safe routing  
- Multi-tenant behavior  
- Future multimodal inputs (audio, images, events, etc.)  

---

## 🚀 Key Features

- **LLM-ready** – carries optional LLM hints (model, temperature, etc.)
- **Microservice-safe** – standardized error envelope on downstream failures.
- **Adapter-agnostic** – works for Web, Voice, Mobile, WhatsApp, etc.
- **Traceable** – `trace_id`, `span_id`, `message_id` for every message.
- **Versionable** – Canonical Envelope v1.1, backward-compatible with v1.0.
- **Future-proof for multimodal** – supports `type: text | audio | image | event`.

---

## ⚙️ Environment Variables

| Variable   | Description                                      | Example                                                     |
|-----------|--------------------------------------------------|-------------------------------------------------------------|
| `MCP_URL` | URL of the MCP Orchestrator `/orchestrate`      | `https://mcp-service.up.railway.app/orchestrate`          |
| `API_KEY` or `API_KEYS` | One or more allowed API keys       | `abc123,xyz789`                                            |

---

## 🔐 Authentication

Every canonical request **must** include:

```http
X-API-Key: <your_key>
```

Requests without valid keys will return:

- `401 Unauthorized` (no key)
- `403 Forbidden` (invalid key)

---

## 📡 API Endpoints

### `GET /health`

Basic healthcheck.

**Response:**

```json
{
  "status": "ok",
  "service": "mcp_adapter_python",
  "mcp_url": "https://mcp-service.up.railway.app/orchestrate"
}
```

---

### `POST /canonical/message`  (Primary endpoint)

Channel-agnostic canonical message gateway.

**Headers:**

```http
X-API-Key: <your_key>
Content-Type: application/json
```

**Body (Canonical Request Envelope v1.1 – text example):**

```json
{
  "version": "1.1",
  "timestamp": "2025-11-21T21:28:56.146Z",
  "context": {
    "channel": "web",
    "device": "browser",
    "locale": "en-US",
    "tenant": "blinksbuy",
    "client_app": "elevenlabs",
    "llm": {
      "model_hint": "gpt-4.1-mini",
      "temperature": 0.2
    }
  },
  "session": {
    "session_id": "user-123:web",
    "conversation_id": "conv-001",
    "user_id": "user-123",
    "turn": 3
  },
  "request": {
    "type": "text",
    "text": "Can you read me the menu?",
    "intent_override": null,
    "metadata": {
      "raw_transcript": "can you read me the menu",
      "confidence": 0.97
    }
  },
  "observability": {
    "trace_id": "trace-abc-123",
    "span_id": "span-inbound-1",
    "message_id": "msg-0001"
  }
}
```

---

### Multimodal Example (Audio)

```json
{
  "version": "1.1",
  "timestamp": "2025-11-21T21:30:00Z",
  "context": {
    "channel": "voice",
    "device": "iphone",
    "locale": "en-US",
    "tenant": "blinksbuy",
    "client_app": "elevenlabs"
  },
  "session": {
    "session_id": "voice-user-44",
    "conversation_id": "conv-331",
    "user_id": "voice-user-44",
    "turn": 12
  },
  "request": {
    "type": "audio",
    "audio_url": "https://cdn.you/audio.wav",
    "transcript": "I would like to order garlic chicken"
  },
  "observability": {
    "trace_id": "trace-22aa1bb3",
    "span_id": "span-991",
    "message_id": "msg-0002"
  }
}
```

---

## Canonical Response (Success)

```json
{
  "version": "1.1",
  "timestamp": "2025-11-21T21:28:56.146Z",
  "context": {
    "channel": "web",
    "device": "browser",
    "locale": "en-US",
    "tenant": "blinksbuy",
    "client_app": "elevenlabs"
  },
  "session": {
    "session_id": "user-123:web",
    "conversation_id": "conv-001",
    "user_id": "user-123",
    "turn": 4,
    "route": "food_ordering.menu"
  },
  "response": {
    "status": "success",
    "code": 200,
    "type": "text",
    "text": "Here is the menu from Uno Bistro:\n\n1. Garlic Chicken – 500\n2. Sizzling Pata – 650\n3. Sisig – 400",
    "metadata": {
      "source": "mcp_orchestrator",
      "duration_ms": 16250.315
    }
  },
  "error": null,
  "observability": {
    "trace_id": "trace-abc-123",
    "span_id": "span-outbound-1",
    "message_id": "msg-0003"
  }
}
```

---

## Canonical Response (Error)

```json
{
  "version": "1.1",
  "timestamp": "2025-11-21T21:28:56.146Z",
  "context": {
    "channel": "web",
    "device": "browser",
    "locale": "en-US",
    "tenant": "blinksbuy",
    "client_app": "elevenlabs"
  },
  "session": {
    "session_id": "user-123:web",
    "conversation_id": "conv-001",
    "user_id": "user-123",
    "turn": 4,
    "route": null
  },
  "response": {
    "status": "error",
    "code": 502,
    "type": "text",
    "text": null,
    "metadata": {
      "source": "mcp_adapter",
      "duration_ms": 112.529
    }
  },
  "error": {
    "type": "MCP_ORCHESTRATOR_ERROR",
    "code": 502,
    "message": "MCP orchestrator error: 503 Service Unavailable",
    "retryable": false,
    "details": {
      "mcp_url": "https://your-mcp-service.railway.app/orchestrate"
    }
  },
  "observability": {
    "trace_id": "trace-abc-123",
    "span_id": "span-error-1",
    "message_id": "msg-err-001"
  }
}
```

---

## 🧪 Testing with Postman

1. Set headers:
   - `X-API-Key: <your_key>`
   - `Content-Type: application/json`
2. POST to:
   - `https://<your-railway-url>/canonical/message`
3. Paste a canonical request JSON body and send.

---

## 🚉 Railway Deployment

This project is **Railway-ready**:

- `Procfile` is provided:
  ```bash
  web: uvicorn app:app --host 0.0.0.0 --port ${PORT:-8000}
  ```
- Railway will:
  - Install dependencies from `requirements.txt`
  - Detect the Procfile
  - Expose the service on the assigned `$PORT`

> Set your `MCP_URL` and `API_KEY` / `API_KEYS` in **Railway → Variables**.

---

## 🏗 Project Structure

```text
blinksbuy_mcp_adapter/
├── app.py           # FastAPI app with Canonical v1.1 adapter logic
├── README.md        # This documentation
├── requirements.txt # Python dependencies
├── Procfile         # Railway process definition
└── .env.example     # Example env vars (for local dev)
```

---

## ✅ Summary

The Blinksbuy MCP Adapter is:

- Channel-agnostic
- Multi-tenant & multi-locale
- LLM-aware and multimodal-ready
- Traceable end-to-end
- Designed for modern AI automation platforms
- Ready to deploy on **Railway** out of the box

Enjoy building with it 🚀
