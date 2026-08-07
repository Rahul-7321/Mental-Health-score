# Mental Health Signal — Student Wellness Analytics

Predicting a student's **mental health score (0–10)** from their social media habits, study time, sleep, physical activity, and stress level — powered by a Random Forest regression model, served through a FastAPI backend, and presented in a custom web UI.

> ⚠️ **Disclaimer:** This is an informational/educational project, not a clinical or diagnostic tool. If you or someone you know is struggling, please reach out to a mental health professional or a trusted person.

---

## ✨ Features

- **ML pipeline** (`ML_Project.ipynb`) — full workflow from EDA to a tuned regression model, including skew correction, ordinal/one-hot encoding, and hyperparameter search.
- **FastAPI backend** (`app.py`) — a `/predict` REST endpoint that validates input with Pydantic and returns a predicted score.
- **Frontend** (`index.html`, `style.css`, `script.js`) — a form-driven UI with live validation, an animated gauge, and clear result/error states.

---

## 📁 Project Structure

```
Mental Health/
├── ML_Project.ipynb                                    # Data exploration, preprocessing, model training
├── Mental_Health_Model.pkl                              # Saved scikit-learn pipeline (preprocessing + model)
├── Student Social Media And Mental Health Impact.csv     # Dataset (5,000 students, 13 columns)
├── app.py                                                # FastAPI backend serving the model
├── index.html                                            # Frontend markup
├── style.css                                             # Frontend styling
├── script.js                                             # Frontend logic (form handling, API calls)
└── README.md
```

---

## 🧠 The Model

**Problem type:** Regression — predicting `Mental_Health_Score`, a continuous value roughly between 3 and 10.

**Dataset:** 5,000 student records covering demographics (Age, Gender, Country), platform usage (Avg. Daily Usage Hours, Daily Unlocks, Most Used Platform), lifestyle (Study Hours, Sleep Hours, Physical Activity Hours), and Stress Level.

**Preprocessing pipeline (`ColumnTransformer`):**
| Feature type | Treatment |
|---|---|
| Skewed numeric (`Study_Hours`) | Impute → `log1p` transform → scale |
| Other numeric | Impute → scale |
| `Stress_Level` | Impute → `OrdinalEncoder` (Low < Medium < High < Very High) |
| Nominal categoricals (Gender, Academic Level, Platform, Purpose, Country group) | Impute → One-Hot Encoding |

**Models compared:**

| Model | Training R² | Testing R² | MAE | RMSE |
|---|---|---|---|---|
| Linear Regression | 0.72 | 0.74 | 0.54 | 0.68 |
| **Random Forest (default)** | **0.98** | **0.88** | **0.35** | **0.46** |
| Random Forest (tuned) | 0.85 | 0.82 | 0.35 | 0.56 |

The **default Random Forest pipeline** was selected and saved to `Mental_Health_Model.pkl` via `joblib`, as it gave the best test-set performance.

---

## 🚀 Getting Started

### 1. Prerequisites
- Python 3.10+
- pip

### 2. Install dependencies

```bash
pip install fastapi uvicorn pandas scikit-learn joblib pydantic
```

### 3. Run the backend

```bash
uvicorn app:app --port 8000 --reload
```

This starts the API at `http://127.0.0.1:8000`. Visit `http://127.0.0.1:8000/docs` for interactive Swagger docs.

### 4. Run the frontend

Open `index.html` directly in a browser, or serve it locally, e.g.:

```bash
python -m http.server 5500
```

Then visit `http://127.0.0.1:5500`. The frontend calls the API at `http://127.0.0.1:8000` (configured in `script.js` via `API_BASE`) — make sure the backend is running first.

---

## 🔌 API Reference

### `GET /`
Health check.

```json
{ "message": "Welcome to my web" }
```

### `POST /predict`

**Request body:**

```json
{
  "Age": 21,
  "Gender": "Female",
  "Country": "India",
  "Academic_Level": "Undergraduate",
  "Most_Used_Platform": "Instagram",
  "Purpose_Of_Use": "Entertainment",
  "Avg_Daily_Usage_Hours": 4.5,
  "Daily_Unlocks": 60,
  "Study_Hours": 3.0,
  "Physical_Activity_Hours": 1.0,
  "Sleep_Hours_Per_Night": 6.5,
  "Stress_Level": "High"
}
```

**Response:**

```json
{ "Predicted_mental_health_score": 5.83 }
```

| Field | Type | Notes |
|---|---|---|
| `Age` | int | 10–100 |
| `Gender` | enum | `Male`, `Female` |
| `Country` | string | Grouped into top 10 countries + `Other` server-side |
| `Academic_Level` | enum | `High School`, `Undergraduate`, `Graduate` |
| `Most_Used_Platform` | enum | Facebook, LinkedIn, Instagram, Snapchat, Twitter, YouTube, TikTok, LINE, KakaoTalk, VKontakte, WhatsApp, WeChat |
| `Purpose_Of_Use` | enum | Networking, Education, Entertainment, News |
| `Avg_Daily_Usage_Hours` | float | 0–24 |
| `Daily_Unlocks` | int | ≥ 0 |
| `Study_Hours` | float | 0–24 |
| `Physical_Activity_Hours` | float | 0–24 |
| `Sleep_Hours_Per_Night` | float | 0–24 |
| `Stress_Level` | enum | Low, Medium, High, Very High |

⚠️ **Note:** `allow_origins=["*"]` is currently set in the CORS middleware for local development. Restrict this to your actual frontend domain before deploying to production.

---

## 🛠️ Tech Stack

- **ML:** scikit-learn, pandas, numpy, joblib
- **Backend:** FastAPI, Pydantic, Uvicorn
- **Frontend:** vanilla HTML/CSS/JS (no build step required)

---

## 📄 License

Add a license of your choice here (e.g. MIT) if you plan to make this repository public.
