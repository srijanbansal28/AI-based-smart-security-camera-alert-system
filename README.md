<<<<<<< HEAD
# 🔒 AI-Based Smart Camera Surveillance System

A production-ready, real-time surveillance platform that uses deep learning to detect threats (fire, weapons, violence, suspicious activity) from webcam or CCTV feeds — with instant email + WhatsApp alerts, and all data stored in MongoDB Atlas.

---

## 🏗 Architecture

```
┌─────────────────────────────────────────────────────┐
│                   Browser (React)                    │
│  Dashboard · Live Feed · Alerts · Incidents · RBAC  │
└──────────────────────┬──────────────────────────────┘
                       │ HTTP / Vite proxy
┌──────────────────────▼──────────────────────────────┐
│              Node.js API Server (Express)            │
│  Auth · Alerts · Analytics · GridFS image serving   │
└────────────┬──────────────────────┬─────────────────┘
             │ Mongoose             │ GridFSBucket
┌────────────▼──────────────────────▼─────────────────┐
│                  MongoDB Atlas                        │
│  alerts · users · cameras · alert_images (GridFS)   │
└─────────────────────────────────────────────────────┘
             ▲
             │ POST /api/alerts/ingest (base64 image)
┌────────────┴──────────────────────────────────────── ┐
│            Python AI Model (Flask + YOLOv8)          │
│  Fire · Weapon · Activity detection · ByteTrack     │
│  → Email alert (Gmail SMTP) + WhatsApp (Twilio)     │
└─────────────────────────────────────────────────────┘
```

---

## 📋 Prerequisites

| Tool | Version | Purpose |
|------|---------|---------|
| **Node.js** | 18+ | Frontend + API server |
| **npm** | 9+ | Package management |
| **Python** | 3.10+ | AI model |
| **pip3** | latest | Python packages |
| **MongoDB Atlas** | Free tier | Database + image storage |
| **Webcam / RTSP** | — | Video source |

> **GPU strongly recommended** for real-time inference. Tested on RTX 3060+ with CUDA 11.8/12.x.  
> CPU-only is possible but will be slow (~3–5 FPS).

---

## 🚀 Quick Start

### 1. Clone the repository

```bash
git clone https://github.com/<your-username>/<your-repo>.git
cd <your-repo>
```

---

### 2. Install Node.js dependencies

```bash
# Root (React frontend)
npm install

# Node.js API server
cd server && npm install && cd ..
```

---

### 3. Install Python dependencies

```bash
cd model
pip3 install -r requirements.txt

# For SMS/WhatsApp alerts (optional)
pip3 install twilio

# If using CUDA GPU (skip if CPU-only):
pip3 install torch torchvision --index-url https://download.pytorch.org/whl/cu121
cd ..
```

---

### 4. Configure environment variables

#### 4a — Server (`server/.env`)

```bash
cp server/.env.example server/.env
```

Edit `server/.env`:

```env
PORT=5050
MONGODB_URI=mongodb+srv://<user>:<password>@cluster0.mongodb.net/secureai?appName=Cluster0
JWT_SECRET=<generate: node -e "console.log(require('crypto').randomBytes(48).toString('hex'))">
MODEL_BASE_URL=http://localhost:5001
MODEL_API_KEY=secureai_model_ingest_key_2026
```

> Get your MongoDB URI from **MongoDB Atlas → Clusters → Connect → Drivers**.

#### 4b — AI Model (`model/.env`)

```bash
cp model/.env.example model/.env
```

Edit `model/.env`:

```env
# Node.js ingest (required — stores images in MongoDB)
NODE_SERVER_URL=http://localhost:5050
MODEL_API_KEY=secureai_model_ingest_key_2026

# Email alerts — Gmail App Password (NOT your regular password)
# Setup: myaccount.google.com → Security → App passwords
SENDER_EMAIL=you@gmail.com
EMAIL_PASSWORD=xxxx xxxx xxxx xxxx
RECIPIENT_EMAIL=admin@company.com,security@company.com

# WhatsApp alerts via Twilio (works internationally on free trial)
TWILIO_ACCOUNT_SID=ACxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
TWILIO_AUTH_TOKEN=your_auth_token
TWILIO_WHATSAPP_SANDBOX=whatsapp:+14155238886
WHATSAPP_RECIPIENT_NUMBERS=+919876543210

# SMS fallback (requires international-capable Twilio number)
TWILIO_PHONE_NUMBER=+1xxxxxxxxxx
SMS_RECIPIENT_NUMBERS=+919876543210
```

