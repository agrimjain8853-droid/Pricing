Fair Value Engine — Architecture Diagram
┌────────────────────┐
│      User / UI     │
│  (Browser Client)  │
│                    │
│  - Search item     │
│  - Submit price    │
│  - View chart      │
└─────────┬──────────┘
          │ HTTP (GET / POST)
          ▼
┌──────────────────────────────┐
│          FastAPI App         │
│                              │
│  ┌────────────────────────┐ │
│  │  HTML Routes (SSR)     │ │
│  │  - GET /               │ │
│  │  - POST /submit        │ │
│  │  (Jinja Templates)    │ │
│  └──────────┬────────────┘ │
│             │                │
│  ┌──────────▼────────────┐ │
│  │   API Routes (JSON)   │ │
│  │  - GET /api/fair-value│ │
│  │  - GET /chart-data    │ │
│  └──────────┬────────────┘ │
│             │                │
│  ┌──────────▼────────────┐ │
│  │  Pricing Engine       │ │
│  │  (pricing.py)         │ │
│  │  - Outlier removal    │ │
│  │  - Median             │ │
│  │  - Trimmed mean       │ │
│  │  - Fair value         │ │
│  │  - Confidence score   │ │
│  └──────────┬────────────┘ │
│             │                │
│  ┌──────────▼────────────┐ │
│  │   Persistence Layer   │ │
│  │  SQLAlchemy ORM       │ │
│  └──────────┬────────────┘ │
└─────────────┼────────────────┘
              │
              ▼
┌──────────────────────────────────┐
│          PostgreSQL DB            │
│  (Supabase / Managed Postgres)   │
│                                  │
│  Tables:                          │
│  ┌────────────────────────────┐  │
│  │  price_estimates           │  │
│  │  - item_id                 │  │
│  │  - price                   │  │
│  │  - timestamp               │  │
│  └────────────────────────────┘  │
│                                  │
│  ┌────────────────────────────┐  │
│  │  fair_value_history        │  │
│  │  - item_id                 │  │
│  │  - fair_value              │  │
│  │  - median                  │  │
│  │  - trimmed_mean            │  │
│  │  - confidence              │  │
│  │  - timestamp               │  │
│  └────────────────────────────┘  │
└──────────────────────────────────┘

🧠 How the system works (clear narrative)
1️⃣ User Interaction (Frontend)

User opens the website

Can:

Search for an item’s fair value

Submit a new price estimate

View trends on a chart

Frontend uses:

HTML + CSS (Jinja templates)

JavaScript (Fetch API)

Chart.js for visualization

2️⃣ FastAPI Backend

FastAPI acts as the central orchestrator:

A. Server-Side Rendering (SSR)

GET / → loads UI

POST /submit → validates & stores price, recomputes fair value

B. API Layer (AJAX)

GET /api/fair-value/{item} → returns fair value + confidence

GET /chart-data/{item} → returns historical analytics

This separation keeps:

Writes → server-side

Reads → API-driven

3️⃣ Pricing Engine (Core Intelligence)

Encapsulated in pricing.py

Responsibilities:

Remove outliers

Compute median

Compute trimmed mean

Blend into fair value

Generate confidence score

This layer is:

Reusable

Testable

Independent of UI or database

4️⃣ Persistence Layer (Data Storage)

Using SQLAlchemy ORM over PostgreSQL:

price_estimates

Raw user-submitted prices

High volume, noisy data

fair_value_history

Aggregated, computed analytics

Used for charts and trends

Grouped daily for clean insights

This separation is very important:

Raw data ≠ Analytics data

5️⃣ Analytics & Visualization

Chart pulls data from fair_value_history

Backend aggregates per day

Frontend renders:

Fair Value

Median

Trimmed Mean

Updated on:

Search

Submit
