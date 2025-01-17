<div align="center">

# 🛡️ Spam Classifier

**AI-powered spam detection API with a modern web UI**

Built with Python · FastAPI · scikit-learn · Naive Bayes

[![Python](https://img.shields.io/badge/Python-3.8+-3776AB?logo=python&logoColor=white)](https://python.org)
[![FastAPI](https://img.shields.io/badge/FastAPI-0.100+-009688?logo=fastapi&logoColor=white)](https://fastapi.tiangolo.com)
[![scikit-learn](https://img.shields.io/badge/scikit--learn-ML-F7931E?logo=scikit-learn&logoColor=white)](https://scikit-learn.org)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

</div>

---

## 📋 Table of Contents

- [About](#about)
- [Features](#features)
- [Tech Stack](#tech-stack)
- [Project Structure](#project-structure)
- [Getting Started](#getting-started)
- [Usage](#usage)
- [API Reference](#api-reference)
- [How It Works](#how-it-works)
- [Dataset](#dataset)
- [Deployment](#deployment)
- [License](#license)
- [Author](#author)

---

## About

Spam Classifier is an intelligent spam detection system that classifies text messages (SMS, emails, notifications) as **spam** or **ham** (not spam) in real time. It combines a trained machine learning model with a clean REST API and a modern, responsive web UI — no scrolling needed.

---

## Features

- **Real-time classification** — instant spam/ham predictions via API or UI
- **Modern web interface** — dark-themed, two-column layout with dot-grid background, clickable examples, and color-coded results
- **REST API** — single `POST /predict` endpoint, easy to integrate anywhere
- **Text preprocessing** — input cleaning, noise stripping, and email part extraction
- **Pre-trained model** — train once with `trainModel.py`, serve forever
- **Auto-generated docs** — Swagger UI at `/docs`, ReDoc at `/redoc`

---

## Tech Stack

| Layer           | Technology                          |
| --------------- | ----------------------------------- |
| **Language**    | Python 3.8+                         |
| **Framework**   | FastAPI                             |
| **ML Model**    | Multinomial Naive Bayes (scikit-learn) |
| **Vectorizer**  | TF-IDF (scikit-learn)               |
| **Serialization** | joblib                            |
| **Server**      | Uvicorn                             |
| **Data**        | pandas                              |

---

## Project Structure

```
Spam-Classifier/
├── app/
│   ├── main.py                 # FastAPI app entry point + static file serving
│   ├── routes/
│   │   └── spam.py             # POST /predict endpoint
│   ├── model/
│   │   └── request.py          # Pydantic request schema
│   ├── services/
│   │   └── spam_detector.py    # ML prediction logic
│   ├── utils/
│   │   ├── model_loader.py     # Load vectorizer & model at startup
│   │   └── payload_cleaner.py  # Text cleaning & email parsing
│   ├── logger/
│   │   └── logger.py           # Logging utility
│   └── static/
│       └── index.html          # Web UI (single-page)
├── data/
│   └── spam_data.tsv           # Training dataset
├── model/
│   ├── vectorizer.pkl          # Saved TF-IDF vectorizer
│   └── spam_model.pkl          # Trained Naive Bayes model
├── trainModel.py               # Script to train & save the model
├── requirements.txt            # Python dependencies
├── render.yaml                 # Render deployment config
├── LICENSE                     # MIT License
└── README.md
```

---

## Getting Started

### Prerequisites

- Python 3.8 or newer
- pip

### Installation

```bash
# 1. Clone the repository
git clone https://github.com/jinishshah00/Spam-Classifier.git
cd Spam-Classifier

# 2. Create a virtual environment
python3 -m venv venv
source venv/bin/activate        # macOS/Linux
# venv\Scripts\activate         # Windows

# 3. Install dependencies
pip install -r requirements.txt

# 4. Train the model (generates .pkl files in model/)
python trainModel.py

# 5. Start the server
uvicorn app.main:app --reload
```

Open **http://127.0.0.1:8000** in your browser to use the web UI.

---

## Usage

### Web UI

Visit `http://127.0.0.1:8000` — paste or type a message, click **Analyze Message**, and get an instant color-coded result. You can also click one of the example chips to try it out quickly.

### cURL

```bash
curl -X POST http://127.0.0.1:8000/predict \
  -H "Content-Type: application/json" \
  -d '{"message": "Congratulations! You have won a free ticket!"}'
```

### Python (requests)

```python
import requests

response = requests.post(
    "http://127.0.0.1:8000/predict",
    json={"message": "Hey, are we still meeting for lunch tomorrow?"}
)
print(response.json())
```

---

## API Reference

### `POST /predict`

Classify a text message as spam or not spam.

**Request Body:**

```json
{
  "message": "Congratulations! You've won a free ticket!"
}
```

**Response (spam):**

```json
{
  "message": "congratulations youve won a free ticket",
  "spam": true,
  "parts": {
    "subject": null,
    "body": "congratulations youve won a free ticket"
  },
  "status": "success"
}
```

**Response (not spam):**

```json
{
  "message": "hey are we still meeting for lunch tomorrow",
  "spam": false,
  "parts": {
    "subject": null,
    "body": "hey are we still meeting for lunch tomorrow"
  },
  "status": "success"
}
```

| Status Code | Description                   |
| ----------- | ----------------------------- |
| `200`       | Prediction returned           |
| `422`       | Invalid input                 |
| `500`       | Internal server error         |

Interactive docs available at **`/docs`** (Swagger) and **`/redoc`** (ReDoc).

---

## How It Works

```
Input Message
     │
     ▼
┌─────────────────┐
│ Text Cleaning   │  Strip noise, normalize, extract email parts
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ TF-IDF Vectorizer │  Convert text → numerical feature vector
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Naive Bayes     │  Multinomial NB classifier predicts spam/ham
└────────┬────────┘
         │
         ▼
   ┌──────────┐
   │  Result  │  → { "spam": true/false }
   └──────────┘
```

1. **Text Preprocessing** — The input message is cleaned and normalized (lowercased, noise stripped, email parts extracted).
2. **TF-IDF Vectorization** — The cleaned text is converted into a numerical feature vector using Term Frequency–Inverse Document Frequency, which captures how important each word is relative to the training corpus.
3. **Multinomial Naive Bayes** — The classifier applies Bayes' theorem to compute the probability of the message being spam vs. ham, and returns the most likely class.
4. **Response** — A JSON result with the prediction, cleaned message, and extracted parts is returned instantly.

---

## Dataset

This project uses the [SMS Spam Collection Dataset](https://archive.ics.uci.edu/ml/datasets/sms+spam+collection) from the UCI Machine Learning Repository.

- **Format:** TSV (tab-separated values)
- **Labels:** `ham` (legitimate) / `spam`
- **Size:** 5,574 messages

Place the file as `spam_data.tsv` in the project root before training.

---

## Deployment

A `render.yaml` is included for deploying to [Render](https://render.com):

```bash
# Build
pip install -r requirements.txt

# Start
uvicorn app.main:app --host 0.0.0.0 --port 10000
```

---

## License

This project is licensed under the **MIT License** — see the [LICENSE](LICENSE) file for details.

---

## Author

**Jinish Shah**

- Email: [jinishshah00@gmail.com](mailto:jinishshah00@gmail.com)
- GitHub: [@jinishshah00](https://github.com/jinishshah00)

---

<div align="center">

Made with ☕ and Python

</div>
