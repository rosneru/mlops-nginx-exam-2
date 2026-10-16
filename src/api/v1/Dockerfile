# Build-Kontext ist das Repo-Root (wegen model/ und uv.lock):
#   docker build -f src/api/v1/Dockerfile -t api-v1 .
FROM python:3.13-slim

COPY --from=ghcr.io/astral-sh/uv:0.13.1 /uv /uvx /bin/

ENV UV_COMPILE_BYTECODE=1 \
    UV_LINK_MODE=copy \
    UV_PYTHON_DOWNLOADS=never \
    PATH="/app/.venv/bin:$PATH"

WORKDIR /app

# Abhängigkeiten zuerst installieren, damit dieser Layer gecacht bleibt,
# solange sich pyproject.toml/uv.lock nicht ändern
COPY pyproject.toml uv.lock .python-version ./
RUN uv sync --frozen --no-dev --no-install-project

# main.py lädt das Modell aus seinem eigenen Verzeichnis
COPY src/api/v1/main.py .
COPY model/model.joblib .

EXPOSE 8000

CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
