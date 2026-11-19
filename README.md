<h1 align="center">Ambi</h1>

<p align="center"><strong>A searchable memory of everything you read on the web, running entirely on your own machine.</strong><br/>
Press <kbd>⌘</kbd> <kbd>Shift</kbd> <kbd>K</kbd> on any page and find that article you half-remember by what it meant, not the exact words.</p>

<p align="center">
  <img alt="Chrome MV3" src="https://img.shields.io/badge/Chrome-Manifest%20V3-4285F4?logo=googlechrome&logoColor=white">
  <img alt="FastAPI" src="https://img.shields.io/badge/FastAPI-009688?logo=fastapi&logoColor=white">
  <img alt="ChromaDB" src="https://img.shields.io/badge/ChromaDB-vectors-FF6F00">
  <img alt="sentence-transformers" src="https://img.shields.io/badge/sentence--transformers-MiniLM-FFD21E?logo=huggingface&logoColor=black">
  <img alt="SQLite" src="https://img.shields.io/badge/SQLite-003B57?logo=sqlite&logoColor=white">
</p>

## Why

Browser history only stores titles and URLs, so "that post about vector databases I read last week" is gone unless you remember the exact title. Cloud "memory" tools solve this by uploading everything you read. Ambi solves it without anything leaving your laptop: no cloud, no account, no tracking.

## What it does

- **Remembers what you actually read.** The extension saves a page only after you've stayed on it for 10+ seconds, and extracts the clean article text with Mozilla's Readability, so skimmed tabs and page clutter stay out.
- **Searches by meaning.** Each page is embedded locally with `all-MiniLM-L6-v2` and stored in ChromaDB. A query like *"how do transformers handle long context"* finds the right page even if it never used those words.
- **Groups pages into topics.** New pages join topic clusters automatically. Clusters merge when they drift together and relabel themselves as they grow, and search results from your main topic get ranked higher.
- **Ranks like you think.** Each result's score blends meaning (60%), keyword match (25%), how recently you read it, how long you spent on it, and whether it belongs to the topic you're exploring right now.
- **Captures your X/Twitter bookmarks.** Scroll through your bookmarks once and every tweet you see is saved, so you can search them later.
- **Works on any page.** The search panel opens on top of whatever site you're on, isolated in a Shadow DOM so page styles can't break it. It still works in tabs that were open before the extension loaded.
- **Respects private browsing.** Incognito tabs are never recorded.
- **Lets you forget.** Remove any page from the panel with one click, or call `DELETE /delete-by-url`.

## How it works

```
Chrome extension (MV3, plain JS)               FastAPI server on 127.0.0.1
──────────────────────────────────              ────────────────────────────────
content.js   dwell timer + Readability  ──POST /ingest──▶ embed (MiniLM, local)
background.js message routing                             SQLite  (page records)
panel        ⌘⇧K search overlay         ──POST /search──▶ ChromaDB (vectors)
                                        ◀── ranked results, cluster-boosted
                                                          GET /clusters  (topics)
```

## Run it

Requires Python 3.10+ and Chrome.

```bash
git clone https://github.com/Sunny-ybe/Ambi-local.git
cd Ambi-local
python3 -m venv ambivenv
ambivenv/bin/pip install -r server/requirements.txt
ambivenv/bin/uvicorn server.main:app --reload     # http://127.0.0.1:8000
```

Then load the extension: open `chrome://extensions`, turn on **Developer mode**, click **Load unpacked**, and pick the `extension/` folder. Browse normally, then press <kbd>⌘</kbd> <kbd>Shift</kbd> <kbd>K</kbd> (<kbd>Ctrl</kbd> <kbd>Shift</kbd> <kbd>K</kbd> on Windows/Linux).

## API

| Method | Route | Purpose |
| --- | --- | --- |
| `POST` | `/ingest` | Save a page (URL, title, text, dwell time) |
| `POST` | `/search` | Semantic search with cluster-aware ranking |
| `GET` | `/clusters` | List topic clusters with sample pages |
| `DELETE` | `/delete/{id}`, `/delete-by-url` | Forget a page |
| `GET` | `/health` | Health check |

## Privacy

All data lives in `server/ambi_v0.db` (SQLite) and `server/chroma_db/` (vectors), both gitignored. The embedding model runs on your CPU. The server listens only on `127.0.0.1`.

## How I built it

Development used a small team of Claude Code subagents with fixed roles and file ownership (code writer, security auditor, test writer, reviewer), run in a set order for every feature. The routing rules are in [`CLAUDE.md`](CLAUDE.md) and the agent definitions in [`.claude/agents`](.claude/agents).
