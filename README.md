# 🎧 Story2Audio

**Story2Audio** is a multi-modal AI storytelling system that takes a short prompt and generates a complete narrated story with expressive emotional audio. It uses a custom gRPC backend powered by LLaMA3 (via Ollama), XTTS (voice cloning), and Streamlit or REST clients to provide a responsive and flexible frontend experience.

---

## 🧠 Architecture

```
User (Streamlit or REST)
        │
        ▼
┌─────────────────────────────────────────┐
│         gRPC Server         │
│  ┌─────────────────────────┐  │
│  │ - Story prompt + meta │  │
│  │ - LLaMA3/Mistral LLM   │  │
│  │ - XTTS for voice TTS  │  │
│  └─────────────────────────┘  │
└─────────────────────────────────────┘
        │
        ▼
🎧 Audio Output + 📜 Story Text
```

---

## ⚙️ Set Up Environment

```bash
# 1. Create a virtual environment
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# 2. Install dependencies
pip install -r requirements.txt
```

> ✅ Make sure you have:
>
> * A CUDA-compatible GPU (for XTTS)
> * `ollama` installed and running locally
> * XTTS model files inside `xtts_model/`

---

## 🚀 Running the Service

### Start the gRPC server

```bash
python server.py
```

### Start the Streamlit frontend

```bash
streamlit run streamlit_ms.py
```

### (Optional) Start the REST proxy server

```bash
python rest_server.py
```

This will start a REST server at `http://localhost:8000/generate-story/`.

---

## 📱 gRPC Interface

### Proto file: `story_service.proto`

```proto
syntax = "proto3";

service StoryService {
  rpc GenerateStory (StoryRequest) returns (StoryResponse);
}

message StoryRequest {
  string prompt = 1;
  string emotion = 2;
  float speed = 3;
  string language = 4;
  string speaker_audio = 5;
  bool include_narration = 6;
}

message StoryResponse {
  bytes audio = 1;
  string text = 2;
  string message = 3;
}
```

---

## ✨ Features

* 📜 Natural story generation via LLaMA 3 / Mistral
* 🗣️ Narration-only or narration + dialogue (female character)
* 🎭 Emotion detection and synthesis (happy, sad, angry, etc.)
* 🌐 Multi-language support (en, es, fr, de, hi, it, ru)
* 🔊 Clone any speaker voice by uploading a `.wav` file (≥15s)
* 🧠 Real-time processing using gRPC with concurrent tasks

---

## 🗣️ How to Add Custom Voice

In the Streamlit interface:

1. Upload a `.wav` file (min 15 seconds)
2. OR record your voice using the microphone recorder
3. Provide a speaker name (must be unique)
4. Your voice will appear in the dropdown for story generation

All voices are saved under the `voices/` folder and indexed in `speakers.json`.

---

## 🌐 REST API Usage

### Endpoint:

```
POST http://localhost:8000/generate-story/
```

### JSON Payload:

```json
{
  "prompt": "[PARA_LEVEL:1–3] A young girl finds a lost puppy in the rain.",
  "emotion": "happy",
  "speed": 1.0,
  "language": "en",
  "include_narration": true,
  "speaker_audio_base64": "Uk1GR..."
}
```

### Response:

```json
{
  "text": "Generated story text...",
  "message": "success",
  "audio_file": "response_audio.wav"
}
```

---

## 🧪 Test Case Format & Automation

Sample test case format (`test_cases.json`):

```json
[
  {
    "prompt": "[PARA_LEVEL:1–3] A young girl finds a lost puppy in the rain.",
    "emotion": "happy",
    "speed": 1.0,
    "language": "en",
    "include_narration": true,
    "speaker_audio_base64": "Uk1GR..."
  }
]
```

You can use this with `rest_server.py` to send batches of test prompts to the gRPC server via REST.

---

> ℹ️ For best results, ensure GPU is enabled, and XTTS + LLaMA models are preloaded.
