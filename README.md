# AttentionPay

AttentionPay is an AI-assisted payment security system designed to
detect phishing websites and identify potentially fraudulent
transactions through a layered security pipeline.

The system combines BERT-based phishing detection, deterministic backend
transaction-security checks, TabTransformer-based fraud detection, and
Explainable AI (XAI) to provide both a security decision and an
explanation of the model's prediction.

> **Project Scope:** AttentionPay is a controlled payment-security
> simulation developed for academic and demonstration purposes. It is
> not intended to process real financial transactions or replace
> production banking security systems.

------------------------------------------------------------------------

## Live Demo

**Deployed Link:** https://attention-pay.vercel.app

The backend is deployed separately and is accessed by the frontend
through the configured `VITE_API_BASE_URL`.

------------------------------------------------------------------------

## Features

-   BERT-based phishing URL detection
-   Two-stage payment security pipeline
-   Temporary Stage 2 access token issued only after a legitimate Stage
    1 result
-   API route integrity verification
-   Impossible-travel detection
-   PostgreSQL-backed transaction simulation
-   TabTransformer-based transaction fraud detection
-   SHAP/LIME-based explainability
-   Human-readable security explanations
-   Interactive transaction-security dashboard
-   React + Vite frontend
-   FastAPI backend
-   Docker-ready backend
-   CI/CD deployment through GitHub Actions and Azure Container Apps

------------------------------------------------------------------------

## System Architecture

``` text
                         ┌─────────────────────┐
                         │        User         │
                         └──────────┬──────────┘
                                    │
                                    ▼
                         ┌─────────────────────┐
                         │   Stage 1: BERT     │
                         │ Phishing Detection  │
                         └──────────┬──────────┘
                                    │
                       ┌────────────┴────────────┐
                       │                         │
                    Phishing                  Legitimate
                       │                         │
                       ▼                         ▼
                    BLOCK              ┌─────────────────────┐
                                       │ Stage 2: Transaction│
                                       │      Security       │
                                       └──────────┬──────────┘
                                                  │
                                                  ▼
                                  ┌──────────────────────────┐
                                  │ Layer 1 Security Checks  │
                                  │                          │
                                  │ • API Route Integrity    │
                                  │ • Impossible Travel      │
                                  └────────────┬─────────────┘
                                               │
                              ┌────────────────┴────────────────┐
                              │                                 │
                            BLOCK                              PASS
                              │                                 │
                              ▼                                 ▼
                           BLOCKED                    ┌─────────────────────┐
                                                      │ Feature Engineering │
                                                      └──────────┬──────────┘
                                                                 │
                                                                 ▼
                                                      ┌─────────────────────┐
                                                      │    TabTransformer   │
                                                      │  Fraud Detection    │
                                                      └──────────┬──────────┘
                                                                 │
                                                                 ▼
                                                      ┌─────────────────────┐
                                                      │   Explainable AI    │
                                                      │     SHAP / LIME     │
                                                      └──────────┬──────────┘
                                                                 │
                                                                 ▼
                                                      ┌─────────────────────┐
                                                      │   Result Dashboard  │
                                                      └─────────────────────┘
```

------------------------------------------------------------------------

## Technology Stack

### Frontend

-   React
-   Vite
-   Axios
-   React Leaflet
-   Recharts
-   CSS

### Backend

-   Python
-   FastAPI
-   Uvicorn
-   Pydantic Settings
-   SQLAlchemy
-   PostgreSQL
-   Python-dotenv

### Machine Learning

#### Stage 1 --- Phishing Detection

-   BERT
-   Hugging Face Transformers
-   PyTorch

The model analyzes submitted URLs and classifies them as:

-   Legitimate
-   Phishing

#### Stage 2 --- Transaction Fraud Detection

-   TabTransformer
-   PyTorch
-   Scikit-learn
-   Joblib

The deployed model processes the following **11 engineered features**:

