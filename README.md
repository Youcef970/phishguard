## Project structure

```
phishguard/
├── backend/
│   ├── api/            # routes.py, models.py
│   ├── core/            # url_analyzer, context_analyzer, visual_matcher, risk_scorer
│   ├── data/            # training_samples.py
│   ├── utils/           # validators.py
│   ├── main.py
│   └── requirements.txt
├── frontend/
│   ├── src/
│   │   ├── components/   # Layout, Dashboard, EmailAnalyzer, TrainingSimulator, Stats
│   │   ├── styles/globals.css
│   │   ├── App.jsx, main.jsx, api.js
│   ├── index.html
│   ├── package.json
│   ├── vite.config.js
│   ├── postcss.config.js
│   └── tailwind.config.js
├── extension/
│   ├── manifest.json
│   ├── background.js
│   ├── content.js / content.css
│   ├── popup.html / popup.js
│   └── assets/icon-*.png

```

## Running locally (without Docker)

**Backend** (Python 3.11+):
```bash
cd backend
pip install -r requirements.txt
python main.py
# → http://localhost:8000  (docs at /api/docs)
```

**Frontend** (Node 20+), in a second terminal:
```bash
cd frontend
npm install
npm run dev
# → http://localhost:3000, proxies /api to the backend on :8000
```

## Browser extension

`extension/` is a Manifest V3 Chrome/Edge extension that scans the active tab's
URL as you browse and shows a small on-page indicator plus a popup summary.

To load it:
1. Run the backend (`http://localhost:8000`) — the extension calls it directly.
2. Go to `chrome://extensions`, enable Developer mode, click **Load unpacked**,
   and select the `extension/` folder.
3. Visit any `http(s)://` page — a small marker appears bottom-right; click the
   toolbar icon for a popup summary.

It calls `POST /api/v1/analyze/quick`, a URL-only variant of `/analyze` that
skips context/visual analysis (there's no email body or screenshot on a bare
page navigation) and scores purely on URL structure.

## What's real vs. illustrative

- **URL analysis** (entropy, suspicious TLDs, IP-literal hosts, subdomain brand
  abuse, redirect params) and **context analysis** (urgency, threats, sensitive-info
  requests, authority/secrecy language) are functioning rule-based engines — they
  score real input.
- **Visual brand matching** implements a real perceptual-hash + Hamming-distance
  pipeline, but ships with placeholder brand fingerprints rather than hashes
  generated from real captured screenshots. Swap in real reference hashes
  before relying on it for production brand-impersonation detection.
- **Scan history and stats** are stored in memory and reset whenever the backend
  restarts. Swap in a real datastore for persistence across restarts/multiple
  instances.

## API endpoints

| Method | Path | Purpose |
|---|---|---|
| POST | `/api/v1/analyze` | Score a URL and/or message body |
| POST | `/api/v1/analyze/quick` | URL-only score, used by the browser extension |
| GET | `/api/v1/training/samples` | List training samples (filter by `difficulty`, `category`) |
| POST | `/api/v1/training/validate` | Check a training attempt's identified flags |
| GET | `/api/v1/stats` | Session-aggregate detection stats |
| POST/GET | `/api/v1/report` | Report a URL for review (POST for the dashboard form, GET for the extension's report link) |
| GET | `/api/v1/blacklist` | Get the current known-bad domain list |
