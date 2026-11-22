# Mock API Server

[![Python Version](https://img.shields.io/badge/python-3.10%20%7C%203.11%20%7C%203.12-blue.svg)](https://www.python.org/)
[![Build Status](https://img.shields.io/badge/build-passing-brightgreen.svg)]()
[![Code Style: Black](https://img.shields.io/badge/code%20style-black-000000.svg)](https://github.com/psf/black)
[![Framework: FastAPI](https://img.shields.io/badge/Framework-FastAPI-009688.svg)](https://fastapi.tiangolo.com/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

A lightweight, self-hosted mock API engine and web dashboard engineered for frontend developers, mobile teams, and QA engineers. Create dynamic HTTP mock endpoints, inject network delays, simulate status code failures, and validate payload schemas without writing backend code.

---

## Table of Contents

- [Problem Statement & Target Audience](#problem-statement--target-audience)
- [Key Features](#key-features)
- [Architecture & System Design](#architecture--system-design)
- [Technology Stack](#technology-stack)
- [Installation & Setup](#installation--setup)
- [Environment Variables Configuration](#environment-variables-configuration)
- [Usage Guide](#usage-guide)
  - [Starting the Application](#starting-the-application)
  - [API Endpoints](#api-endpoints)
  - [cURL Examples](#curl-examples)
- [Automated Testing](#automated-testing)
- [Project Structure](#project-structure)
- [Roadmap](#roadmap)
- [License](#license)

---

## Problem Statement & Target Audience

### The Problem
Frontend and mobile integration workflows often stall due to:
* **Unfinished Microservices:** Waiting for backend endpoints blocks user interface development.
* **Unstable Sandbox Environments:** External partner APIs frequently go down or enforce unpredictable rate limits during integration testing.
* **Hardcoded Client Stubs:** Placing static JSON files inside client apps leads to messy git commits, code pollution, and an inability to simulate HTTP server errors (4xx/5xx), variable latency, or rate-limiting handling.

### Target Audience
* **Frontend Developers:** Build and verify UI components against precise mock endpoints.
* **QA & Automation Engineers:** Conduct edge-case, validation, and performance testing against predictable backend states.
* **Mobile Developers:** Test app resilience under bad network conditions (high latency, rate limiting, system errors).

---

## Key Features

* **Interactive Web Interface:** Define custom route paths, HTTP methods (GET, POST, PUT, DELETE, PATCH), HTTP status codes (200-599), and JSON response bodies on the fly.
* **Dynamic Route & Schema Validation Engine:** Match incoming requests and validate incoming JSON payloads against user-defined dynamic Pydantic schema rules.
* **Configurable Network Conditions:** Simulate real-world network anomalies by configuring artificial asynchronous latency and rate-limiting rules per endpoint.
* **Live Request Audit Log:** Inspect client IP addresses, incoming headers, request body contents, matching rules, and microsecond-level execution timings in real-time.

---

## Architecture & System Design

```
                     +-------------------------------------------------+
                     |                 Client Request                  |
                     +-------------------------------------------------+
                                              |
                                              v
                     +-------------------------------------------------+
                     |           FastAPI Router (src/api.py)           |
                     |       - Dynamic Catch-All Dynamic Interceptor   |
                     |       - Management Web Dashboard / UI          |
                     +-------------------------------------------------+
                                              |
                                              v
                     +-------------------------------------------------+
                     |          Core Mock Engine (src/core.py)         |
                     |  1. Path Matching & Method Lookup               |
                     |  2. Rate-Limit Evaluation                       |
                     |  3. Pydantic Dynamic Schema Validation          |
                     |  4. Asynchronous Latency Injector (async sleep) |
                     +-------------------------------------------------+
                                    |                   |
                                    v                   v
                     +----------------------+   +----------------------+
                     |   SQLAlchemy Models  |   |   Audit Logger       |
                     |   (src/models.py)    |   |   (Record Logs)      |
                     +----------------------+   +----------------------+
                                    |                   |
                                    +---------+---------+
                                              |
                                              v
                     +-------------------------------------------------+
                     |            SQLite Database (DB Layer)           |
                     +-------------------------------------------------+
```

### Module Responsibilities

| Module | Purpose |
| :--- | :--- |
| **`src/database.py`** | SQLAlchemy engine setup, session factories, and database connection lifecycle management. |
| **`src/models.py`** | SQLAlchemy ORM models for endpoint mock rules, rate limit policies, and request audit logs. |
| **`src/schemas.py`** | Pydantic DTO schemas for endpoint creation, updates, response formatting, and payload validation. |
| **`src/core.py`** | Core matching algorithm, dynamic runtime schema validation, rate-limiting, and async latency sleep routines. |
| **`src/api.py`** | FastAPI routes for dashboard rendering, management REST API, and catch-all mock request interception. |
| **`src/main.py`** | FastAPI initialization, database migration execution on startup, static/Jinja2 template mounting, and Uvicorn entrypoint. |

---

## Technology Stack

| Category | Technology |
| :--- | :--- |
| **Language** | Python 3.10+ |
| **Web Framework** | FastAPI |
| **Database** | SQLite |
| **ORM & Database Toolkit** | SQLAlchemy 2.0 |
| **Data Validation** | Pydantic v2 |
| **Frontend Rendering** | Jinja2, Vanilla JavaScript, HTML5/CSS3 |
| **Testing** | pytest, HTTPX (Async Test Client) |
| **ASGI Server** | Uvicorn |

---

## Installation & Setup

### Prerequisites
* Python 3.10 or higher installed.
* `git` and `pip` installed.

### 1. Clone the Repository
```bash
git clone https://github.com/your-org/mock-api-server.git
cd mock-api-server
```

### 2. Create and Activate Virtual Environment
```bash
# On Linux/macOS
python3 -m venv venv
source venv/bin/activate

# On Windows
python -m venv venv
venv\Scripts\activate
```

### 3. Install Dependencies
```bash
pip install --upgrade pip
pip install -r requirements.txt
```

---

## Environment Variables Configuration

Copy `.env.example` to create your custom configuration file:

```bash
cp .env.example .env
```

### Available Settings

| Variable Name | Description | Default Value |
| :--- | :--- | :--- |
| `DATABASE_URL` | SQLite database connection string | `sqlite:///./mock_server.db` |
| `HOST` | Bind network host address | `127.0.0.1` |
| `PORT` | Service listening port | `8000` |
| `DEBUG` | Enable verbose application logging | `True` |
| `LOG_LEVEL` | Python logging level (`INFO`, `DEBUG`, `WARN`, `ERROR`) | `INFO` |

---

## Usage Guide

### Starting the Application

Launch the server locally using the main entrypoint:

```bash
python -m src.main
```

Alternatively, launch using Uvicorn directly with hot-reload enabled:

```bash
uvicorn src.main:app --host 127.0.0.1 --port 8000 --reload
```

Access the **Web Dashboard** at `http://127.0.0.1:8000/dashboard` to manage endpoints interactively.

---

### API Endpoints

#### Management & System Endpoints

| Method | Endpoint | Description |
| :--- | :--- | :--- |
| `GET` | `/dashboard` | Render the Jinja2 management user interface. |
| `GET` | `/api/v1/rules` | Fetch all configured mock endpoint rules. |
| `POST` | `/api/v1/rules` | Register a new mock endpoint rule. |
| `GET` | `/api/v1/rules/{id}` | Get details for a specific mock rule. |
| `DELETE` | `/api/v1/rules/{id}` | Delete a mock endpoint rule. |
| `GET` | `/api/v1/logs` | Fetch real-time request audit logs. |

#### Catch-All Dynamic Mock Interceptor

| Method | Endpoint | Description |
| :--- | :--- | :--- |
| `ANY` | `/mock/{path:path}` | Intercepts matching requests, applies rules, delays, & returns stub responses. |

---

### cURL Examples

#### 1. Define a Mock Endpoint Rule
Create a mock rule that returns a HTTP 201 Created status with artificial 500ms latency:

```bash
curl -X POST "http://127.0.0.1:8000/api/v1/rules" \
  -H "Content-Type: application/json" \
  -d '{
    "path": "users/create",
    "method": "POST",
    "status_code": 201,
    "delay_ms": 500,
    "rate_limit_rpm": 60,
    "response_body": {
      "id": 42,
      "status": "success",
      "message": "User provisioned successfully"
    },
    "schema_validation": {
      "type": "object",
      "required": ["username", "email"]
    }
  }'
```

#### 2. Trigger the Mock Endpoint
Send a request to the configured mock endpoint:

```bash
curl -X POST "http://127.0.0.1:8000/mock/users/create" \
  -H "Content-Type: application/json" \
  -d '{
    "username": "johndoe",
    "email": "john@example.com"
  }'
```

**Response (after ~500ms):**
```json
{
  "id": 42,
  "status": "success",
  "message": "User provisioned successfully"
}
```

#### 3. View Execution Audit Logs
Retrieve recent execution logs and execution times:

```bash
curl -X GET "http://127.0.0.1:8000/api/v1/logs"
```

---

## Automated Testing

The suite uses `pytest` and FastAPI's `TestClient` / HTTPX library for comprehensive coverage.

### Run All Tests
```bash
pytest
```

### Run Core Engine Tests specifically
```bash
pytest tests/test_core.py -v
```

### Run API Interceptor Tests specifically
```bash
pytest tests/test_api.py -v
```

### Run Tests with Coverage Report
```bash
pytest --cov=src --cov-report=term-missing
```

---

## Project Structure

```
mock-api-server/
│
├── .env.example           # Environment template file
├── .gitignore             # Git ignore patterns
├── requirements.txt       # Production dependencies
│
├── src/                   # Core application source code
│   ├── __init__.py        # Package initializer
│   ├── database.py        # SQLAlchemy engine & session lifecycle
│   ├── models.py          # Database ORM models (Rules, Logs)
│   ├── schemas.py         # Pydantic validation schemas
│   ├── core.py            # Core rule matching & validation engine
│   ├── api.py             # FastAPI routes & mock interceptor
│   └── main.py            # Entrypoint & FastAPI app startup
│
└── tests/                 # Test suite
    ├── __init__.py        # Test package initializer
    ├── test_core.py       # Core logic unit tests
    └── test_api.py        # API endpoint integration tests
```

---

## Roadmap

- [ ] **OpenAPI / Swagger Import & Export:** Automatically generate mock endpoints from uploaded `openapi.json` files.
- [ ] **Dynamic Variable Templating:** Support dynamic placeholders inside response bodies (e.g., `{{$randomUUID}}`, `{{$timestamp}}`).
- [ ] **WebSocket & SSE Mocking:** Support dynamic event streaming mock handlers.
- [ ] **JWT Bearer Auth Validation:** Enable configurable identity verification on stub routes.

---

## License

Distributed under the MIT License. See `LICENSE` for details.