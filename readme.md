<div align="center">

<h1 align="center"><b>🛡️ SECURE EHR INSIGHT – CLINICAL VALIDATOR</b></h1>

<br/>

<img src="https://img.shields.io/badge/Python-3.10+-7c3aed?style=for-the-badge&logo=python&logoColor=white" alt="Python"/>
<img src="https://img.shields.io/badge/FastAPI-Backend-059669?style=for-the-badge&logo=fastapi&logoColor=white" alt="FastAPI"/>
<img src="https://img.shields.io/badge/Streamlit-UI-ef4444?style=for-the-badge&logo=streamlit&logoColor=white" alt="Streamlit"/>
<img src="https://img.shields.io/badge/PostgreSQL-pgvector-336791?style=for-the-badge&logo=postgresql&logoColor=white" alt="PostgreSQL"/>
<br/>
<img src="https://img.shields.io/badge/NeMo-Guardrails-76b900?style=for-the-badge&logo=nvidia&logoColor=white" alt="NeMo Guardrails"/>
<img src="https://img.shields.io/badge/Groq-gpt--oss--120b-f97316?style=for-the-badge" alt="Groq"/>
<img src="https://img.shields.io/badge/Google_Cloud-Deployed-4285f4?style=for-the-badge&logo=googlecloud&logoColor=white" alt="GCP"/>
<img src="https://img.shields.io/badge/Status-Working_Prototype-8b5cf6?style=for-the-badge" alt="Status"/>

<br/><br/>

<a href="#-key-features"><kbd> ✨ Features </kbd></a>
<a href="#-how-it-works"><kbd> 🧠 How it works </kbd></a>
<a href="#️-project-structure"><kbd> 🗂️ Structure </kbd></a>
<a href="#-quick-start"><kbd> 🚀 Quick start </kbd></a>
<a href="#️-deployment-on-google-cloud"><kbd> ☁️ Deployment </kbd></a>
<a href="#️-roadmap"><kbd> 🗺️ Roadmap </kbd></a>

</div>

<br/>

> [!WARNING]
> **Research prototype, not a medical device.** Outputs are AI-generated summaries and must never be used for diagnosis or treatment decisions.

---

## 🌟 At a Glance

<table align="center" width="100%">
  <tr>
    <td align="center" width="25%"><h3>🔒</h3><b>1 patient</b><br/><sub>per retrieval scope</sub></td>
    <td align="center" width="25%"><h3>🛡️</h3><b>100%</b><br/><sub>replies pass guardrails</sub></td>
    <td align="center" width="25%"><h3>⚡</h3><b>120B</b><br/><sub>open-weight LLM on Groq</sub></td>
    <td align="center" width="25%"><h3>☁️</h3><b>GCP</b><br/><sub>Ubuntu 22.04 VM</sub></td>
  </tr>
</table>

## ✨ Key Features

<table>
  <tr>
    <td width="33%" valign="top">
      <h3>🎯 Grounded answers</h3>
      Responds only from retrieved record context. If a fact is missing (a drug is listed but no dose is recorded), it says so instead of guessing.
    </td>
    <td width="33%" valign="top">
      <h3>🔒 Patient-scoped retrieval</h3>
      Vector search is locked to one Patient ID, so one patient's data cannot leak into another's answer. The sidebar shows the active scope.
    </td>
    <td width="33%" valign="top">
      <h3>🛡️ Guardrails on every reply</h3>
      <a href="https://docs.nvidia.com/nemo/guardrails/">NVIDIA NeMo Guardrails</a> blocks unauthorized clinical recommendations and appends a safety disclaimer.
    </td>
  </tr>
  <tr>
    <td width="33%" valign="top">
      <h3>🕵️ PII protection</h3>
      <a href="https://microsoft.github.io/presidio/">Microsoft Presidio</a> with a <a href="https://spacy.io">spaCy</a> <code>en_core_web_lg</code> model detects and anonymizes identifiers.
    </td>
    <td width="33%" valign="top">
      <h3>⚡ Fast inference</h3>
      <code>openai/gpt-oss-120b</code> served through the <a href="https://console.groq.com/docs">Groq API</a>.
    </td>
    <td width="33%" valign="top">
      <h3>🧩 Decoupled design</h3>
      A <a href="https://fastapi.tiangolo.com">FastAPI</a> backend and a <a href="https://streamlit.io">Streamlit</a> frontend talk over a clean HTTP API.
    </td>
  </tr>
