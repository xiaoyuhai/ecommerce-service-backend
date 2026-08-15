# AGENTS.md

Mock e-commerce support service (FastAPI + SQLAlchemy) that the separate
`atguigu` customer-service chatbot project calls for orders/logistics/products.

## Toolchain
- Package manager: `uv` (Python >=3.11,<3.13). No tests, linter, or typecheck config exists; verify by executing code directly.
- `pyproject.toml` sets `[tool.uv] package = false` — the project is NOT installed; code runs from the repo root (cwd is on `sys.path`).

## Running
- Do NOT run `python app/app.py` — it does `from app.api import router`, so running it as a script puts `app/` (not the repo root) on `sys.path` and fails with `'app' is not a package`. `app/app.py` also has no `__main__` block.
- Run from the repo root as a module: `uv run python -m app.app`, or serve with `uv run uvicorn app.app:app --host 0.0.0.0 --port 18081` (matches the Dockerfile CMD). Docker image is `python:3.11-slim` + `uv sync --frozen --no-dev`.

## Database
- Requires an external MySQL. `app/config.py` defaults to `mysql+pymysql://atguigu:Atguigu.123@192.168.200.145:3306/commerce?charset=utf8mb4`, overridable via `DATABASE_URL` (also `APP_HOST`, `APP_PORT`). No `.env` needed here (unlike the atguigu customer-service project).
- `models.py` defines all tables (DeclarativeBase) but nothing calls `create_all` and there is no migration or seed script — schema and data are expected to exist externally. `/health` (in `app/api.py`) checks the DB connection.

## Layout and conventions
- Flat `app/` package: `app.py` (FastAPI app + openapi tags), `api.py` (all routes on one `APIRouter`), `models.py`, `schemas.py`, `database.py` (engine/SessionLocal, `get_db` dependency).
- Docstrings/comments are in Chinese — match that style. All responses are wrapped in `ApiResponse(code, message, data)` from `schemas.py`; 404/409 via `HTTPException`.
- POST `/orders/{order_id}/shipping-reminders` and `/refund-applications` insert rows into `shipping_urge_requests` / `refund_requests`; refunds return 409 if an active one exists.

## Gotcha
- The sibling `../AGENTS.md` (auto-loaded) describes the `atguigu` LangChain customer-service project — do not apply it here (no `atguigu/` package, no `.env` requirement, no `Infrastructure/`).
