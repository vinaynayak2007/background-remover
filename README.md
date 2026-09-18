# 🖼️ Background Remover Service

A production-ready **AI background removal API** — upload an image, get a clean cut-out back in seconds. Built with **Flask** + **rembg** (U²-Net under the hood), Dockerized, and deployed on Render with a frontend that can live on any host.

<p align="left">
  <img src="https://img.shields.io/badge/Python-3.11-3776AB?style=flat-square&logo=python&logoColor=white" />
  <img src="https://img.shields.io/badge/Flask-3.0-000000?style=flat-square&logo=flask&logoColor=white" />
  <img src="https://img.shields.io/badge/rembg-2.0-FF6F00?style=flat-square" />
  <img src="https://img.shields.io/badge/Docker-ready-2496ED?style=flat-square&logo=docker&logoColor=white" />
  <img src="https://img.shields.io/badge/Deploy-Render-46E3B7?style=flat-square&logo=render&logoColor=black" />
</p>

---

## ✨ Features

- 🧠 **AI-powered** subject detection and background removal (no green screen needed)
- 🔌 **Simple REST API** — send a base64 image, receive a base64 PNG with transparency
- 🌐 **CORS-enabled** — call it from any frontend, any host
- 🐳 **Docker support** — reproducible builds, no "works on my machine"
- ⚡ **Production server** — Gunicorn with a tuned timeout for heavy inference
- 📦 **Size-guarded** — request payloads capped to keep the service responsive

---

## 🏗️ Architecture

```
┌──────────────────┐   base64 image    ┌──────────────────────┐
│   Frontend       │ ────────────────► │   Flask API          │
│  (any host)      │                   │   + rembg (ONNX)     │
│                  │ ◄──────────────── │   Gunicorn           │
└──────────────────┘  transparent PNG  └──────────────────────┘
                                              │
                                        Docker / Render
```

The frontend and backend are fully decoupled — you can host the UI on GoDaddy, Cloudflare Pages, or serve it straight from the API.

---

## 🚀 Quick Start

### Local

```bash
git clone https://github.com/vinaynayak2007/background-remover.git
cd background-remover

python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate

pip install -r requirements.txt
python app.py
```

The API starts on `http://localhost:10000` (override with the `PORT` env var).

### Docker

```bash
docker build -t background-remover .
docker run -p 10000:10000 -e PORT=10000 background-remover
```

---

## 📡 API

### `GET /` — Health check

```json
{ "status": "ok" }
```

### `POST /remove` — Remove background

**Request**

```json
{
  "image": "<base64-encoded-image-string>"
}
```

**Response**

```json
{
  "success": true,
  "image": "<base64-encoded-png-with-transparency>"
}
```

**Example**

```bash
curl -X POST http://localhost:10000/remove \
  -H "Content-Type: application/json" \
  -d "{\"image\": \"$(base64 -w0 photo.jpg)\"}"
```

```javascript
const res = await fetch("https://your-service.onrender.com/remove", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ image: base64String }),
});
const { image } = await res.json();
document.querySelector("img").src = `data:image/png;base64,${image}`;
```

---

## ☁️ Deployment

### Render (recommended)

| Setting | Value |
| :--- | :--- |
| Environment | `Python 3` (or `Docker`) |
| Build Command | `pip install -r requirements.txt` |
| Start Command | `gunicorn --bind 0.0.0.0:$PORT --workers 1 --timeout 120 app:app` |
| Python Version | `3.11.7` (pinned via `runtime.txt`) |

> **Note:** `rembg` is happiest on Python 3.11. Newer Python versions can break the ONNX runtime — `runtime.txt` pins this for you. If the build still misbehaves, deploy the Docker image instead.

---

## 📁 Project Structure

```
background-remover/
├── app.py                 # Flask API + rembg inference
├── server.py              # Standalone server entrypoint
├── wsgi.py                # WSGI entrypoint for Gunicorn
├── gunicorn.conf.py       # Worker/timeout tuning
├── Dockerfile             # Container build
├── requirements.txt       # Pinned dependencies
├── runtime.txt            # Python version pin
├── render.yaml            # Render blueprint
└── Procfile               # Process definition
```

---

## 🧯 Troubleshooting

| Problem | Fix |
| :--- | :--- |
| Port binding error on deploy | Bind to `0.0.0.0`, not `127.0.0.1` |
| Build fails on rembg/onnxruntime | Confirm Python 3.11 via `runtime.txt`, or use Docker |
| Request times out | First inference downloads model weights (~176 MB) — warm it up, or raise Gunicorn `--timeout` |
| `413 Payload Too Large` | Images are capped at 3 MB; compress before sending |

---

## 📄 License

MIT — use it, fork it, ship it.

---

<p align="center"><sub>Built by <a href="https://github.com/vinaynayak2007">Vinay N</a> · <a href="https://vinunayak.pages.dev">vinunayak.pages.dev</a></sub></p>