1.  `transactions_last_1min`
2.  `transactions_last_5min`
3.  `transactions_last_10min`
4.  `transaction_amount`
5.  `previous_transaction_amount`
6.  `known_device_flag`
7.  `device_changed_flag`
8.  `device_type`
9.  `browser_name`
10. `operating_system`
11. `session_risk_score`

The model uses the project's final label convention:

``` text
0 = Fraud
1 = Legitimate
```

### Explainable AI

-   SHAP
-   LIME
-   Model-focused explanation information
-   Human-readable explanation generation

------------------------------------------------------------------------

## Project Structure

``` text
AttentionPay/
│
├── backend/
│   ├── app/
│   │   ├── dependencies/
│   │   ├── models/
│   │   ├── routes/
│   │   ├── schemas/
│   │   ├── services/
│   │   │   └── explainability/
│   │   ├── utils/
│   │   ├── config.py
│   │   ├── database.py
│   │   └── main.py
│   │
│   ├── ml/
│   │   └── tabtransformer/
│   │       ├── artifacts/
│   │       └── model.py
│   │
│   ├── create_tables.py
│   ├── seed_transactions.py
│   ├── Dockerfile
│   ├── requirements.txt
│   └── requirements-docker.txt
│
├── frontend/
│   ├── public/
│   ├── src/
│   │   ├── api/
│   │   ├── app/
│   │   ├── components/
│   │   ├── hooks/
│   │   ├── pages/
│   │   ├── state/
│   │   ├── styles/
│   │   └── utils/
│   ├── package.json
│   ├── package-lock.json
│   └── vite.config.js
│
├── model/
│   ├── config.json
│   ├── model.safetensors
│   ├── tokenizer.json
│   └── tokenizer_config.json
│
├── AttentionPay_SDD.pdf
├── .github/
│   └── workflows/
│       └── backend-deploy.yml
├── .gitignore
└── README.md
```

------------------------------------------------------------------------

## Security Pipeline

AttentionPay uses a layered approach rather than relying entirely on a
single machine-learning model.

### Stage 1 --- Phishing Detection

The user first submits a payment or website URL.

The BERT model analyzes the URL and determines whether it is
**Legitimate** or **Phishing**.

-   If the URL is classified as phishing, the transaction flow is
    stopped.
-   If the URL is considered legitimate, the backend issues temporary
    Stage 2 access and the user can proceed to the transaction-security
    stage.

### Stage 2 --- Transaction Security

Stage 2 contains two security layers.

#### Layer 1 --- Backend Security Checks

Before the machine-learning fraud model is executed, the backend
performs deterministic security checks including:

**API Route Integrity**

Checks whether the expected API endpoint matches the actual endpoint
associated with the transaction.

**Impossible Travel**

Checks whether the user's transaction locations and timestamps imply an
implausible travel speed.

For example:

``` text
Previous Transaction
Location: Bengaluru
Time: 10:00 AM

Current Transaction
Location: London
Time: 10:05 AM
```

Such a transition can be flagged by the backend security layer.

If a Layer 1 security check fails, the transaction is blocked before the
TabTransformer is executed.

#### Layer 2 --- TabTransformer Fraud Detection

Transactions that pass Layer 1 are processed by the TabTransformer
model.

The backend first converts the transaction data into the exact feature
representation expected by the trained model.

The TabTransformer then produces a:

-   Fraud prediction
-   Legitimate prediction
-   Fraud probability
-   Legitimate probability

The model operates only on the 11-feature inference contract listed
above. API route integrity and impossible-travel results are **Layer 1
security checks**, not TabTransformer input features.

------------------------------------------------------------------------

## Explainable AI

AttentionPay does not only provide a prediction.

When the TabTransformer is executed, the system also generates
explanation information for the prediction.

The XAI layer uses:

-   SHAP
-   LIME
-   Model-focused explanation information
-   Human-readable explanation generation

The frontend presents these explanations through the results dashboard
so the user can understand which transaction characteristics influenced
the model's decision.

