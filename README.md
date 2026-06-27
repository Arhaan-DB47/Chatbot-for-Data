# 📄 Chatbot for Your Data

> A RAG-powered (Retrieval-Augmented Generation) chatbot that lets you **upload a PDF document** and ask questions about its contents — using **LLaMA** via IBM watsonx for intelligent responses and **ChromaDB** for vector-based document retrieval.

Built as part of **Course 8 – Building Generative AI-Powered Applications with Python** in the [IBM AI Developer Professional Certificate](https://www.coursera.org/professional-certificates/applied-artifical-intelligence-ibm-watson-ai).

---

## 📌 Overview

This project implements a full **Retrieval-Augmented Generation (RAG)** pipeline. Instead of relying solely on an LLM's training data, the chatbot first retrieves relevant chunks from your uploaded PDF, then uses LLaMA to generate accurate, context-grounded answers.

### Architecture

```
┌──────────────────────────────────────────────────────────┐
│                        Browser                           │
│  ┌────────────────────────────────────────────────────┐  │
│  │  index.html + style.css + script.js                │  │
│  │  • Upload PDF via button                           │  │
│  │  • Ask questions in chat interface                 │  │
│  │  • Light/Dark mode toggle                          │  │
│  │  • Reset chat button                               │  │
│  └─────────┬─────────────────────┬────────────────────┘  │
│     POST /process-document    POST /process-message      │
│     (PDF file upload)         (user question)            │
└────────────┼─────────────────────┼────────────────────────┘
             ▼                     ▼
┌──────────────────────────────────────────────────────────┐
│              Flask Backend (server.py)                    │
│                                                          │
│  worker.py — RAG Pipeline:                               │
│  ┌────────────────────────────────────────────────────┐  │
│  │  1. PyPDFLoader → load PDF                         │  │
│  │  2. RecursiveCharacterTextSplitter → chunk text     │  │
│  │  3. HuggingFace Embeddings → vectorize chunks      │  │
│  │  4. ChromaDB → store & retrieve vectors            │  │
│  │  5. LLaMA (IBM watsonx) → generate answer          │  │
│  │  6. LangChain RetrievalQA → orchestrate pipeline   │  │
│  └────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────┘
```

---

## 📂 Project Structure

```
Chatbot-for-Data/
├── Lab - Build a Chatbot for Your Data.pdf   # Lab instructions document
└── build_chatbot_for_your_data/
    ├── server.py                 # Flask server — completed (routes for chat & upload)
    ├── server_exercise.py        # Exercise skeleton with TODOs
    ├── worker.py                 # RAG pipeline — completed (watsonx + LangChain)
    ├── Worker_completed.py       # Reference solution for worker
    ├── worker_huggingFace.py     # Alternative worker using HuggingFace Hub
    ├── Dockerfile                # Docker container configuration
    ├── requirements.txt          # Python dependencies
    ├── LICENSE                   # Apache 2.0 License
    ├── templates/
    │   └── index.html            # Chat UI with PDF upload & dark mode
    └── static/
        ├── script.js             # Frontend logic — upload, messaging, reset
        └── style.css             # Light/dark mode styling with animations
```

---

## 🚀 Getting Started

### Prerequisites

- Python 3.10+
- Access to **IBM watsonx.ai** (provided in the IBM Skills Network lab environment)

### Run Locally

```bash
# Clone the repository
git clone https://github.com/Arhaan-DB47/Chatbot-for-Data.git
cd Chatbot-for-Data/build_chatbot_for_your_data

# Install dependencies
pip install -r requirements.txt

# Start the server
python server.py
```

The app will be available at `http://0.0.0.0:8000`.

### Run with Docker

```bash
cd Chatbot-for-Data/build_chatbot_for_your_data

# Build the Docker image
docker build -t chatbot-for-data .

# Run the container
docker run -p 8000:8000 chatbot-for-data
```

---

## 📖 How It Works

### Backend

#### `server.py` — Flask Server

Exposes three routes:

| Route | Method | Description |
|-------|--------|-------------|
| `/` | GET | Serves the chat interface |
| `/process-document` | POST | Receives a PDF upload, processes it through the RAG pipeline, and confirms readiness |
| `/process-message` | POST | Receives a user question, retrieves relevant PDF chunks, and returns an LLM-generated answer |

#### `worker.py` — RAG Pipeline (Primary)

The core module that orchestrates the entire RAG workflow:

1. **`init_llm()`** — Initializes the LLaMA model via IBM watsonx and loads HuggingFace sentence embeddings (`all-MiniLM-L6-v2`)
2. **`process_document(path)`** — Loads a PDF with `PyPDFLoader`, splits it into 1024-character chunks (64-char overlap), creates vector embeddings, and stores them in ChromaDB. Builds a `RetrievalQA` chain using MMR (Maximal Marginal Relevance) search with `k=6`
3. **`process_prompt(prompt)`** — Queries the RetrievalQA chain with the user's question and conversation history, returns the generated answer

#### Worker Variants

| File | LLM Provider | Embeddings |
|------|-------------|------------|
| `worker.py` | IBM watsonx (LLaMA 4 Maverick 17B) | `HuggingFaceEmbeddings` |
| `Worker_completed.py` | IBM watsonx (LLaMA 3.3 70B) | `HuggingFaceInstructEmbeddings` |
| `worker_huggingFace.py` | HuggingFace Hub (Falcon-7B-Instruct) | `HuggingFaceInstructEmbeddings` |

### Frontend — `index.html` + `script.js` + `style.css`

A polished chat interface built with Bootstrap 4 and jQuery:

- **Guided flow** — On first load, the bot greets the user and presents an **Upload File** button. The send button is disabled until a PDF is uploaded
- **PDF upload** — Accepts `.pdf` files, sends them to `/process-document` via `FormData`, and confirms when analysis is complete
- **Question answering** — After uploading, users can type questions and receive context-aware answers from the RAG pipeline
- **Reset button** — Clears chat history and allows uploading a new document
- **Light/Dark mode** — Toggle switch for theme preference
- **Loading animations** — Bouncing dots during processing
- **Input sanitization** — Strips HTML tags, special characters, and whitespace

---

## 🧠 RAG Pipeline — Key Concepts

| Component | Technology | Purpose |
|-----------|-----------|---------|
| **Document Loading** | PyPDFLoader | Extracts text from uploaded PDF files |
| **Text Splitting** | RecursiveCharacterTextSplitter | Breaks documents into 1024-char chunks with 64-char overlap |
| **Embeddings** | sentence-transformers/all-MiniLM-L6-v2 | Converts text chunks into vector representations |
| **Vector Store** | ChromaDB | Stores and retrieves document vectors using MMR search |
| **LLM** | LLaMA (via IBM watsonx) | Generates natural-language answers from retrieved context |
| **Orchestration** | LangChain RetrievalQA | Chains retrieval and generation into a single pipeline |

---

## 🛠️ Technologies & Libraries

| Technology | Purpose |
|------------|---------|
| [Flask](https://flask.palletsprojects.com/) | Python web server |
| [Flask-CORS](https://flask-cors.readthedocs.io/) | Cross-origin request support |
| [LangChain](https://python.langchain.com/) | RAG pipeline orchestration |
| [IBM watsonx.ai](https://www.ibm.com/products/watsonx-ai) | LLaMA model hosting and inference |
| [ChromaDB](https://www.trychroma.com/) | Vector database for document embeddings |
| [Hugging Face Transformers](https://huggingface.co/docs/transformers) | Sentence embeddings |
| [PyPDF](https://pypdf.readthedocs.io/) | PDF text extraction |
| [Bootstrap 4](https://getbootstrap.com/docs/4.5/) | Responsive UI framework |
| [jQuery](https://jquery.com/) | DOM manipulation and AJAX |
| [Docker](https://www.docker.com/) | Containerized deployment |

---

## 🔗 Related Projects

This project is part of a series built during the IBM AI Developer Professional Certificate (Course 8):

| Project | Description |
|---------|-------------|
| [Image Captioning with BLIP & Gradio](https://github.com/Arhaan-DB47/Img_captioning) | AI-powered image captioning — from local inference to automated web scraping |
| [Simple Chatbot Using LLM](https://github.com/Arhaan-DB47/Simple-chatbot-using-LLM) | Terminal-based chatbots using BlenderBot and SmolLM2 |
| [Integrate Chatbot into Web App](https://github.com/Arhaan-DB47/Integrate-chatbot-into-web-application) | Full-stack web chatbot with Flask + custom UI + Docker |
| [Voice Assistant](https://github.com/Arhaan-DB47/Voice-Assistant) | Voice-enabled assistant with OpenAI GPT + IBM Watson STT/TTS |
| [AI Meeting Companion](https://github.com/Arhaan-DB47/AI-Meeting-Companion) | Audio transcription + LLM key-point extraction with Whisper + LLaMA |
| **Chatbot for Your Data** *(this repo)* | RAG chatbot — upload PDFs and ask questions with LLaMA + ChromaDB |

---

## 📜 License

Licensed under the [Apache License 2.0](./build_chatbot_for_your_data/LICENSE).

Original template copyright © 2020 IBM Developer Skills Network.

---

## 👤 Author

**Arhaan Khan** — [@Arhaan-DB47](https://github.com/Arhaan-DB47)
