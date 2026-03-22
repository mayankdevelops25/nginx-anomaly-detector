# nginx-anamoly-detecter

A real-time NGINX anomaly detection and alert system built with FastAPI, scikit-learn, and Python.

## What it does

nginx-anamoly-detecter watches your NGINX access logs in real time. Every incoming log line is parsed, feature-extracted, and sent to a FastAPI backend running an Isolation Forest ML model. If an anomaly is detected, it sends an email alert with details about the suspicious request.

## Tech Stack

- **FastAPI** — prediction API server
- **Isolation Forest** — unsupervised ML model for anomaly detection
- **Python** — log watcher, feature extraction, email alerts
- **NGINX** — log source

## Prerequisites

- Ubuntu / WSL 2 (Windows users must use WSL)
- NGINX installed and running
- Python 3.8+
- Git
- A Gmail account with 2FA enabled (for email alerts)

## Setup

### 1. Install NGINX

```bash
sudo apt install nginx
sudo service nginx start
```

Verify it's running by visiting `http://localhost` in your browser.

### 2. Clone the repository

```bash
cd ~
git clone https://github.com/YOUR_USERNAME/nginx-anamoly-detecter.git
cd nginx-anamoly-detecter
```

### 3. Create and activate a virtual environment

```bash
python3 -m venv venv
source venv/bin/activate
```

### 4. Install dependencies

```bash
pip install -r requirements.txt --timeout 120
```

### 5. Set up Gmail App Password

1. Go to [myaccount.google.com](https://myaccount.google.com)
2. Enable 2-Step Verification under Security
3. Search for "App passwords" and generate one
4. Copy the 16-character password

### 6. Create the .env file

Copy the template and fill in your details:

```bash
cp .env.example .env
nano .env
```

```
PREDICT_ONE_URL = "http://127.0.0.1:8000/predict_one"
NOGINX_MAIL_ID = "your-gmail@gmail.com"
NOGINX_MAIL_PASSWORD = "xxxx xxxx xxxx xxxx"
RECEIVER_MAIL_ID = "receiver@gmail.com"
```

## Running the app

You need two terminals open, both with the venv activated (`source venv/bin/activate`).

**Terminal 1 — Start the FastAPI backend:**

```bash
cd fastapi_backend
uvicorn api_server:app --reload
```

**Terminal 2 — Start the log watcher:**

```bash
cd watcher
python3 watcher.py
```

Generate some NGINX traffic to test:

```bash
for i in {1..20}; do curl -s http://localhost > /dev/null; done
```

When an anomaly is detected, an email alert will be sent to your configured receiver address containing the IP, timestamp, path, and user agent of the suspicious request.

## On detecting an anomaly

- An email is sent to the configured receiver with full details of the anomalous request
- Desktop notifications are supported on Linux (not available in WSL)

## ML Model

- Trained using the **Isolation Forest** algorithm
- Trained on a publicly available NGINX logs dataset
- Pre-trained model stored at `data/model.pkl`

## API Endpoints

**POST /predict** — batch prediction

```json
{
  "features": [{
    "status": 304,
    "size": 0,
    "method": 1,
    "path": 0,
    "user_agent": 54,
    "hour_of_day": 8
  }]
}
```

**POST /predict_one** — single log prediction

```json
{
  "feature": {
    "status": 304,
    "size": 0,
    "method": 1,
    "path": 0,
    "user_agent": 54,
    "hour_of_day": 8
  }
}
```

Response:

```json
[{ "anomaly": false }]
```

## Training on custom data

For company-specific use cases, you can retrain the model on your own NGINX logs.

1. Store your NGINX logs at `data/nginx.log`
2. Parse the logs:
```bash
python3 parse_logs.py
```
3. Extract features:
```bash
python3 process_csv.py
```
4. Train the model:
```bash
python3 train_model.py
```

You can customise the model through feature engineering or by tweaking the Isolation Forest parameters.

## Notes for Windows users

Run everything inside **WSL 2** with Ubuntu. Do not use PowerShell or Command Prompt. Clone the repo into the Linux home directory (`~`) not the Windows filesystem (`/mnt/c/...`) to avoid permission and performance issues.