If Layer 1 blocks a transaction, the TabTransformer and its XAI stages
are skipped. The frontend explicitly displays those stages as skipped
rather than generating artificial explanation values.

------------------------------------------------------------------------

## Transaction Simulation

AttentionPay includes a controlled transaction simulation system for
demonstrating different security conditions.

Supported scenarios include:

-   Normal Transaction
-   Impossible Travel
-   API Route Tampering
-   Behaviour Fraud
-   Random Transaction

The final implementation uses **database-backed simulation records**
rather than generating a new transaction inside the request flow.

The flow is:

``` text
Stage 2 Access
       │
       ▼
Scenario Selection
       │
       ▼
PostgreSQL Transaction Retrieval
       │
       ▼
Read-only Transaction Preview
       │
       ▼
Layer 1 Security Checks
       │
       ├── Block
       │
       └── Pass
              │
              ▼
       Feature Engineering
              │
              ▼
        TabTransformer
              │
              ▼
       Explainable AI
              │
              ▼
        Final Result
```

The database seeding script is a separate initialization utility. It is
**not automatically executed as part of the normal application startup
or CI/CD deployment workflow**.

------------------------------------------------------------------------

## Backend API

The backend is implemented using FastAPI.

### Health Check

``` http
GET /health
```

Used to verify that the backend is running.

### Analyze URL

``` http
POST /api/v1/analyze/url
```

Analyzes a submitted URL using the Stage 1 BERT phishing detection
model.

A legitimate result creates the temporary authorization required for the
Stage 2 transaction-security flow.

### Retrieve Simulation Transaction

``` http
POST /api/v1/simulate/transaction
```

Retrieves a transaction record for the selected simulation scenario.

The endpoint requires the Stage 2 access token.

### Process Transaction

``` http
POST /api/v1/simulate/transaction/{transaction_id}/process
```

Processes the selected transaction through:

``` text
Layer 1 Security
       ↓
Feature Engineering
       ↓
TabTransformer
       ↓
Explainability
       ↓
Final Decision
```

The endpoint also requires the Stage 2 access token.

------------------------------------------------------------------------

## Installation

### 1. Clone the Repository

``` bash
git clone https://github.com/Venu-R/AttentionPay.git
cd AttentionPay
```

### Backend Setup

### 2. Create a Python Virtual Environment

From the project root:

**Windows**

``` bash
python -m venv .venv
.venv\Scripts\activate
```

**Linux / macOS**

``` bash
python3 -m venv .venv
source .venv/bin/activate
```

### 3. Install Backend Dependencies

``` bash
pip install -r backend/requirements.txt
```

### 4. Configure Environment Variables

Create a `.env` file according to the backend configuration required by
the project.

The database connection is provided through the backend environment
configuration.

> **Do not** commit database passwords, API keys, credentials, or other
> secrets to Git.

### 5. Initialize the Database

Create the required tables:

``` bash
python backend/create_tables.py
```

To populate the controlled demonstration dataset:

``` bash
python backend/seed_transactions.py
```

> The seed script is an explicit database initialization command. It is
> not automatically run every time the backend starts or every time the
> CI/CD pipeline deploys a new container.

### 6. Start the Backend

From the project root:

``` bash
uvicorn backend.app.main:app --reload
```

The backend will be available at:

``` text
http://127.0.0.1:8000
```

FastAPI Swagger documentation:

``` text
http://127.0.0.1:8000/docs
```

### Frontend Setup

Open another terminal:

``` bash
cd frontend
```

Install dependencies:

``` bash
npm install
```

Start the development server:

``` bash
npm run dev
```

Vite will provide the local frontend URL in the terminal.

------------------------------------------------------------------------

## Running the Full Application

You should have two terminals running.

**Terminal 1 --- Backend**

From the project root:

``` bash
uvicorn backend.app.main:app --reload
```

**Terminal 2 --- Frontend**

``` bash
cd frontend
npm run dev
```

