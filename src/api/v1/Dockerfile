# The build context is the repo root (because of model/ and uv.lock):
#   docker build -f src/api/v1/Dockerfile -t api-v1 .
FROM python:3.13-slim

# Install uv: copy the binaries from the official uv image
# (version pinned to match the local uv used to create uv.lock)
COPY --from=ghcr.io/astral-sh/uv:0.13.1 /uv /uvx /bin/

# uv settings + put the .venv on PATH (no activation needed)
ENV UV_COMPILE_BYTECODE=1 \
    UV_LINK_MODE=copy \
    UV_PYTHON_DOWNLOADS=never \
    PATH="/app/.venv/bin:$PATH"

WORKDIR /app

# Install dependencies first so that this layer remains cached,
# as long as pyproject.toml/uv.lock does not change
COPY pyproject.toml uv.lock .python-version ./
RUN uv sync --frozen --no-dev --no-install-project

# main.py loads the model from its own directory
COPY src/api/v1/main.py .
COPY model/model.joblib .

EXPOSE 8000

CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
