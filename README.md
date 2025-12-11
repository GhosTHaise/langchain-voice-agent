
---

# **LangChain Voice Agent**

*A lightweight, real-time, voice-enabled agent powered by LangChain — speak, process, and receive intelligent responses instantly.*

Turn your voice into an AI-powered conversation.
This agent listens, transcribes, processes with LLM reasoning, and responds back using high-quality speech synthesis.

---

## 🚀 **Features**

* 🎤 **Real-time voice input**
* 🧠 **LangChain reasoning engine**
* 🔊 **Natural speech response output**
* ⚡ **Low-latency, streaming pipeline**
* 🧩 **Easily extendable — plug your own tools or prompts**
* 🪶 **Lightweight codebase — no unnecessary dependencies**

---

## 🧱 **Architecture**

```
Your Microphone
      ↓
AssemblyAI (Speech-to-Text)
      ↓
LangChain (LLM reasoning, tools, agents, memory)
      ↓
Cartesia (Text-to-Speech)
      ↓
Speaker Output
```

---

## 📦 **Installation**

```bash
git clone https://github.com/your-username/langchain-voice-agent.git
cd langchain-voice-agent
npm install
```

---

## 🔧 **Environment Variables**

Create a `.env` file in the project root:

```env
# Speech-to-Text (input)
ASSEMBLYAI_API_KEY=your_assemblyai_key_here

# Speech synthesis (output)
CARTESIA_API_KEY=your_cartesia_key_here

# LLM / LangChain (processing)
GOOGLE_API_KEY=your_google_genai_key_here
```

> **Note:**
> • AssemblyAI → transcribes your microphone input
> • Cartesia → speaks the AI’s response
> • Google → powers the LLM (Gemini) in LangChain
> • All three keys are required for full functionality

---

## ▶️ **Run the Voice Agent**

```bash
uv sync --dev
cd ./web
pnpm install && pnpm build
cd ..
uv run src/main.py
```

Then speak into your microphone — the agent will think and talk back.

---

## 🛠️ **Configuration**

You can customize:

* 🎚️ **LLM model**
* 🗣️ **Voice selection**
* 🧠 **LangChain prompt templates**
* 🎛️ **Streaming settings**
* 📡 **Tool integrations**

Open `config.ts` (or your equivalent file) to modify parameters.

---

## 🧩 **Example: Plug in your own tool**

```ts
agent.addTool({
  name: "weather",
  description: "Get weather info",
  func: async (location) => {
    return fetch(`https://api.weather.com/q=${location}`).then(r => r.text());
  }
});
```

---

## 📚 **Tech Stack**

* **LangChain** – agent orchestration
* **Gemini / Google Generative AI** – LLM reasoning
* **AssemblyAI** – speech-to-text
* **Cartesia** – neural TTS (speech synthesis)
* **Node.js / TypeScript** – runtime

---

## 🧪 Development Status

This project is lightweight and designed as a minimal base.
Perfect for:

* Voice assistants
* Customer support bots
* Voice-operated tools
* Hands-free interfaces
* Real-time conversational AI demos

---

## 📜 License

MIT — free to use, modify, and deploy.

---