The frontend communicates with the FastAPI backend through the
configured API base URL.

------------------------------------------------------------------------

## Database

AttentionPay uses PostgreSQL for transaction simulation data.

The backend contains two separate database utilities:

### Create Tables

``` bash
python backend/create_tables.py
```

This creates the SQLAlchemy-defined tables if they do not already exist.

### Seed Demonstration Transactions

``` bash
python backend/seed_transactions.py
```

The seed script creates the controlled demonstration records used by the
simulation.

The current seed configuration creates:

``` text
15 Normal Transactions
15 Impossible Travel Transactions
15 API Route Tampering Transactions
15 Behaviour Fraud Transactions
--------------------------------
60 Total Transactions
```

The seeding script first removes existing transaction simulation records
and then inserts a new randomized set.

**Important:** this script is not called by `main.py`, the Dockerfile,
or the GitHub Actions deployment workflow. Therefore, deploying a new
backend image does not automatically reseed the PostgreSQL database.

------------------------------------------------------------------------

## Model Artifacts

The repository contains the required trained-model artifacts.

### Stage 1

The BERT model and tokenizer files are located under `model/`,
including:

-   `model.safetensors`
-   `config.json`
-   `tokenizer.json`
-   `tokenizer_config.json`

### Stage 2

The TabTransformer model and preprocessing artifacts are located under:

``` text
backend/ml/tabtransformer/artifacts/
```

These include the trained model and preprocessing components required
for inference.

------------------------------------------------------------------------

## Deployment and CI/CD

The backend is containerized using Docker.

The repository contains a GitHub Actions workflow:

``` text
.github/workflows/backend-deploy.yml
```

The workflow is configured to run on:

``` text
push → main
```

The backend deployment process is:

``` text
Push to main
      │
      ▼
GitHub Actions
      │
      ├── Checkout repository
      ├── Verify BERT model
      ├── Compile backend Python files
      ├── Authenticate with Azure
      ├── Build Docker image
      ├── Push image to Azure Container Registry
      └── Update Azure Container App
```

The workflow does **not** run:

``` text
backend/create_tables.py
```

or:

``` text
backend/seed_transactions.py
```

Therefore, a normal source-code deployment does not recreate tables or
reseed transaction data.

Database initialization/seeding is a separate operation.

------------------------------------------------------------------------

## Development Notes

AttentionPay separates security decisions into different stages.

The general decision flow is:

``` text
URL
 │
 ▼
BERT Phishing Detection
 │
 ├── Phishing ───────────────► BLOCK
 │
 └── Legitimate
        │
        ▼
 Temporary Stage 2 Access
        │
        ▼
   Transaction Retrieval
        │
        ▼
 Layer 1 Security
        │
        ├── Failed ──────────► BLOCK
        │
        └── Passed
               │
               ▼
        Feature Engineering
               │
               ▼
         TabTransformer
               │
               ▼
          Fraud Check
               │
               ▼
        SHAP / LIME / XAI
               │
               ▼
         Result Dashboard
```

This separation ensures that deterministic security checks can stop a
transaction before invoking the machine-learning fraud model.

The backend is the source of truth for security decisions. The frontend
visualizes the backend response and does not independently determine
whether a transaction is fraudulent or legitimate.

------------------------------------------------------------------------

## Project Documentation

The detailed Software Design Document is available in:

``` text
AttentionPay_SDD.pdf
```

It contains the project's architecture, system design, processing flow,
model components, security layers, explainability design, deployment
architecture, and implementation details.

------------------------------------------------------------------------

## Disclaimer

AttentionPay is an academic and demonstration project.

It uses simulated transaction scenarios and machine-learning models to
demonstrate a layered payment-security architecture.

It should not be used as a production payment-processing system, banking
system, or security solution without appropriate security validation,
compliance review, infrastructure hardening, monitoring, and independent
testing.

------------------------------------------------------------------------

## License

This project is intended for academic and educational purposes.

No open-source license is currently specified for the repository.
