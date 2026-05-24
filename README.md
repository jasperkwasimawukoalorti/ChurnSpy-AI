# ChurnSpy AI · Churn Predictor

> **A live, AI-powered web app that predicts telecom customer churn in real time — backed by a production ML model on Google Cloud Run.**

🔗 **Live demo:** [https://jasperkwasimawukoalorti.github.io/ChurnSpy-AI/](https://jasperkwasimawukoalorti.github.io/ChurnSpy-AI/)

---

## What Is ChurnSpy AI?

ChurnSpy AI is a front-to-back machine learning project built on real telecom customer data. It predicts whether a customer is likely to churn (cancel their service) based on their account profile, contract type, usage behaviour, and service subscriptions.

The web app is a single-page interface that:
- Walks a user through **4 input steps** to capture customer details
- Calls a **live REST API** hosted on Google Cloud Run
- Returns an instant **churn probability, risk level, and recommended action**
- Displays **live MLflow model registry metadata** for the current champion model

---

## Project Architecture

```
User (Browser)
      │
      ▼
GitHub Pages — index.html
  (Static frontend — no server needed)
      │
      │  POST /predict
      │  GET  /risk-segment
      │  GET  /model-info
      │  GET  /health
      ▼
Google Cloud Run API
  https://churn-api-315673521215.us-central1.run.app
      │
      ▼
MLflow Model Registry (GCS)
  Experiment : project-b-group-b1
  Model name : group-b1-model
  Stage      : Champion
```

---

## How the Prediction Works

### Input — 4-Step Form

| Step | Fields collected |
|------|-----------------|
| 1 · Account basics | Tenure (months), Monthly charges, Total charges |
| 2 · Contract & billing | Contract type, Payment method, Paperless billing |
| 3 · Services | Internet service, Tech support, Online security, and optional add-ons |
| 4 · Customer profile | Gender, Senior citizen, Partner, Dependents |

### Output — Result Card

The API returns a `churn_probability` (0.0 – 1.0). The frontend interprets it as follows:

| Probability | Verdict | Risk level | Recommended action |
|---|---|---|---|
| ≥ 0.65 | ⚠️ High churn risk | High | Immediate retention intervention |
| 0.32 – 0.64 | 📊 Moderate risk | Medium | Monitor and consider soft retention offer |
| < 0.32 | ✅ Likely to stay | Low | No action required |

> **Decision threshold: 0.32** — this is the optimised threshold from training, chosen to maximise F1 score on the validation set. The `churn_prediction` tag (Churn / No Churn) reflects the model's binary decision at this threshold.

### Result Metadata Tags

Each prediction result shows four tags:

- `Risk: Low / Medium / High` — probability bucket from the `/risk-segment` endpoint
- `Prediction: Churn / No Churn` — binary model decision at threshold 0.32
- `Model vN` — version number of the current champion model in MLflow
- `ID: xxxxxxxx…` — unique prediction request ID for traceability

---

## API Endpoints

All requests go to:
```
https://churn-api-315673521215.us-central1.run.app
```

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/predict` | Submit customer features, receive churn probability and prediction |
| `GET` | `/risk-segment?churn_probability=0.XX` | Returns risk bucket and recommended action string |
| `GET` | `/model-info` | Returns live champion model metadata from MLflow registry |
| `GET` | `/health` | API health check — used by the status pill in the header |

---

## Model Details

The champion model is promoted through a **quad-gate evaluation system** — it must pass all four gates before it can replace the current champion in the MLflow registry:

| Gate | Metric | Minimum threshold |
|------|--------|-------------------|
| Gate 1 | ROC-AUC | ≥ 0.75 (+0.01 gain over current champion) |
| Gate 2 | F1 Score | ≥ 0.55 (+0.01 gain over current champion) |
| Gate 3 | PR-AUC | ≥ 0.50 (+0.01 gain over current champion) |
| Gate 4 | Business Profit | Positive expected profit vs. current champion |

### Why PR-AUC?

The dataset has a **~26% churn rate** — meaning it is class-imbalanced. PR-AUC (Precision-Recall Area Under Curve) is a more honest metric than ROC-AUC for imbalanced datasets, because it directly measures performance on the minority class (churners) without being inflated by the large number of true negatives.

### Training Pipeline Summary

- **Balancing:** SMOTETomek (oversamples churners, removes borderline noise)
- **Tuning:** RandomizedSearchCV with 5-fold cross-validation, scoring on F1
- **Architecture:** Stacking ensemble — XGBoost + Random Forest + Logistic Regression base learners, with a calibrated Logistic Regression meta-learner
- **Threshold optimisation:** Decision boundary tuned to 0.32 on the validation set
- **Tracking:** All experiments logged in MLflow under `project-b-group-b1`

---

## Repository Structure

```
ChurnSpy-AI/
│
├── index.html          # Full single-page app (HTML + CSS + JS)
└── README.md           # This file
```

The frontend is intentionally a single self-contained file — no build step, no dependencies, no Node.js. GitHub Pages serves it directly.

---

## Running Locally

No build step required. Just open the file:

```bash
# Option 1 — open directly in browser
open index.html

# Option 2 — serve locally (avoids any CORS quirks)
python -m http.server 8000
# then visit http://localhost:8000
```

> The app calls the Cloud Run API directly from the browser. As long as the API is running, predictions work the same locally as on GitHub Pages.

---

## API Status

The header pill shows live API status on page load:

- 🟢 **API live** — Cloud Run is reachable and healthy
- ⚫ **offline** — API is cold-starting or unreachable (Cloud Run scales to zero when idle — first request after inactivity may take ~3–5 seconds)

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Frontend | Vanilla HTML / CSS / JavaScript |
| Fonts | Fraunces (serif) + Instrument Sans — via Google Fonts |
| Hosting | GitHub Pages |
| API | FastAPI on Google Cloud Run |
| ML framework | scikit-learn, XGBoost, imbalanced-learn |
| Experiment tracking | MLflow + GCS artifact storage |
| Containerisation | Docker |

---

## Changelog

### Latest — threshold & label fixes
- Decision threshold corrected from `0.35` → `0.32` to match trained model
- Prediction label corrected from `Retain` → `No Churn` for consistency
- Risk gauge thumb capped at `96%` to prevent overflow at high probabilities

---

## Author

**Jasper Kwasi Mawuko Alorti**
Built as part of a group ML project — telco churn prediction, end to end.

🔗 [Live App](https://jasperkwasimawukoalorti.github.io/ChurnSpy-AI/)
