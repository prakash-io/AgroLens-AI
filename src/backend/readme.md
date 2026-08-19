# IH-21 — Crop Disease & Stress Detection Backend

## Overview

The backend of IH-21 will be built using **FastAPI (Python)**.

It will act as the common backend for both:

* **Web Application — Phase 1**
* **Mobile Application — Phase 2**

The backend will connect the frontend applications with the **AI/ML model, database, and recommendation engine**.

```text
Web App ────────┐
                │
                ▼
           FastAPI Backend
                │
       ┌────────┼────────┐
       ▼        ▼        ▼
      ML     Database  Recommendation
    Model      │          Engine
       │       │
       └───────┴──────────┐
                          │
                     Prediction
                          │
                          ▼
                   Web / Mobile
```

---

# 1. Backend Folder Structure

```text
backend/
│
├── app/
│   ├── main.py
│   │
│   ├── api/
│   │   └── v1/
│   │       ├── auth.py
│   │       ├── users.py
│   │       ├── predictions.py
│   │       ├── crops.py
│   │       └── history.py
│   │
│   ├── models/
│   │   ├── user.py
│   │   ├── crop.py
│   │   └── prediction.py
│   │
│   ├── schemas/
│   │   ├── auth.py
│   │   ├── user.py
│   │   ├── crop.py
│   │   └── prediction.py
│   │
│   ├── services/
│   │   ├── auth_service.py
│   │   ├── prediction_service.py
│   │   ├── image_service.py
│   │   └── recommendation_service.py
│   │
│   ├── ml/
│   │   ├── model_loader.py
│   │   ├── inference.py
│   │   └── preprocessing.py
│   │
│   ├── db/
│   │   └── database.py
│   │
│   └── core/
│       ├── config.py
│       └── security.py
│
├── tests/
├── .env
├── .env.example
├── requirements.txt
└── README.md
```

---

# 2. What Each Part Does

### `main.py`

Entry point of the FastAPI application.

Responsible for:

* Creating FastAPI app
* Registering API routes
* CORS
* Application startup

---

### `api/v1/`

Contains all API endpoints.

```text
auth.py          → Login / Register
users.py         → User profile
crops.py         → Crop information
predictions.py   → AI prediction
history.py       → Previous predictions
```

API version:

```text
/api/v1/
```

---

### `models/`

Defines the database structure.

Initial models:

```text
User
Crop
Prediction
```

---

### `schemas/`

Defines the structure of API requests and responses using Pydantic.

For example:

```text
PredictionRequest
PredictionResponse
UserResponse
LoginRequest
```

**Models = Database structure**

**Schemas = API data structure**

---

### `services/`

Contains the actual business logic.

For example:

```text
prediction_service.py
        ↓
Image Processing
        ↓
ML Model
        ↓
Severity
        ↓
Recommendation
        ↓
Save Prediction
```

Routes should remain thin and call services instead of containing all the logic.

---

### `ml/`

Handles AI model integration.

```text
model_loader.py   → Load trained model
inference.py      → Run prediction
preprocessing.py  → Prepare image
```

The **training code should remain separate from the backend**.

The backend only needs the trained model for inference.

---

### `db/`

Handles database connection and configuration.

We can use **PostgreSQL** as the main database.

---

### `core/`

Contains common configuration and security.

Examples:

```text
config.py
    ↓
Environment variables

security.py
    ↓
JWT
Password hashing
Authentication
```

---

# 3. Main API Endpoints

Initial API design:

```text
AUTH
POST   /api/v1/auth/register
POST   /api/v1/auth/login
GET    /api/v1/auth/me


USERS
GET    /api/v1/users/me
PUT    /api/v1/users/me


CROPS
GET    /api/v1/crops
GET    /api/v1/crops/{crop_id}


PREDICTION
POST   /api/v1/predictions


HISTORY
GET    /api/v1/history
GET    /api/v1/history/{prediction_id}
```

More endpoints can be added later when required.

---

# 4. Main Prediction Flow

This is the most important backend workflow:

```text
Web App
   │
   │ Image + Crop
   ▼
FastAPI
   │
   ▼
Validate Image
   │
   ▼
Preprocess Image
   │
   ▼
AI Model
   │
   ▼
Disease / Stress
   │
   ▼
Confidence
   │
   ▼
Severity
   │
   ▼
Recommendation Engine
   │
   ▼
Save Prediction
   │
   ▼
Return Result
   │
   ▼
Web App
```

Example result:

```json
{
  "crop": "Tomato",
  "disease": "Early Blight",
  "confidence": 0.92,
  "severity": "Moderate",
  "recommendation": [
    "Remove affected leaves",
    "Avoid overhead watering"
  ]
}
```

---

# 5. Database

Initial database structure:

```text
User
 └── id
 └── name
 └── email
 └── password_hash
 └── location

Crop
 └── id
 └── name

Prediction
 └── id
 └── user_id
 └── crop_id
 └── image
 └── disease
 └── confidence
 └── severity
 └── recommendation
 └── created_at
```

Relationship:

```text
User
 │
 │ 1 : Many
 ▼
Prediction
 │
 │ Many : 1
 ▼
Crop
```

---

# 6. Authentication

We will use:

```text
JWT Authentication
+
Password Hashing
```

Protected APIs such as prediction and history will require authentication.

---

# 7. Environment Variables

Sensitive configuration will be stored in `.env`.

Example:

```text
DATABASE_URL=
SECRET_KEY=
MODEL_PATH=
ALLOWED_ORIGINS=
```

`.env` will not be committed to GitHub.

An `.env.example` file will be provided for other developers.

---

# 8. Backend Development Order

We should implement the backend in this order:

```text
1. FastAPI setup
       ↓
2. Database setup
       ↓
3. Authentication
       ↓
4. Crop APIs
       ↓
5. Image upload
       ↓
6. ML model integration
       ↓
7. Prediction API
       ↓
8. Recommendation engine
       ↓
9. Prediction history
       ↓
10. Web App integration
```

---

# 9. Important Architecture Rule

The backend should follow:

```text
API Route
    ↓
Service
    ↓
ML / Database / Recommendation
```

Do not put everything inside `predictions.py`.

For example:

```text
predictions.py
      ↓
prediction_service.py
      ↓
ml/inference.py
      ↓
AI Model
```

This will keep the project clean and make it easier to add the mobile application later.

---

# 10. Future Mobile Application

The mobile application will use the **same FastAPI backend**.

```text
              ┌──────────────┐
              │   Web App    │
              └──────┬───────┘
                     │
                     ▼
                FastAPI API
                     ▲
                     │
              ┌──────┴───────┐
              │ Mobile App   │
              └──────────────┘
```

Therefore, we should design the backend to be **client-independent from the beginning**.

---

## Backend Goal

The backend should provide a clean and scalable API that handles:

**Authentication → Image → AI Prediction → Severity → Recommendation → History**

The Web Application will be the first client, and the Mobile Application will be added later without requiring major changes to the backend.