> **`MODEL_API_KEY` must match in both `.env` files exactly.**

---

### 5. Download AI model weights

The pre-trained YOLOv8 base model downloads automatically on first run.  
Custom-trained weights (`fire_detector.pt`, `weapon_detector.pt`, `activity_classifier.pt`) must be placed in `model/models/`:

```
model/
└── models/
    ├── fire_detector.pt        ← fire/smoke detection
    ├── weapon_detector.pt      ← gun/knife detection
    └── activity_classifier.pt  ← violence/suspicious activity
```

> If weights are missing, the system will **warn and skip** that detection class — other detectors still run.

---

### 6. Run the full system

```bash
# Start all three services simultaneously
npm start
```

This runs concurrently:
| Service | URL | Command |
|---------|-----|---------|
| React frontend | http://localhost:5173 | `vite` |
| Node.js API | http://localhost:5050 | `node server/server.js` |
| Python AI model | http://localhost:5001 | `python3 model/app.py` |

---

## 🖥 Running Services Individually

```bash
# Frontend only
npm run client

# API server only
npm run server

# AI model only
npm run model

# AI model with a video file instead of webcam
cd model && python3 app.py --source path/to/video.mp4

# AI model with RTSP stream
cd model && python3 app.py --source "rtsp://admin:password@192.168.1.100:554/stream"

# Dry run (no models loaded — test camera only)
cd model && python3 app.py --dry-run
```

---

## 📁 Project Structure

```
.
├── src/                        # React frontend (Vite)
│   ├── pages/
│   │   ├── Dashboard.jsx       # Live overview + recent alerts
│   │   ├── Alerts.jsx          # Alert list with frame review
│   │   ├── Incidents.jsx       # Resolved incident history
│   │   ├── LiveCameras.jsx     # MJPEG live feed
│   │   └── Analytics.jsx       # Charts and statistics
│   ├── services/api.js         # Axios API client
│   └── components/             # Shared UI components
│
├── server/                     # Node.js Express API
│   ├── server.js               # Entry point
│   ├── routes/
│   │   ├── alerts.js           # Alerts CRUD + GridFS image serving
│   │   ├── auth.js             # JWT authentication
│   │   ├── cameras.js          # Camera management
│   │   └── analytics.js        # Aggregation queries
│   ├── models/
│   │   ├── Alert.js            # Alert schema (with imageId → GridFS)
│   │   └── User.js             # User schema (RBAC roles)
│   ├── middleware/auth.js       # JWT protect + role authorize
│   ├── .env                    # ← create from .env.example (gitignored)
│   └── .env.example            # Safe template
│
├── model/                      # Python AI detection system
│   ├── app.py                  # Flask API server
│   ├── realtime_detection.py   # Detection pipeline
│   ├── configs/
│   │   └── detection_config.yaml  # Thresholds, alert config, contacts
│   ├── utils/
│   │   ├── alert.py            # Debounce + MongoDB ingest + notifications
│   │   ├── email_alert.py      # Gmail SMTP email sender
│   │   ├── sms_alert.py        # Twilio WhatsApp + SMS sender
│   │   ├── logger.py           # Rotating file + colored console logs
│   │   ├── tracker.py          # ByteTrack wrapper
│   │   └── visualizer.py       # Bounding box drawing
│   ├── models/                 # Trained weight files (gitignored)
│   ├── logs/                   # JSONL audit log + detections.log
│   ├── requirements.txt        # Python dependencies
│   ├── .env                    # ← create from .env.example (gitignored)
│   └── .env.example            # Safe template
│
├── package.json                # Root npm scripts (concurrently)
└── .gitignore
```

---

## 🔔 Alert & Notification System

When a threat is confirmed (3 consecutive frames + debounce):

1. **Terminal** — color-coded `⚠ ALERT!` banner with threat type, confidence, and emergency contacts
2. **MongoDB** — alert document + JPEG screenshot stored in GridFS (no local files)
3. **Email** — HTML email with screenshot attached, sent to all `RECIPIENT_EMAIL` addresses
4. **WhatsApp** — message via Twilio WhatsApp Sandbox (works internationally on free trial)

### Configure emergency contacts

Edit `model/configs/detection_config.yaml`:

