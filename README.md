# ChurnSpy AI — Frontend (`index.html`)

**Group B1 · ML Lead: Jasper Kwasi Mawuko Alorti (AJKM) · Cloud Lead: Kobia Williams (KW)**

---

## Overview

`index.html` is the single-page frontend for **ChurnSpy AI**, a live customer churn prediction tool. It is hosted on **GitHub Pages** and communicates directly with the Cloud Run API backend.

**Live URL:**
```
https://jasperkwasimawukoalorti.github.io/ChurnSpy-AI/
```

The page is fully self-contained — one HTML file with embedded CSS and JavaScript, no build step, no dependencies beyond Google Fonts.

---

## Features

- **4-step prediction form** collecting 19 customer attributes
- **Live API status pill** with auto-retry (3 attempts with backoff)
- **Animated result card** with probability counter and risk gauge
- **3-tier verdict display** (Likely to stay / Moderate risk / High churn risk)
- **Model Info tab** showing live MLflow champion model metadata
- **Responsive layout** — works on mobile and desktop

---

## Form Steps

### Step 1 — Account basics
| Field | Type | Constraints |
|---|---|---|
| Tenure | Number | 0–72 months (integer) |
| Monthly charges | Number | > 0 |
| Total charges | Number | ≥ 0 |

> **Important:** `tenure` must be a whole number. Fractional values (e.g. `0.1`) are accepted by the HTML form but will be truncated to `0` by the API's integer schema, which distorts `avg_monthly_from_total` and several engineered features.

### Step 2 — Contract & billing
| Field | Options |
|---|---|
| Contract type | Month-to-month · One year · Two year |
| Payment method | Electronic check · Mailed check · Bank transfer (automatic) · Credit card (automatic) |
| Paperless billing | Yes · No |

### Step 3 — Services used
| Field | Options |
|---|---|
| Internet service | Fiber optic · DSL · No |
| Tech support | Yes · No · No internet service |
| Online security | Yes · No · No internet service |
| Online backup *(optional)* | Yes · No · No internet service |
| Device protection *(optional)* | Yes · No · No internet service |
| Phone service *(optional)* | Yes · No |
| Multiple lines *(optional)* | Yes · No · No phone service |
| Streaming TV *(optional)* | Yes · No · No internet service |
| Streaming movies *(optional)* | Yes · No · No internet service |

The optional services are collapsed under **"More services (optional)"** and expand on tap.

### Step 4 — About the customer
| Field | Options |
|---|---|
| Gender | Male · Female |
| Senior citizen | No · Yes |
| Has a partner | Yes · No |
| Has dependents | Yes · No |

---

## Result Card

After prediction, the result card shows:

| Element | Description |
|---|---|
| Icon + verdict | ✅ Likely to stay / 📊 Moderate risk / ⚠️ High churn risk |
| Churn probability | Animated counter (0% → actual %) |
| Risk gauge | Sliding thumb on green→red track, capped at 96% |
| Recommended action | From `/risk-segment` API response |
| Risk badge | `Risk: Low / Medium / High` from `/risk-segment` |
| Prediction label | `Prediction: Churn` or `Prediction: No Churn` |
| Model version | `Model v17` |
| Prediction ID | First 8 chars of UUID from `/predict` |

---

## Verdict Thresholds

The frontend applies these display bands to `churn_probability`:

| Probability | Verdict | Colour | Icon |
|---|---|---|---|
| < 0.31 | Likely to stay | Green (`--success`) | ✅ |
| 0.32 – 0.45 | Moderate risk | Orange (`--warn`) | 📊 |
| ≥ 0.46 | High churn risk | Red (`--danger`) | ⚠️ |

> **Note:** These thresholds differ from the API's `/risk-segment` endpoint (which uses ≤ 0.29 / ≤ 0.69). This means the `Risk:` badge in the meta-chips (from the API) may show **Medium** while the card header shows **High churn risk**. This is a known inconsistency — aligning them requires updating the `/risk-segment` thresholds in `main.py`.

---

## API Integration

The frontend calls three API endpoints:

| Endpoint | When | Purpose |
|---|---|---|
| `GET /health` | On page load | API status pill (live/offline) |
| `POST /predict` | On "Predict →" | Get churn probability + prediction |
| `GET /risk-segment?churn_probability=X` | After predict | Get risk tier + recommended action |
| `GET /model-info` | When Model Info tab opens | Champion model metadata |

**API base URL** (set as `const BASE`):
```javascript
const BASE = 'https://churn-api-315673521215.us-central1.run.app';
```

---

## State Management

All form state is held in a single JavaScript object `s`:

```javascript
const s = {
  tenure, monthly_charges, total_charges,
  contract, payment_method, paperless_billing,
  internet_service, tech_support, online_security,
  online_backup, device_protection,
  phone_service, multiple_lines,
  streaming_tv, streaming_movies,
  gender, senior_citizen, partner, dependents
}
```

This object is serialised directly to JSON and sent as the `/predict` request body. The API field names match the Pydantic schema in `main.py`.

---

## Default Values

Fields not explicitly selected by the user default to:

| Field | Default |
|---|---|
| `paperless_billing` | `"Yes"` |
| `online_backup` | `"No"` |
| `device_protection` | `"No"` |
| `phone_service` | `"Yes"` |
| `multiple_lines` | `"No"` |
| `streaming_tv` | `"No"` |
| `streaming_movies` | `"No"` |
| `gender` | `"Male"` |
| `senior_citizen` | `0` |
| `partner` | `"No"` |
| `dependents` | `"No"` |

---

## API Health Pill

The pill in the top-right corner shows live API status:

- 🟢 **API live** — health check returned 200
- ⚪ **waking up… (1/3)** — retrying with backoff (3s, 6s, 10s)
- ⚪ **offline — tap to retry** — all retries exhausted

The pill is also tappable to manually trigger a re-check. `modelLoaded` flag resets on failure so the Model Info tab retries automatically on next open.

---

## Design System

Built with CSS custom properties:

| Variable | Value | Usage |
|---|---|---|
| `--bg` | `#f5f4f0` | Page background |
| `--white` | `#fff` | Card background |
| `--ink` | `#111010` | Primary text |
| `--muted` | `#888580` | Secondary text, labels |
| `--accent` | `#3d5afe` | Brand blue, active states |
| `--danger` | `#e53935` | High churn risk |
| `--warn` | `#f57c00` | Moderate risk |
| `--success` | `#00897b` | Low risk / likely to stay |

**Fonts:**
- `Fraunces` (serif) — headings, large numbers
- `Instrument Sans` (sans-serif) — body, labels, buttons

---

## Deployment

The file is deployed via **GitHub Pages** from the repo root. No build step required — push `index.html` to the `main` branch and it is live within seconds.

**Repo:** `https://github.com/jasperkwasimawukoalorti/ChurnSpy-AI`

---

## Changelog

| Date | Change |
|---|---|
| 2026-05-25 | Fixed `modelLoaded` flag — now resets on failure so Model Info tab retries |
| 2026-05-25 | Corrected decision threshold display from 0.35 → 0.32 |
| 2026-05-25 | Corrected "No Churn" label (was incorrectly "Retain") |
| 2026-05-25 | Adjusted High risk display threshold from 0.65 → 0.45 to match model's actual probability range |
| 2026-05-25 | Risk gauge capped at 96% to prevent thumb overflowing track |
