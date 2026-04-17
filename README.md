# 🎙️ n8n-Voice-Enabled-AI-Assistant

<div align="center">

![n8n](https://img.shields.io/badge/n8n-Workflow-orange?style=for-the-badge&logo=n8n)
![ElevenLabs](https://img.shields.io/badge/ElevenLabs-TTS-black?style=for-the-badge)
![Gemini](https://img.shields.io/badge/Google-Gemini-blue?style=for-the-badge&logo=google)
![Pinecone](https://img.shields.io/badge/Pinecone-VectorDB-green?style=for-the-badge)
![OpenAI](https://img.shields.io/badge/OpenAI-Embeddings-412991?style=for-the-badge&logo=openai)

**A production-ready Voice AI Assistant that listens, thinks, and speaks — powered by n8n automation, ElevenLabs ultra-realistic voice synthesis, Google Gemini LLM, and Pinecone RAG.**

</div>

---

## 📌 What This Does

This workflow turns your documents into a **talking AI assistant**. Here's the complete user journey:

```
User speaks a question
        ↓
Speech-to-text (your frontend)
        ↓
POST query → n8n Webhook
        ↓
Gemini AI searches your knowledge base (Pinecone RAG)
        ↓
AI generates a text answer
        ↓
ElevenLabs converts the answer to ultra-realistic voice 🔊
        ↓
Audio plays back to the user
```

---

## 🧠 Full System Architecture

```
╔══════════════════════════════════════════════════════════════════╗
║                  PART 1 — DOCUMENT INGESTION                    ║
╠══════════════════════════════════════════════════════════════════╣
║                                                                  ║
║  [Manual Trigger]                                                ║
║       │                                                          ║
║       ▼                                                          ║
║  [Search Google Drive Folder]                                    ║
║       │                                                          ║
║       ▼                                                          ║
║  [Download Files]                                                ║
║       │                                                          ║
║       ▼                                                          ║
║  [Loop Over Items]  ◄─────────────────────────────────┐         ║
║       │                                               │         ║
║       ▼                                               │         ║
║  [Pinecone Vector Store] ─────────────────────────────┘         ║
║    ▲              ▲                                              ║
║  [OpenAI      [Default Data Loader]                              ║
║  Embeddings]       ▲                                             ║
║              [Recursive Character                                ║
║               Text Splitter]                                     ║
║                                                                  ║
╠══════════════════════════════════════════════════════════════════╣
║                  PART 2 — VOICE QUERY PIPELINE                  ║
╠══════════════════════════════════════════════════════════════════╣
║                                                                  ║
║  [POST /voiceaiassistant]                                        ║
║       │  { "query": "user's transcribed speech" }               ║
║       ▼                                                          ║
║  [AI Agent — Gemini]                                             ║
║    ├── [Gemini LLM]                                              ║
║    └── [Vector Store Tool]                                       ║
║              ├── [Pinecone Retrieve]                             ║
║              │       ▲                                           ║
║              │   [OpenAI Embeddings]                             ║
║              └── [Gemini Chat Model]                             ║
║       │                                                          ║
║       ▼                                                          ║
║  [Respond to Webhook]                                            ║
║       │  { "output": "AI text answer" }                          ║
║       ▼                                                          ║
║  ╔══════════════════════════════╗                                ║
║  ║   ELEVENLABS TTS LAYER 🔊   ║                                ║
║  ║  Text Answer → Audio Stream  ║                                ║
║  ╚══════════════════════════════╝                                ║
║                                                                  ║
╚══════════════════════════════════════════════════════════════════╝
```

---

## 🔧 Tech Stack

| Layer | Tool | Purpose |
|---|---|---|
| 🔁 Automation | [n8n](https://n8n.io) | Workflow orchestration |
| 🧠 AI Agent | Google Gemini (PaLM) | Main reasoning & response generation |
| 🧠 RAG Chain LLM | Google Gemini Chat Model | Context-aware answer generation |
| 📦 Vector Database | [Pinecone](https://pinecone.io) | Semantic document storage & retrieval |
| 🔢 Embeddings | OpenAI (`text-embedding-ada-002`) | Convert text to vectors |
| 📂 Knowledge Source | Google Drive | Document storage & auto-sync |
| 🔊 Voice Synthesis | [ElevenLabs](https://elevenlabs.io) | Ultra-realistic AI voice output |
| 🌐 API Layer | n8n Webhook (POST) | Receives queries from any client |

---

## 🚀 Key Features

| Feature | Description |
|---|---|
| 🎤 **Voice-Ready Responses** | All AI answers are formatted for natural spoken delivery via ElevenLabs |
| 🔍 **RAG-Powered Knowledge** | Answers are grounded in YOUR uploaded documents, not just general AI knowledge |
| 🔄 **Batch Ingestion** | Safely ingests large document sets with loop-based batch processing |
| ✂️ **Smart Chunking** | Recursive Character Text Splitter ensures optimal chunk sizes for retrieval |
| 🤖 **Dual Gemini Models** | Separate Gemini instances for the agent layer and the RAG chain layer |
| 📡 **REST API Endpoint** | Any app, browser, or voice interface can trigger the assistant via HTTP POST |
| 🔊 **ElevenLabs TTS** | Converts AI text responses into lifelike audio with any chosen voice |

---

## 📦 Prerequisites

- [ ] **n8n** instance (self-hosted or cloud)
- [ ] **Google Drive** OAuth2 credentials
- [ ] **Pinecone** account + index named `voice`
- [ ] **OpenAI** API key (embeddings only)
- [ ] **Google Gemini (PaLM)** API key
- [ ] **ElevenLabs** API key + a Voice ID of your choice

---

## ⚙️ Step-by-Step Setup

### Step 1 — Import the Workflow

In your n8n dashboard:
```
Workflows → ⋮ → Import from file
→ Upload: Voice-Enabled_Assistant__Elevenlab_.json
```

### Step 2 — Add Credentials

| Credential | Where to get it |
|---|---|
| Google Drive OAuth2 | [Google Cloud Console](https://console.cloud.google.com) |
| Pinecone API | [app.pinecone.io](https://app.pinecone.io) |
| OpenAI API | [platform.openai.com](https://platform.openai.com) |
| Google Gemini API | [Google AI Studio](https://aistudio.google.com) |
| ElevenLabs API | [elevenlabs.io/app/settings/api-keys](https://elevenlabs.io/app/settings/api-keys) |

### Step 3 — Create Pinecone Index

```
Index name : voice
Dimensions : 1536
Metric     : cosine
Cloud      : AWS (or your preferred)
```

### Step 4 — Run Document Ingestion

1. Place your PDF/DOCX/TXT files in a **Google Drive folder**
2. Update the **"Search files and folders"** node with your folder ID
3. Click **"Execute workflow"** — files will be chunked, embedded, and stored in Pinecone
4. Re-run whenever you add new documents

### Step 5 — Connect ElevenLabs for Voice Output

After n8n returns the AI text answer, pass it to ElevenLabs:

#### Option A — n8n HTTP Request Node (add after "Respond to Webhook")
```json
POST https://api.elevenlabs.io/v1/text-to-speech/{voice_id}

Headers:
  xi-api-key: YOUR_ELEVENLABS_API_KEY
  Content-Type: application/json

Body:
{
  "text": "{{ $json.output }}",
  "model_id": "eleven_multilingual_v2",
  "voice_settings": {
    "stability": 0.5,
    "similarity_boost": 0.8,
    "style": 0.2,
    "use_speaker_boost": true
  }
}
```

#### Option B — JavaScript / Frontend
```javascript
// 1. Send user query to n8n
const n8nResponse = await fetch('https://YOUR_N8N_URL/webhook/voiceaiassistant', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ query: userTranscribedSpeech })
});
const { output } = await n8nResponse.json();

// 2. Send AI answer to ElevenLabs
const voiceResponse = await fetch(
  `https://api.elevenlabs.io/v1/text-to-speech/${VOICE_ID}/stream`,
  {
    method: 'POST',
    headers: {
      'xi-api-key': ELEVENLABS_API_KEY,
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({
      text: output,
      model_id: 'eleven_multilingual_v2',
      voice_settings: {
        stability: 0.5,
        similarity_boost: 0.8
      }
    })
  }
);

// 3. Stream and play the audio
const audioBlob = await voiceResponse.blob();
const audioUrl = URL.createObjectURL(audioBlob);
new Audio(audioUrl).play();
```

#### Option C — Python Backend
```python
import requests

def ask_voice_assistant(query: str) -> bytes:
    # Step 1: Get AI answer from n8n
    n8n_res = requests.post(
        "https://YOUR_N8N_URL/webhook/voiceaiassistant",
        json={"query": query}
    )
    ai_text = n8n_res.json()["output"]

    # Step 2: Convert to speech via ElevenLabs
    voice_res = requests.post(
        f"https://api.elevenlabs.io/v1/text-to-speech/{VOICE_ID}",
        headers={
            "xi-api-key": ELEVENLABS_API_KEY,
            "Content-Type": "application/json"
        },
        json={
            "text": ai_text,
            "model_id": "eleven_multilingual_v2",
            "voice_settings": {
                "stability": 0.5,
                "similarity_boost": 0.8
            }
        }
    )
    return voice_res.content  # MP3 audio bytes
```

---

## 🌐 API Reference

### POST `/webhook/voiceaiassistant`

**Request:**
```json
{
  "query": "What are the key features described in the uploaded document?"
}
```

**Response:**
```json
{
  "output": "Based on your documents, the key features are..."
}
```

> 💡 Pass `output` directly to ElevenLabs TTS for voice playback.

---

## 🎤 Recommended ElevenLabs Voices

| Voice | Style | Best For |
|---|---|---|
| `Rachel` | Calm, professional | Business assistant |
| `Josh` | Deep, authoritative | Technical support |
| `Bella` | Friendly, warm | Customer service |
| `Antoni` | Well-rounded | General use |
| `Elli` | Upbeat, young | Consumer apps |

> Browse all voices at [elevenlabs.io/voice-library](https://elevenlabs.io/voice-library)

---

## 🔄 Workflow Node Reference

### Ingestion Nodes
| Node | Type | Role |
|---|---|---|
| Manual Trigger | Trigger | Starts ingestion on demand |
| Search Files and Folders | Google Drive | Lists all files in target folder |
| Download File | Google Drive | Downloads file binary |
| Loop Over Items | Utility | Batches files to prevent rate limits |
| Recursive Character Text Splitter | LangChain | Splits docs into optimal chunks |
| Default Data Loader | LangChain | Parses binary → LangChain documents |
| Embeddings OpenAI | LangChain | Generates 1536-dim vectors |
| Pinecone Vector Store (insert) | LangChain | Stores vectors in `voice` index |

### Query Nodes
| Node | Type | Role |
|---|---|---|
| Webhook | Trigger | POST endpoint for incoming queries |
| AI Agent | LangChain | Orchestrates Gemini + tools |
| Gemini | LangChain LLM | Powers the agent's reasoning |
| Answer Questions with Vector Store | LangChain Tool | Performs semantic search + QA |
| Pinecone Vector Store (retrieve) | LangChain | Fetches top-k relevant chunks |
| Embeddings OpenAI1 | LangChain | Embeds query for retrieval |
| Gemini Chat Model | LangChain LLM | Generates answer from context |
| Respond to Webhook | Output | Returns JSON response to caller |

---

## 📁 Repository Structure

```
n8n-Voice-Enabled-AI-Assistant/
├── Voice-Enabled_Assistant__Elevenlab_.json   # n8n workflow (import this)
└── README.md                                  # This file
```

---

## 🛡️ Security Best Practices

- 🔐 Store all API keys in **n8n's encrypted credential store** — never hardcode them
- 🛡️ Add **Header Auth** to the webhook for production:
  `Settings → Webhook → Add Authentication → Header Auth`
- 🌐 Use **HTTPS** for all webhook URLs
- 🚫 Never expose your ElevenLabs API key on the frontend — proxy through a backend

---

## 🔧 Troubleshooting

| Issue | Fix |
|---|---|
| Webhook returns empty response | Check that AI Agent is connected to Respond to Webhook node |
| Pinecone returns no results | Re-run ingestion pipeline; verify index name is `voice` |
| ElevenLabs returns 401 | Verify API key is correct and not expired |
| Gemini returns empty text | Check PaLM API key and quota limits |
| Files not ingested | Confirm Google Drive folder ID matches in both nodes |

---

## 📄 License

MIT License — free to use, modify, and distribute.

---

## 🙌 Contributing

Pull requests are welcome! For major changes, please open an issue first to discuss what you'd like to change.

---

<div align="center">

*Built with ❤️ using n8n · ElevenLabs · Google Gemini · Pinecone · OpenAI*

</div>