</table>

## 🧠 How It Works

```mermaid
flowchart LR
    U(["👩‍⚕️ Clinician"]) -->|"question + Patient ID"| UI["🖥️ Streamlit UI<br/>:8501"]
    UI -->|"POST /api/v1/chat"| API["⚙️ FastAPI<br/>:8000"]
    API --> EMB["🔢 Embed query<br/>sentence-transformers"]
    EMB --> DB[("🗄️ PostgreSQL<br/>+ pgvector")]
    DB -->|"top-k chunks<br/>filtered by patient"| LLM["🤖 Groq LLM<br/>gpt-oss-120b"]
    LLM --> GR{{"🛡️ NeMo<br/>Guardrails"}}
    GR -->|"validated answer<br/>+ disclaimer"| API
    API --> UI
    UI --> U

    classDef purple fill:#4c1d95,stroke:#a78bfa,color:#fff;
    classDef cyan fill:#0e7490,stroke:#22d3ee,color:#fff;
    class UI,API,EMB,LLM purple;
    class DB,GR cyan;
```

<details>
<summary><b>📋 Step-by-step request path</b></summary>

1. **Ask.** The user picks a patient in the sidebar and submits a question.
2. **Call.** The UI sends `POST /api/v1/chat` to the FastAPI backend.
3. **Embed.** The question is vectorized with a [sentence-transformers](https://www.sbert.net) model.
4. **Retrieve.** A similarity search runs on [pgvector](https://github.com/pgvector/pgvector), filtered by Patient ID *before* ranking.
5. **Generate.** Retrieved passages and the question go to the LLM with instructions to answer strictly from context.
6. **Guard.** NeMo Guardrails refuses or rewrites disallowed clinical advice and appends `⚠️ AI generated summary. Do not use for diagnostic purposes.`
7. **Return.** The validated answer is displayed in the UI.

</details>

## 🧰 Tech Stack

<table>
  <tr><th align="left">Layer</th><th align="left">Technology</th></tr>
  <tr><td>🖥️ Frontend</td><td><a href="https://streamlit.io">Streamlit</a></td></tr>
  <tr><td>⚙️ Backend</td><td><a href="https://fastapi.tiangolo.com">FastAPI</a> served by <a href="https://www.uvicorn.org">Uvicorn</a></td></tr>
  <tr><td>🗄️ Vector store</td><td><a href="https://www.postgresql.org">PostgreSQL</a> + <a href="https://github.com/pgvector/pgvector">pgvector</a> via <a href="https://www.sqlalchemy.org">SQLAlchemy</a> and <a href="https://www.psycopg.org/psycopg3/">psycopg 3</a></td></tr>
  <tr><td>🔢 Embeddings</td><td><a href="https://www.sbert.net">sentence-transformers</a> (PyTorch, Hugging Face Transformers)</td></tr>
  <tr><td>🤖 LLM</td><td><code>openai/gpt-oss-120b</code> on <a href="https://console.groq.com/docs">Groq</a></td></tr>
  <tr><td>🛡️ Guardrails</td><td><a href="https://docs.nvidia.com/nemo/guardrails/">NeMo Guardrails</a></td></tr>
  <tr><td>🕵️ PII</td><td><a href="https://microsoft.github.io/presidio/">Presidio</a> Analyzer and Anonymizer, <a href="https://spacy.io">spaCy</a></td></tr>
  <tr><td>📊 Utilities</td><td>pandas, NumPy, python-dotenv, requests</td></tr>
  <tr><td>☁️ Hosting</td><td>Google Cloud Compute Engine, Ubuntu 22.04 LTS</td></tr>
</table>

Pinned versions live in [`requirements.txt`](requirements.txt); a [uv](https://docs.astral.sh/uv/) workflow is described in [`uv_instructions.txt`](uv_instructions.txt).

## 🗂️ Project Structure

```text
Secure-EHR-Insight-Clinical-Validator/
├── 📁 data/                  # EHR source data and derived artifacts (keep real PHI out of Git)
├── 📁 instructor_notes/      # Supervision notes and design rationale
├── 📁 scripts/               # Helpers: ingestion, embedding, DB setup
├── 📁 src/
│   ├── 📁 api/
│   │   └── 🐍 main.py        # FastAPI app → POST /api/v1/chat
│   ├── 📁 ui/
│   │   └── 🐍 app.py         # Streamlit frontend, patient selector, chat view
│   └── ...                   # Retrieval / vector search, guardrails config, PII utilities
├── 📄 .gitignore
├── 📄 README.md              # You are here
├── 📄 requirements.txt       # Pinned Python dependencies
└── 📄 uv_instructions.txt    # Setup notes for the uv package manager
```

> [!NOTE]
> `src/api/main.py` and `src/ui/app.py` are the two entry points. Adjust the `src/` sub-tree to match your actual module names.

## 🚀 Quick Start

**Prerequisites:** 🐍 Python 3.10+ · 🐘 PostgreSQL 14+ with the [pgvector extension](https://github.com/pgvector/pgvector#installation) · 🔑 a [Groq API key](https://console.groq.com/keys) · 💾 about 4 GB free disk

### 1️⃣ Install

```bash
git clone https://github.com/Raghav0079/Secure-EHR-Insight-Clinical-Validator.git
cd Secure-EHR-Insight-Clinical-Validator

python3 -m venv venv
source venv/bin/activate          # Windows: venv\Scripts\activate

pip install --upgrade pip
pip install -r requirements.txt
```

If the `en_core_web_lg` wheel fails to install, run `python -m spacy download en_core_web_lg` instead.

### 2️⃣ Configure

Create a `.env` in the project root (never commit it):

```env
# Database
DB_HOST=localhost
DB_PORT=5432
DB_NAME=ehr_db
DB_USER=ehr_user
DB_PASSWORD=change_me

# LLM
GROQ_API_KEY=your_groq_api_key
LLM_MODEL=openai/gpt-oss-120b

# Backend URL used by the Streamlit UI
API_URL=http://localhost:8000
```

> [!TIP]
> Variable names above are a template; match them to what `src/` reads via `python-dotenv`.

### 3️⃣ Set up the database

```bash
sudo -u postgres psql
```

```sql
CREATE USER ehr_user WITH PASSWORD 'change_me';
CREATE DATABASE ehr_db OWNER ehr_user;
\c ehr_db
CREATE EXTENSION IF NOT EXISTS vector;
```

Then run the ingestion script from `scripts/` to chunk, embed, and insert the clinical notes with their Patient IDs.

### 4️⃣ Run

Start the backend and frontend as **two separate processes**; the UI cannot answer until the API is up.

```bash
# Terminal 1 – FastAPI backend
source venv/bin/activate
uvicorn src.api.main:app --host 0.0.0.0 --port 8000 --reload
```

```bash
# Terminal 2 – Streamlit UI
source venv/bin/activate
streamlit run src/ui/app.py --server.address=0.0.0.0 --server.port=8501
```

<table align="center">
  <tr><th>Service</th><th>URL</th></tr>
  <tr><td>🖥️ Streamlit UI</td><td><code>http://localhost:8501</code></td></tr>
  <tr><td>⚙️ FastAPI</td><td><code>http://localhost:8000</code></td></tr>
  <tr><td>📖 Interactive API docs</td><td><code>http://localhost:8000/docs</code></td></tr>
</table>

## 🔌 API Reference

### `POST /api/v1/chat`

<table>
<tr>
<td width="50%" valign="top">

**Request** (align field names with your Pydantic model)

```json
{
  "patient_id": "10000032",
  "question": "What was the patient's last recorded dosage of Furosemide?"
}
```

</td>
<td width="50%" valign="top">

**Response**

```json
{
  "answer": "Furosemide is listed in the record, but no explicit dosage is documented.\n\n⚠️ AI generated summary. Do not use for diagnostic purposes."
}
```

</td>
</tr>
</table>

## ☁️ Deployment on Google Cloud

Reference deployment: Compute Engine VM `ehr-validator-vm`, Ubuntu 22.04 LTS.

1. 🏗️ **Provision** the VM; install PostgreSQL, pgvector, Python 3, and `python3-venv`.
2. 🔐 **Set up SSH access.** If the login user and project owner differ, fix directory and `~/.ssh` permissions so the SSH user can read what it needs. Ownership mismatch was the main access issue during setup.
3. 📦 **Clone and install** under the project owner's home, create the `venv`, and export the `.env` values.
4. ▶️ **Start both servers** in separate SSH sessions (or use `tmux`, `screen`, or `systemd` for persistence).
5. 🚇 **Tunnel the ports** rather than exposing them. From Windows PowerShell:

```powershell
ssh -L 8501:localhost:8501 -L 8000:localhost:8000 <ssh-user>@<vm-external-ip>
```

Browse to `http://localhost:8501` (an Incognito window avoids stale sessions).

> [!CAUTION]
> Do not expose ports 8000 or 8501 publicly without authentication and TLS; the app fronts clinical data. See the [GCP firewall docs](https://cloud.google.com/firewall/docs/firewalls) to restrict access.

## ✅ Validation Results

<table>
  <tr><th align="left">Check</th><th>Result</th></tr>
  <tr><td>Vector store: PostgreSQL + pgvector stores and searches embedded EHR records</td><td align="center">✅</td></tr>
  <tr><td>Retrieval scope: context locked to Patient ID <code>10000032</code></td><td align="center">✅</td></tr>
  <tr><td>Groundedness: recognized Furosemide in the record and correctly reported the missing dosage, with no hallucination</td><td align="center">✅</td></tr>
  <tr><td>Guardrails: response intercepted and the disclaimer appended</td><td align="center">✅</td></tr>
  <tr><td>Connectivity: <code>[Errno 111]</code> and HTTP 500 errors resolved by running backend and frontend concurrently</td><td align="center">✅</td></tr>
</table>

## 🛡️ Safety and Privacy Design

- 🎯 **Least context.** Retrieval is filtered by Patient ID before ranking, so the LLM never sees another patient's text.
- 🚫 **Refuse rather than advise.** Guardrails target prescriptive requests (dose changes, treatment choices).
- 🤷 **Abstain on missing data.** The model is instructed to state when the record lacks the requested fact.
- 🔑 **Secrets out of Git.** Keys and credentials live in `.env`.
- 🧪 **De-identified data only.** Develop and demo with public, de-identified, or synthetic records. Credentialed datasets usually forbid sending text to third-party APIs; check the data use agreement.
- ⚖️ **Compliance.** Real patient data needs HIPAA or India's DPDP Act work that this prototype does not provide.

## 🩹 Troubleshooting

<details>
<summary><b>🔌 <code>[Errno 111] Connection refused</code> in the UI</b></summary>

The backend is not running or `API_URL` is wrong. Start uvicorn first, then the UI.

</details>

<details>
<summary><b>💥 HTTP 500 from <code>/api/v1/chat</code></b></summary>

Check the uvicorn logs. Common causes: missing `.env` value, database connection failure, or the `vector` extension not created.

</details>

<details>
<summary><b>🔐 <code>permission denied</code> over SSH</b></summary>

Directory ownership mismatch between the SSH user and the project owner. Fix with `chown`/`chmod`, or work as the owner.

</details>

<details>
<summary><b>🧬 spaCy model not found</b></summary>

Run `python -m spacy download en_core_web_lg` inside the virtual environment.

</details>

<details>
<summary><b>🚇 Tunnel works but the page is blank</b></summary>

Confirm Streamlit is bound to `0.0.0.0:8501` and both `-L` forwards are active.

</details>

## ⚕️ Disclaimer

This software is for research and educational purposes only. It is not a medical device, has not been clinically validated, and must not be used to make or support diagnostic or treatment decisions. Always consult a qualified clinician.

## 📜 License

Add a `LICENSE` file (MIT is a common choice for prototypes) and reference it here.

<div align="center">

**Built by [Raghav](https://github.com/Raghav0079)** · ⭐ Star the repo if it helped you

<img src="https://capsule-render.vercel.app/api?type=waving&section=footer&height=120&color=0:22d3ee,100:4c1d95" width="100%" alt="footer wave"/>

</div>
