# PhishGuard AI — Advanced URL Threat Detector

A full-featured Flask web application with User & Admin modules, model selection, and a 3D animated UI for detecting phishing and malicious URLs using 3 trained AI models.

---

## Features

### User Module
- Register / Login / Logout
- **URL Scanner** — choose from XGBoost, Gradient Boosting, Neural Network, or All Models at once
- Confidence scores and per-model breakdown
- Complete scan history with delete option
- Personal dashboard with threat statistics

### Admin Module
- System-wide dashboard (total users, scans, threat rate)
- **User Management** — activate/deactivate, promote to admin, delete users
- **All Scans** — filterable by model and threat status
- Model usage statistics

### 3D Animated UI
- Live Three.js 3D particle field + rotating geometric shapes
- Glassmorphism card design
- Animated counters, confidence bars, hover effects
- Responsive dark-mode design

---

## Setup Instructions

### Requirements
- Python **3.9 – 3.11** (recommended: 3.11)
- pip

### Step 1 — Extract & Navigate
```bash
cd phishing_detector
```

### Step 2 — Create Virtual Environment
```bash
python3 -m venv venv
```

Activate:
- **macOS / Linux:** `source venv/bin/activate`
- **Windows:**       `venv\Scripts\activate`

### Step 3 — Install Dependencies
```bash
pip install -r requirements.txt
```
> First install may take 5–10 minutes (TensorFlow + XGBoost are large).

### Step 4 — Run the Application
```bash
python app.py
```

You will see:
```
[INFO] Default admin created: admin / admin123
 * Running on http://0.0.0.0:5000
```

### Step 5 — Open in Browser
**http://localhost:5000**

---

## Default Login Credentials

| Role  | Username | Password  |
|-------|----------|-----------|
| Admin | `admin`  | `admin123` |

You can register new user accounts from the **Register** page.

---

## Project Structure

```
phishing_detector/
├── app.py                  # Flask application factory + startup
├── config.py               # Configuration (secret key, DB URL)
├── extensions.py           # Flask extensions (SQLAlchemy, Login, Bcrypt)
├── models.py               # Database models: User, ScanHistory
├── auth.py                 # Auth blueprint: /login, /register, /logout
├── user_routes.py          # User blueprint: /dashboard, /scan, /result, /history
├── admin_routes.py         # Admin blueprint: /admin, /admin/users, /admin/scans
├── feature_extractor.py    # URL feature extraction (7, 30, 16 features)
├── models_loader.py        # ML model loading + prediction wrappers
├── requirements.txt        # Python dependencies
├── models/
│   ├── phishing_classifier.pkl   # XGBoost (7 features)
│   ├── newmodel.pkl              # GradientBoosting (30 features)
│   └── Malicious_URL_Prediction.h5  # Neural Network (16 features)
├── templates/
│   ├── base.html           # Shared layout: navbar, 3D canvas, flash msgs
│   ├── auth/               # Login, Register pages
│   ├── user/               # Dashboard, Scan, Result, History pages
│   ├── admin/              # Admin Dashboard, Users, Scans pages
│   └── errors/             # 403, 404 pages
└── static/
    ├── css/style.css       # Complete dark 3D glassmorphism stylesheet
    └── js/
        ├── bg3d.js         # Three.js 3D animated background
        └── app.js          # Counter animations, bar animations, UI helpers
```

---

## Model Details

| Model | Algorithm | Features | Target | Notes |
|-------|-----------|----------|--------|-------|
| `phishing_classifier.pkl` | XGBoost | 7 URL structural | 0=Legit, 1=Phishing | Requires `xgboost` |
| `newmodel.pkl` | Gradient Boosting | 30 domain-level | 1=Legit, -1=Phishing | Requires `scikit-learn==1.3.1` exactly |
| `Malicious_URL_Prediction.h5` | Keras Neural Net (4-layer) | 16 statistical | sigmoid > 0.5 = malicious | Requires `tensorflow` |

---

## REST-style API

The scan form submits via POST. You can also test via curl:

```bash
# Login first in the browser, then use session cookie
curl -X POST http://localhost:5000/scan \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "url=http://example.com&model=xgb"
```

---

## Troubleshooting

| Problem | Fix |
|---------|-----|
| `ModuleNotFoundError: sklearn` | `pip install scikit-learn==1.3.1` |
| `ModuleNotFoundError: xgboost` | `pip install xgboost` |
| `ModuleNotFoundError: tensorflow` | `pip install tensorflow` |
| Port 5000 in use | `PORT=8080 python app.py` |
| TF GPU warnings | Safe to ignore, runs on CPU |
| Database errors | Delete `instance/phishing_app.db` and restart |

---

## Changing the Admin Password

Log in as admin → change via your code or database. To reset:
```bash
python3 -c "
from app import create_app; from extensions import db, bcrypt; from models import User
app = create_app()
with app.app_context():
    u = User.query.filter_by(username='admin').first()
    u.password_hash = bcrypt.generate_password_hash('newpassword').decode()
    db.session.commit()
    print('Done')
"
```
