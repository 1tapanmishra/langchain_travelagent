# ✈️ Travel Agent Chatbot

> A streaming LangChain chatbot that acts as a personal travel agent — answers only travel questions, maintains conversation memory, and auto-summarizes every 5 turns to keep context lean.

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/YOUR_USERNAME/travel-agent-chatbot/blob/main/travel_agent_chatbot.ipynb)
[![Python 3.9+](https://img.shields.io/badge/python-3.9+-blue.svg)](https://www.python.org/)
[![LangChain](https://img.shields.io/badge/LangChain-0.2+-green.svg)](https://python.langchain.com/)
[![Gemini](https://img.shields.io/badge/Gemini-2.0--flash--lite-orange.svg)](https://aistudio.google.com/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

---

## 🎯 What It Does

| Feature | Detail |
|---|---|
| **Travel-only guard** | Refuses all non-travel questions with `I can't help with it` |
| **Streaming output** | Responses print token-by-token as the LLM generates |
| **Manual memory** | Full conversation stored in a plain Python `list` |
| **Auto-summarization** | Every 5 turns, history is compressed to bullet points |
| **Context management** | Summary replaces old messages — keeps prompt size small |

---

## 🏗️ Architecture

```
User Input
    │
    ▼
┌─────────────────────────────────┐
│  Travel Guard (system prompt)   │  ← hard rule: refuse non-travel
└──────────────┬──────────────────┘
               │ travel only
               ▼
┌─────────────────────────────────┐
│  Memory Manager (Python List)   │
│  • HumanMessage / AIMessage     │
│  • Summarizes every 5 turns     │
│  • Replaces history with digest │
└──────────────┬──────────────────┘
               │
               ▼
┌─────────────────────────────────┐
│  LangChain Chain                │
│  ChatPromptTemplate             │
│      │                          │
│  ChatGoogleGenerativeAI         │  ← Gemini 2.0 Flash Lite
│      │                          │
│  .stream() → token by token     │
└─────────────────────────────────┘
```

---

## 💻 Local Installation

```bash
git clone https://github.com/YOUR_USERNAME/travel-agent-chatbot.git
cd travel-agent-chatbot

pip install langchain langchain-google-genai langchain-core google-genai rich

export GOOGLE_API_KEY="your-key-here"

jupyter notebook travel_agent_chatbot.ipynb
```

---

## 🗂️ Notebook Structure

| Cell | Purpose |
|---|---|
| **Cell 1** — Install | Installs all dependencies via `subprocess` (no backslash pip bug) |
| **Cell 2** — Config | Model name, temperature, summary interval, API key via `getpass` |
| **Cell 3** — Prompts | Travel system prompt with hard refusal rule + summarization prompt |
| **Cell 4** — Memory | `chat_history` list, `add_to_memory()`, `summarize_and_compress()` |
| **Cell 5** — Engine | `chat()` with `.stream()` loop, turn counter, summary trigger |
| **Cell 6** — Demo | Interactive chatbot loop with `memory` / `reset` / `history` commands |
| **Cell 7** — Tests | 7 scripted messages verifying streaming, guard, and summarization |

---

## 🧠 How the Three Core Features Work

### 1. Streaming

LangChain's `.stream()` method yields response chunks as the LLM generates them. Each chunk is a tiny string fragment (1–5 tokens) that gets printed immediately:

```python
for chunk in chat_chain.stream({"messages": chat_history}):
    print(chunk.content, end="", flush=True)   # live token output
    full_response += chunk.content
```

This gives the familiar "typing" effect without waiting for the full response.

---

### 2. Manual Memory (Python List)

Every message is stored as a LangChain message object in a plain list:

```python
chat_history: list = []   # HumanMessage and AIMessage objects

# Adding messages
chat_history.append(HumanMessage(content="Best beaches in Thailand?"))
chat_history.append(AIMessage(content="Thailand has stunning beaches..."))
```

The full list is injected into the prompt via `MessagesPlaceholder` on every call, giving the LLM full conversation context.

---

### 3. Auto-Summarization Every 5 Turns

After every 5 complete user+AI exchanges, the history is compressed:

```
Turn 1  HumanMessage + AIMessage  ┐
Turn 2  HumanMessage + AIMessage  │  → sent to summary LLM
Turn 3  HumanMessage + AIMessage  │  → returns 3-5 bullet points
Turn 4  HumanMessage + AIMessage  │  → replaces all messages
Turn 5  HumanMessage + AIMessage  ┘     with one SystemMessage
                                         containing the digest
Turn 6  starts fresh with only the summary in memory
```

This keeps the context window small for long sessions while preserving the gist of the conversation.

---

## 🎮 Chat Commands

| Command | Action |
|---|---|
| Any travel question | Streaming response from the travel agent |
| Any non-travel question | `I can't help with it` |
| `memory` | Show current memory state and message count |
| `history` | Print full conversation history |
| `reset` | Clear all memory and start fresh |
| `quit` / `exit` | Stop the chatbot |

---

## ✅ Travel Topics Covered

- ✈️ Flights, airlines, routes, and booking tips
- 🏨 Hotels, resorts, hostels, and Airbnb guidance
- 🗺️ Destination recommendations and hidden gems
- 📋 Visa requirements and travel documents
- 🗓️ Itinerary planning and day-by-day schedules
- 💰 Budget planning and cost estimates
- 🌏 Local culture, food, transport, and safety
- 🧳 Packing lists and travel gear advice
- 🛡️ Travel insurance and health precautions
- 💱 Currency exchange and money tips

---

## 🧪 Automated Test Results

Cell 7 runs 7 scripted messages and verifies:

| Turn | Input | Expected Output |
|---|---|---|
| 1 | Top destinations in SE Asia? | Travel answer ✅ |
| 2 | Japan on $2000 for 10 days? | Travel answer ✅ |
| 3 | Schengen visa requirements? | Travel answer ✅ |
| 4 | 5-day Bali itinerary? | Travel answer ✅ |
| 5 | Best time for Patagonia? | Travel answer + **summarization triggered** ✅ |
| 6 | Write a Python sort function? | `I can't help with it` ✅ |
| 7 | Direct flights NYC to Tokyo? | Travel answer resumes ✅ |

---

## ⚙️ Configuration

```python
# In Cell 2
MODEL_NAME    = "gemini-2.0-flash-lite"   # cheapest, free-tier friendly
TEMPERATURE   = 0.7                        # creativity level (0.0–1.0)
SUMMARY_EVERY = 5                          # summarize every N turns
```

### Model options (from your API key)

| Model | Speed | Quality | Free Tier |
|---|---|---|---|
| `gemini-2.0-flash-lite` | ⚡ Fastest | Good | ✅ Most generous |
| `gemini-2.0-flash` | Fast | Better | ✅ Good |
| `gemini-2.5-flash` | Medium | Best | ⚠️ Higher quota cost |

---

## 🔧 Troubleshooting

| Error | Fix |
|---|---|
| `404 NOT_FOUND` | Model name wrong — run `genai.list_models()` to get valid names |
| `429 RESOURCE_EXHAUSTED` with `limit: 0` | Daily quota exhausted — wait for reset or switch to `gemini-2.0-flash-lite` |
| `429 RESOURCE_EXHAUSTED` with retry time | Per-minute limit — the retry logic handles this automatically |
| `CalledProcessError` on pip install | Don't use backslash line continuations in `!pip` — use the `subprocess` pattern in Cell 1 |
| Non-travel question not refused | Check that the system prompt is loading correctly — run `show_memory_status()` |

---

## 📄 License

MIT — free to use, modify, and distribute.
