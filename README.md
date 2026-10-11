# 🍳 Personal Chef Assistant

An AI cooking agent built with **LangChain**. Tell it the leftover ingredients you have in your
kitchen, and it searches the web for real recipes that use them, remembers the conversation
across runs, and answers any follow-up question.

It can also take a **photo of your fridge** (no typing needed) or an **audio question**
(speak instead of type) — and every mode shares the same memory.

> Built with nothing but the techniques from notebooks **1.1 → 1.4** of the
> *LangChain for AI Foundations* course: chat models & agents, tools / web search,
> memory, and multimodal messages.

---

## ✨ Features

| Feature | What it does |
|---|---|
| 🥕 **Leftover ingredients → recipes** | You list what is in your kitchen; the agent uses a web-search tool (Tavily) to find real recipes that use those ingredients. |
| 🧠 **Memory of previous runs** | The agent keeps the conversation in memory, so follow-up questions like *"give me the steps for the first one"* or *"what did I have again?"* work. |
| 📷 **Fridge / ingredient photo** | Upload a picture of your fridge or ingredients; the agent reads it, extracts a structured ingredient list, tells you what staples are missing, and finds recipes — **no text input needed**. |
| 🎙️ **Audio questions** | Record a question with your microphone; the agent listens to the audio and answers it (with memory). |
| 📚 **Unified chat entry point** | One `personal_chef(...)` function dispatches text, photo or audio, all on the same conversation thread. |

---

## 🗂️ Repository structure

```
personal-chef-assistant/
├── personal_chef_assistant.ipynb   # the notebook (run this)
├── requirements.txt                # dependencies
├── .env.example                    # template for your API keys
├── LICENSE                         # MIT
└── README.md
```

---

## 🚀 Setup

### 1. Clone the repository

```bash
git clone https://github.com/<your-username>/personal-chef-assistant.git
cd personal-chef-assistant
```

### 2. Create a virtual environment

```bash
python -m venv .venv

# Windows
.venv\Scripts\activate

# macOS / Linux
source .venv/bin/activate
```

### 3. Install the dependencies

```bash
pip install -r requirements.txt
```

### 4. Create your `.env` file from the example

The notebook reads its API keys from a `.env` file (that file is **never** committed).

```bash
# macOS / Linux
cp .env.example .env

# Windows
copy .env.example .env
```

Now open `.env` and replace the placeholders with your own keys:

```dotenv
GOOGLE_API_KEY="your_google_api_key_here"   # https://aistudio.google.com/apikey
TAVILY_API_KEY="your_tavily_api_key_here"   # https://app.tavily.com
```

| Variable | Why it is needed |
|---|---|
| `GOOGLE_API_KEY` | Calls the Gemini model (chat, vision for the photo feature, audio for the voice feature). |
| `TAVILY_API_KEY` | Powers the `search_recipes` web-search tool. |

No key is bundled with this repo, and `.env` is listed in `.gitignore` so it can never be
pushed by accident.

### 5. Launch Jupyter and open the notebook

```bash
jupyter lab
```

Then open `personal_chef_assistant.ipynb` and run the cells **top to bottom**.

---

## 🧑‍🍳 How to use it

### Text (with memory)

```python
ask_chef("I have chicken thighs, rice, half an onion and two tomatoes. What can I cook?")
```

Later — still the same memory thread:

```python
ask_chef("Great, give me the full step-by-step instructions for the first one.")
ask_chef("Which ingredients did I say I had at the very beginning?")
```

### Photo of your fridge or ingredients (no text needed)

Run the upload-widget cell, pick a `.jpg` / `.jpeg` / `.png` picture, then:

```python
fridge_workflow(uploader)
```

The notebook prints the detected ingredients and missing staples, then searches the web for
recipes using them. The photo is also sent to the chef agent, so you can ask follow-ups about it.

### Audio question

```python
question_audio = record_question(duration=5)   # records from your microphone
print(ask_chef_by_audio(question_audio))
```

### One entry point for everything

```python
personal_chef(text="I have lentils, carrots and cumin - what can I make?")
personal_chef(uploader=uploader)          # photo
personal_chef(aud_b64=question_audio)     # audio
```

### Interactive session

```python
run_chef_session()   # /photo, /audio or /quit
```

---

## 🔧 How it works — techniques and where they are used

Everything in this notebook comes from the course notebooks 1.1 → 1.4:

| Feature | Technique | Learned in |
|---|---|---|
| Chat model + agent | `init_chat_model`, `create_agent`, `system_prompt` | 1.1 |
| Clean ingredient list from a photo | Structured output (`response_format` + Pydantic) | 1.1 |
| Answer streamed token by token | `stream(..., stream_mode="messages")` | 1.1 |
| Recipe lookup | `@tool` + Tavily `web_search` added to the agent | 1.2 |
| Memory of previous runs | `InMemorySaver` checkpointer + `thread_id` config | 1.3 |
| Photo input | Multimodal `HumanMessage` with a base64 **image** | 1.4 |
| Voice input | Multimodal `HumanMessage` with a base64 **audio** | 1.4 |

---

## 🧯 Troubleshooting

- **`401` / `API key not valid`** — check that `.env` exists next to the notebook and that
  `load_dotenv()` ran; the keys must not still be the placeholders.
- **`TavilyClient` authentication error** — the `TAVILY_API_KEY` is missing or has no credits.
- **Audio cell does nothing / no microphone** — make sure `sounddevice` is installed and that
  your OS grants microphone access to Python; `scipy` is required to write the WAV file.
- **Upload widget does not appear** — `ipywidgets` must be installed and the cell has to be run
  in Jupyter (not in a plain Python script).
- **Memory seems empty** — the same `thread_id` (default `"my-personal-chef"`) must be reused
  between runs; `InMemorySaver` keeps memory only while the kernel is alive.

---

## 📄 License

MIT — see [LICENSE](LICENSE).
