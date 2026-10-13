# Employee Attendance & Analytics API

A FastAPI and MongoDB backend service for employee attendance tracking and organizational analytics, implementing the OpenAPI 3.0.3 contract and business rules R1–R10.

## Overview
- **Single file implementation**: All service code is consolidated in `app/main.py`.
- **Automatic indexing**: All unique and compound indexes are created idempotently on startup via FastAPI lifespan.
- **Aggregation pipelines**: Analytics endpoints (`monthly`, `summary`, `leaderboard`, `trend`) are executed as native MongoDB aggregations using `$lookup`, `$densify`, and `$setWindowFields`.
- **Explain verification**: `GET /admin/explain/{endpoint}` proves query efficiency through indexed scans (`IXSCAN`) without collection scans (`COLLSCAN`).
- **Review & Decisions**: Defect root-cause analysis is documented in `REVIEW.md`, and architectural decisions are detailed in `DECISIONS.md`.

## Requirements
- Python 3.11+
- MongoDB 6.0 or newer

## Setup & Running

1. **Install dependencies**:
   ```bash
   pip install -r requirements.txt
   ```

2. **Configure environment variables (optional, defaults to localhost:27017 / attendance_db)**:
   ```bash
   export MONGO_URI="mongodb://localhost:27017"
   export MONGO_DB="attendance_db"
   ```

3. **Seed sample data (optional)**:
   ```bash
   python sample_seed.py
   ```

4. **Launch the application**:
   ```bash
   uvicorn app.main:app --port 8000
   ```

5. **Health check**:
   ```bash
   curl http://localhost:8000/health
   # {"status":"ok"}
   ```
