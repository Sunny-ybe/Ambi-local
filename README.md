# Ambi

Local-first browsing memory. A Chrome extension captures pages you dwell on (≥ 10s),
sends them to a local FastAPI backend, and lets you search everything semantically
from a `Cmd+Shift+K` overlay on any page.

No cloud, no account — everything stays on your machine.

## Run it

```bash
# backend
cd ~/Desktop/ambi
source ambivenv/bin/activate
uvicorn server.main:app --reload            # http://127.0.0.1:8000
```

Then load the extension: `chrome://extensions` → Developer mode → **Load unpacked** → select `extension/`.

## Layout

| Path | What |
|---|---|
| `extension/` | Chrome MV3 extension — plain JS, no build step |
| `server/` | FastAPI backend: ingest, semantic search, clustering |
| `ambivenv/` | Python virtualenv — rebuild with `python3 -m venv ambivenv && ambivenv/bin/pip install -r server/requirements.txt` |

Architecture, message protocol, scoring, and cluster internals: [CLAUDE.md](CLAUDE.md).

## Data

SQLite at `server/ambi_v0.db`, vectors in `server/chroma_db/` — both gitignored and local only.
Remove a page from the search panel's × button or `DELETE /delete-by-url`.
