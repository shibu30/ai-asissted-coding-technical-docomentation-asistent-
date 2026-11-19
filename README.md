# Technical Documentation Assistant (RAG + LLM)

A chatbot for engineering teams that answers questions from a company's own technical documents, and can also analyse a file you upload, such as an error log or a spec. It uses **Retrieval-Augmented Generation (RAG)**, shows the source of every answer, and says so when it cannot find an answer instead of guessing. The app has **login with user and admin roles** and an **admin panel** to manage documents and users.

> We did not train any AI model. An existing LLM is connected to our own documents.

---

## Features

- **Three knowledge sources:** Company Docs, My Upload, or Both
- **Company knowledge base:** pre-loaded API guides, coding standards and runbooks, with categories (the folder name is the category)
- **Upload and ask:** PDF, DOCX, TXT, LOG, JSON (private to the session, deleted when it ends)
- **Four quick actions:** Explain this error, Suggest a fix, Summarize this spec, Check against coding standards
- **Sources shown** with every answer (file, page, text), and an "I couldn't find this" answer when the information is missing
- **Live (streamed) answers** and a chat that remembers the last 6 messages
- **Login and sign-up** with salted PBKDF2 password hashes
- **Two roles:** user (chat and upload) and admin (also manages documents and users)
- **Admin panel:** add or delete company documents, rebuild the knowledge base, create, edit and delete users
- **Free to run:** free embedding model, free LLM API tiers, open-source libraries

---

## How it works

**Preparing documents:** Files -> read text -> cut into chunks (about 1000 characters, 150 overlap) -> embeddings (`bge-small`) -> stored in Chroma.

**Answering a question:** Question -> embedding -> search Chroma -> build prompt (rules + chat history + best chunks + question) -> LLM writes the answer -> answer shown with its sources.

---

## Tech stack

| Part | Tool |
|---|---|
| Language | Python 3 |
| Web interface | Streamlit |
| RAG framework | LangChain |
| File reading | PyPDF, python-docx |
| Embeddings | BAAI/bge-small-en-v1.5 (sentence-transformers, runs locally) |
| Vector database | Chroma |
| User database | SQLite |
| Password hashing | hashlib PBKDF2-HMAC-SHA256 |
| LLM | Groq or Google Gemini (free API), or Ollama (local) |

---

## Project structure

```
tech-doc-assistant/
├── app.py                    # Streamlit app: login gate, chat page
├── ui.py                     # Styling and design
├── admin.py                  # Admin panel (documents and users)
├── auth_ui.py                # Login and sign-up screen
├── config.py                 # Settings
├── requirements.txt
├── .env.example              # Copy to .env and add your key
├── core/
│   ├── loader.py             # Read files, split into chunks, extract log errors
│   ├── vectorstore.py        # Embeddings and Chroma stores
│   ├── ingest.py             # Load company documents into the knowledge base
│   ├── rag.py                # Search and streamed answer
│   ├── llm.py                # Groq / Gemini / Ollama selection
│   ├── prompts.py            # System rules and quick actions
│   └── auth.py               # Users, password hashing, roles
├── scripts/
│   └── ingest_company_docs.py
├── data/
│   └── company_docs/         # api / standards / runbooks (sample files included)
└── samples/
    └── sample_error.log      # Test file for "Explain this error"
```

---

## Setup

You need **Python 3.10 or newer** (3.12 is a safe choice) and a free API key from [Groq](https://console.groq.com) or [Google AI Studio](https://aistudio.google.com).

**1. Get the code and open the project folder**

```bash
git clone https://github.com/YOUR_USERNAME/YOUR_REPO.git
cd YOUR_REPO
```

**2. Create and activate a virtual environment**

Windows (PowerShell):
```powershell
python -m venv venv
Set-ExecutionPolicy -ExecutionPolicy Bypass -Scope Process
venv\Scripts\activate
```

Mac / Linux:
```bash
python3 -m venv venv
source venv/bin/activate
```

**3. Install the packages**

```bash
pip install -r requirements.txt
```

**4. Add your settings**

Copy `.env.example` to `.env`, then open `.env` and set:

```
LLM_PROVIDER=groq              # gemini, groq or ollama
GROQ_API_KEY=your_key_here
GROQ_MODEL=your_model_name     # a model your Groq account can use
# or, for Gemini:
# GOOGLE_API_KEY=your_key_here
# GEMINI_MODEL=your_model_name

ADMIN_USERNAME=admin
ADMIN_PASSWORD=choose_a_strong_password
ALLOW_SIGNUP=true
```

> Free-tier model names change often. If you see a "model not found" error, check the provider's model list and update the model name in `.env`. Never upload your real `.env` file to GitHub.

**5. Build the company knowledge base (once)**

```bash
python -m scripts.ingest_company_docs
```

**6. Run the app**

```bash
python -m streamlit run app.py
```

The app opens at `http://localhost:8501`. Log in with the admin account from your `.env` file. If you did not set one, the first admin is `admin` / `admin123`, so change this password right away in **Admin Panel > Users**.

The first run downloads the free embedding model (about 130 MB).

---

## Usage

1. **Log in** (or sign up as a normal user).
2. In the sidebar choose the **knowledge source**: Company Docs, My Upload, or Both.
3. Type a question, or upload a file and click a **quick action** button.
4. Open **Sources used** under an answer to see the file, page and text it came from.
5. Admins: switch to **Admin Panel** to add or delete company documents, click **Rebuild knowledge base**, and manage users.

To use your own company documents, put files in `data/company_docs/<category>/` (for example `api`, `standards`, `runbooks`) or upload them in the Admin Panel, then rebuild the knowledge base.

### Try these questions (with the sample data)

| Source | Question | Expected answer |
|---|---|---|
| Company Docs | What is the rate limit of the Payment API? | 100 requests per minute |
| Company Docs | Can I use a bare except in Python? | No, catch specific exceptions |
| Company Docs | What is the leave policy? | Says it could not find the answer |
| My Upload | Upload `samples/sample_error.log`, click **Explain this error** | ConnectTimeout to the payment gateway, HTTP 503 |

---

## Security notes

- Passwords are stored as salted PBKDF2 hashes, never as plain text.
- The last admin account cannot be deleted or demoted.
- Uploaded files stay in a temporary, per-session database.
- This is suitable for a project or demo. For real use, add HTTPS, login attempt limits and email password reset.

---

## Future enhancements

- Saved chat history for every user
- OCR for scanned PDFs
- Hybrid search and a reranker
- HTTPS, login limits and email password reset
- A server vector database (Qdrant or pgvector) and online deployment
- More sources (wikis, code repositories, cloud drives)

---

## Team

| Name | Roll number |
|---|---|
| Nishant Vaibhav | 2503A520Q7 |
| Shad Imam | 2503A520R5 |
| Shibu Kumar Singh | 2503A520R4 |
| Md. Raiyaan | 2503A520R6 |

School of Computer Science and Artificial Intelligence, SR University, Warangal.

The code was developed with the help of an AI coding assistant (Claude) and was run and tested by the team.