```yaml
alerts:
  auto_alerts_enabled: true          # set false to mute all emails/SMS
  email:
    target_threats: ["FIRE", "SMOKE", "PISTOL", "GUN", "KNIFE"]

  police_contact:
    station_name: "Your Local Police Station"
    phone: "your-number"
    emergency: "112"
    address: "Station address"

  fire_department:
    station_name: "Your Fire Station"
    phone: "your-number"
    emergency: "101"
    address: "Station address"
```

### WhatsApp Sandbox setup (one-time per recipient)

1. Each recipient opens WhatsApp and messages **`+1 415 523 8886`**
2. Sends exactly: `join <your-sandbox-keyword>`  
   *(Find keyword: Twilio Console → Messaging → Try it out → Send a WhatsApp message)*
3. They receive a confirmation — done ✅

---

## 👥 User Roles (RBAC)

| Role | Permissions |
|------|------------|
| **Admin** | Full access — manage users, cameras, resolve alerts, view all data |
| **Operator** | Resolve alerts, view live feed, view incidents |
| **Analyst** | View alerts and analytics (read-only) |
| **Viewer** | Dashboard and live feed only |

Register the first user, then promote to Admin via MongoDB:
```js
// In MongoDB Atlas → Collections → users
db.users.updateOne({ email: "admin@you.com" }, { $set: { role: "admin" } })
```

---

## ⚙️ Configuration Reference

### `model/configs/detection_config.yaml`

| Key | Default | Description |
|-----|---------|-------------|
| `camera.source` | `0` | `0` = webcam, or RTSP URL |
| `alerts.debounce_seconds` | `3.0` | Min seconds between repeat alerts |
| `alerts.consecutive_frames_required` | `2` | Frames before alert fires |
| `alerts.auto_alerts_enabled` | `true` | Toggle email/WhatsApp globally |
| `models.fire.conf_threshold` | `0.25` | Lower = more sensitive |
| `models.weapon.conf_threshold` | `0.30` | Lower = more sensitive |

---

## 🛠 Troubleshooting

| Problem | Fix |
|---------|-----|
| `MongoDB connection error: uri is undefined` | Check `server/.env` has `MONGODB_URI` set |
| `MODEL_API_KEY not set — skipping ingest` | Verify `MODEL_API_KEY` matches in both `.env` files |
| Images show "Uploading…" badge forever | Model API key mismatch or Node.js server not running |
| Email not sending | Use a Gmail **App Password** (16 chars), not your regular password |
| WhatsApp not received | Recipient hasn't joined the sandbox yet (see setup above) |
| SMS fails for +91 numbers | Free Twilio numbers are US-only — use WhatsApp instead |
| `Cannot open video source: 0` | Try `--source 1` or check camera permissions |
| `Fire weights not found` | Place `fire_detector.pt` in `model/models/` |
| CUDA out of memory | Add `--batch 4` or reduce `img_size` in config |

---

## 📦 Key Dependencies

### Python (model)
- `ultralytics` — YOLOv8 object detection + ByteTrack
- `torch` / `torchvision` — PyTorch deep learning
- `opencv-python` — video capture and frame processing
- `flask` / `flask-cors` — model API server
- `python-dotenv` — `.env` loading
- `twilio` — WhatsApp + SMS alerts
- `colorama` — colored terminal output

### Node.js (server)
- `express` — REST API framework
- `mongoose` — MongoDB ODM
- `mongodb` — native driver (GridFS image storage)
- `jsonwebtoken` — JWT authentication
- `bcryptjs` — password hashing
- `cors` — cross-origin requests
- `dotenv` — environment variable loading

### Frontend (React)
- `react` + `react-router-dom` — SPA routing
- `axios` — HTTP client
- `chart.js` + `react-chartjs-2` — analytics charts
- `lucide-react` — icon library
- `vite` — build tool + dev server

---

## 🔐 Security Notes

- **Never commit `.env` files** — both `server/.env` and `model/.env` are gitignored
- Use `.env.example` files as templates when sharing the repo
- Rotate credentials if they are ever exposed in chat/commits
- MongoDB Atlas: restrict IP access to known IPs in the Atlas Network Access settings
- JWT secret should be at least 48 random bytes

---

## 📄 License

MIT — see [LICENSE](LICENSE) for details.
=======
# AI-based-smart-security-camera-alert-system
The AI-Based Smart Camera Surveillance System is an intelligent, real-time security monitoring platform designed to detect potential threats and automate emergency responses using Artificial Intelligence and Deep Learning. The system analyzes live video streams from webcams and CCTV cameras to identify suspicious activities, fire, smoke, weapons.
>>>>>>> 8040469114094dd4486057f2cd41058ed444aa87
