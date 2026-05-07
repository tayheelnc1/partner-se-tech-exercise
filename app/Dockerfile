# syntax=docker/dockerfile:1

# ---- build stage ----
FROM cgr.dev/chainguard/python:latest-dev AS build

USER root

WORKDIR /app

COPY requirements.txt .

RUN python -m venv --copies /opt/venv

RUN --mount=type=secret,id=cg_username \
    --mount=type=secret,id=cg_password \
    USERNAME=$(sed 's|/|%2F|g' /run/secrets/cg_username) && \
    PASSWORD=$(cat /run/secrets/cg_password) && \
    /opt/venv/bin/pip install --no-cache-dir \
      --index-url "https://${USERNAME}:${PASSWORD}@libraries.cgr.dev/python/simple/" \
      -r requirements.txt

# ---- runtime stage ----
FROM cgr.dev/chainguard/python:latest

COPY --from=build --chown=nonroot:nonroot /opt/venv /opt/venv

WORKDIR /app
COPY --chown=nonroot:nonroot . .

ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1
ENV PATH="/opt/venv/bin:$PATH"

USER nonroot

EXPOSE 8000

ENTRYPOINT ["/opt/venv/bin/gunicorn"]
CMD ["--bind", "0.0.0.0:8000", "--workers", "2", "app:app"]
