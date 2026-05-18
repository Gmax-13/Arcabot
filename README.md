# Arcabot – Discord Knowledge Assistant

**Arcabot** is a Discord bot that lets you query over your own documents (PDF, PPTX, DOCX, MD, TXT) using semantic search plus optional LLM‑generated answers (via **Groq**).
It builds a FAISS vector store from the files in a `knowledge/` folder, serves the most relevant snippets, and can refresh the index on‑demand.

> **Why Arcabot?**
> • Works with any file type you add to `knowledge/`.
> • No Docker required for production – can be deployed on Render.
> • Fully self‑hosted; all data stays in your own workspace.

---

## Table of Contents

1. [Features](#features)
2. [Quick Start (local)](#quick-start-local)
3. [Configuration & Environment Variables](#configuration--environment-variables)
4. [Running the Bot](#running-the-bot)
5. [Deploying to Render (no Docker)](#deploying-to-render-no-docker)
6. [Managing Knowledge Files](#managing-knowledge-files)
7. [Bot Commands (Discord)](#bot-commands-discord)
8. [Troubleshooting & FAQ](#troubleshooting--faq)
9. [Contributing](#contributing)
10. [License](#license)

---

## Features

| Feature | Description |
|---------|-------------|
| **File type support** | PDF (via `pdfplumber` + fallback `PyPDF2`), PowerPoint (`.pptx`), Word (`.docx`), Markdown (`.md`) and plain text (`.txt`). |
| **Semantic search** | Uses `sentence‑transformers/all‑MiniLM‑L6‑v2` + FAISS for fast nearest‑neighbour retrieval. |
| **LLM‑generated answers** | Optional Groq integration (`gemma2-9b-it` by default). If no Groq key is provided, the bot falls back to raw snippet replies. |
| **Dynamic refresh** | `?refresh` rebuilds the vector store on‑the‑fly, no container restart needed (when using Render’s persistent disk). |
| **Pagination** | `?more` returns the next batch of relevant chunks (3 at a time). |
| **Admin controls** | `?set_channel <id>` and `?set_role <role>` (admin‑only) to change the monitored channel or required role at runtime. |
| **Safety** | All file‑parsing errors are caught and logged; a problematic PDF never crashes the bot. |
| **Deploy‑free** | No Docker needed – Render’s default Python buildpack handles everything. |
| **Persisted knowledge** | Optional Render persistent disk keeps your `knowledge/` folder across restarts. |
| **Ignorable files** | `.knowledgeignore` lets you exclude large or noisy files via glob patterns. |

---

## Quick Start (local)

> **Prerequisites**
> - Python 3.10 or newer
> - Git
> - A Discord bot token (see *Configuration* below)

```bash
# 1️⃣ Clone the repo
git clone https://github.com/yourusername/arcabot.git
cd arcabot

# 2️⃣ Create a virtual environment
python -m venv venv
source venv/bin/activate   # Windows: venv\Scripts\activate

# 3️⃣ Install dependencies
pip install -r requirements.txt

# 4️⃣ Create a .env file (see next section for required keys)
cp .env-sample.txt .env
# Edit .env with your credentials

# 5️⃣ Run the bot
python discord_bot.py
```

If everything is set up correctly you’ll see:

```
Arcabot#1234 has connected to Discord!
Built FAISS index with X chunks.
```

Now go to your Discord server and start using the commands (see *Bot Commands*).

---

## Configuration & Environment Variables

All secrets are read from environment variables (or a `.env` file when running locally). Render will inject the same variables via its UI.

| Variable | Required? | Description |
|----------|----------|--------------|
| `DISCORD_BOT_TOKEN` | ✅ | Bot token from the Discord Developer Portal. Must have **Message Content Intent** enabled. |
| `GROQ_API_KEY` | ❌ (optional) | API key for Groq. If omitted the bot replies with raw chunks only. |
| `HF_TOKEN` | ❌ (optional) | Hugging‑Face token to avoid rate‑limit warnings when loading the transformer model. |
| `PROJECT_UPDATES_CHANNEL_ID` | ✅ | Discord channel ID where the bot should listen (the default “project‑updates” channel). |
| `BOT_ROLE_NAME` | ✅ | Name of the Discord role that is allowed to use the bot (e.g., `PM Team`). |
| `KNOWLEDGE_DIR` | ✅ | Path to the folder containing your docs. For Render with a persistent disk: `/app/knowledge`. For local testing: `knowledge`. |
| `GROQ_MODEL` | ❌ (optional) | Which Groq model to use (default: `gemma2-9b-it`). |
| `OPENAI_MODEL` | ❌ (legacy) | Ignored – kept for compatibility only. |

### Example `.env.example`

```dotenv
DISCORD_BOT_TOKEN=YOUR_DISCORD_BOT_TOKEN
GROQ_API_KEY=YOUR_GROQ_API_KEY
HF_TOKEN=YOUR_HF_TOKEN
PROJECT_UPDATES_CHANNEL_ID=1502599788707975198
BOT_ROLE_NAME=PM Team
KNOWLEDGE_DIR=knowledge
```

> **Never** commit a real `.env` file – keep it local or use Render’s secret management.

---

## Running the Bot

### Local (development)

```bash
source venv/bin/activate
python discord_bot.py
```

The bot will:

1. Load all supported files from `KNOWLEDGE_DIR`.
2. Build a FAISS vector index (chunks of 500 characters).
3. Connect to Discord and wait for `?`-prefixed commands.

### Production (Render)

Deploying to Render is covered in the next section; once the service is live the bot runs automatically and stays up as long as the container is healthy.

---

## Deploying to Render (no Docker)

Render’s **Python buildpack** does all the heavy lifting. The repo already contains a `render.yaml` that tells Render how to build and start the bot.

### 1️⃣ Push your repo to GitHub (if you haven’t already)

```bash
git remote add origin https://github.com/yourusername/arcabot.git
git push -u origin main
```

Make the repository **private** if it contains any internal documents. Render supports private repos on the free tier.

### 2️⃣ Create a Render Web Service

1. Log in to <https://render.com> (sign up for a free account).
2. Click **New → Web Service**.
3. Choose **Connect a repository**, select your GitHub repo and the `main` branch.
4. Render will automatically detect the `render.yaml` file.

   The file includes:

   ```yaml
   buildCommand: pip install -r requirements.txt
   startCommand: |
     python -m http.server 5000 &
     python discord_bot.py
   ```  

   The dummy HTTP server on port 5000 satisfies Render’s health‑check requirement.  

5. **Add environment variables** (same names as described above).  
6. (Optional) **Enable a persistent disk**: uncomment the `disk:` block in `render.yaml` and set `KNOWLEDGE_DIR` to `/app/knowledge`. This lets you upload new PDFs without rebuilding.  
7. Click **Create Web Service**.  

Render will clone the repo, install dependencies, start the dummy HTTP server, then launch `discord_bot.py`.  

### 3️⃣ Verify the deployment  

- Open the **Logs** tab – you should see “Arcabot#xxxx has connected to Discord!” and “Built FAISS index …”.  
- In Discord, type `?refresh`. The bot should reply with a confirmation.  

### 4️⃣ Updating knowledge files  

| Method | When to use | Steps |
|--------|-------------|-------|
| **Commit & push** | Small changes, you’re fine with a rebuild. | `git add knowledge/newfile.pdf && git commit -m "Add new doc" && git push`. Render auto‑deploys (takes ~30 s). |
| **Upload to persistent disk** (recommended for frequent updates) | You enabled the `disk:` block. | In Render, open **Shell** → `cd /app/knowledge` → `curl -O <url>` or `scp` the file. Then run `?refresh` in Discord. No rebuild needed. |
| **Render Files UI** (if you enabled a disk) | Very small manual uploads. | Use the **Files** view in the service dashboard to drag‑and‑drop PDFs. |

---  

## Managing Knowledge Files  

Your documents live under `knowledge/` (or `/app/knowledge` on Render).  

### Adding a file  

```bash
# Local
cp path/to/mydoc.pdf knowledge/
git add knowledge/mydoc.pdf
git commit -m "Add mydoc.pdf"
git push
```

### Ignoring files  

Create (or edit) `knowledge/.knowledgeignore` using the same syntax as `.gitignore`. Example:

```
# Skip large archives
*.zip
large/*
# Skip all PDFs temporarily
*.pdf
```

The bot reads this file at startup (or on `?refresh`) and silently skips matching paths.

---

## Bot Commands (Discord)

All commands must be sent **in the channel whose ID you set in `PROJECT_UPDATES_CHANNEL_ID`** (or the bot will ignore them).

| Command | Description | Example |
|---------|-------------|---------|
| `?refresh` | Re‑build the FAISS index (use after adding/modifying files). | `?refresh` |
| `?more` | Paginate the previous answer – shows the next 3 chunks. | `?more` |
| `?set_channel <id>` | **Admin only** – change the channel the bot listens to. | `?set_channel 123456789012345678` |
| `?set_role <role>` | **Admin only** – change the Discord role required to use the bot. | `?set_role Knowledge‑Bot` |
| `?help` | Shows a short help message (built‑in). | `?help` |
| `?<question>` | Ask a question about your knowledge base. The bot will retrieve the most relevant chunks and, if a Groq key is set, generate a concise answer. | `?What is the Q3 roadmap?` |

**Permission model**

- Only members with the role name set in `BOT_ROLE_NAME` can invoke the bot.
- Admins (Discord `Administrator` permission) can change the monitored channel / required role via the admin commands.

---

## Troubleshooting & FAQ

| Issue | Likely cause | Resolution |
|-------|---------------|------------|
| Bot doesn’t connect (`Arcabot#xxxx` never appears) | Wrong token **or** Message Content Intent not enabled. | Verify `DISCORD_BOT_TOKEN` and enable “Message Content Intent” in the Discord Developer Portal → *Bot → Privileged Gateway Intents*. |
| “Rate limit” warnings from Hugging Face | `HF_TOKEN` missing. | Add a Hugging‑Face token (`HF_TOKEN`) in Render env or local `.env`. |
| Indexing hangs on a PDF (heartbeat warnings) | Corrupt or extremely large PDF. | Add the PDF pattern to `.knowledgeignore` or split the PDF. |
| `ImportError: No module named 'groq'` | `groq` not installed / `requirements.txt` out‑of‑date. | Ensure `groq` is listed in `requirements.txt` and push a new commit (Render will reinstall). |
| After adding a file, `?refresh` seems to do nothing | File was added to the **persistent disk** but the bot still looks at the repo path. | Make sure `KNOWLEDGE_DIR` is set to `/app/knowledge` (Render) or to `knowledge` (local). |
| Health check fails on Render | No process listening on a port. | The `startCommand` includes `python -m http.server 5000 &`. Verify the `&` is present. |
| Bot replies with raw chunks even though I have a Groq key | Wrong `GROQ_API_KEY` (typo) or the key is invalid. | Verify the key in Render’s environment variables. Check Render logs for `LLM generation failed` messages. |
| `?more` says “No previous query to continue.” | You used `?more` before asking a question, or the bot restarted. | Issue a new query first (`?What is X?`) then `?more`. |
| `?set_channel` returns “Only administrators can change the monitored channel.” | You are not an admin in the Discord server. | Ask a server admin to run the command, or give yourself the `Administrator` permission. |

### Getting help

- Check the **Render Logs** for Python tracebacks.
- Use the **Shell** in Render to inspect `/app/knowledge` (`ls -R /app/knowledge`).
- Open an issue on GitHub with the error message and a short description.

---

## Contributing

Contributions are welcome!

1. Fork the repo.
2. Create a feature branch (`git checkout -b feat/awesome-feature`).
3. Make your changes – please keep the bot’s public interface (`?` commands) backward compatible.
4. Run the test suite (if you add tests).
5. Submit a Pull Request.

### Development guidelines

- Follow PEP 8.
- Keep the `requirements.txt` minimal and pinned only to the major version you need.
- Add a line to `README.md` for any new public command.

---

## License

This project is licensed under the **MIT License** – see the `LICENSE` file for details.

---

**Enjoy querying your own knowledge base on Discord!** 🎉

If you have any questions or run into issues, feel free to open a GitHub issue or ping me on Discord.
