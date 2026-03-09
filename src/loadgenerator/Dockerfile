# ========================
# Base Image
# ========================
FROM python:3.11-slim AS base

# ========================
# Builder Stage
# ========================
FROM base AS builder
WORKDIR /install

RUN apt-get update && apt-get install -y \
    build-essential \
    gcc \
    python3-dev \
    libffi-dev \
    libssl-dev \
    curl \
    && rm -rf /var/lib/apt/lists/*

RUN pip install --upgrade pip

COPY requirements.txt .
RUN pip install --prefix="/install" -r requirements.txt

# ========================
# Runtime Stage
# ========================
FROM base
WORKDIR /loadgen

COPY --from=builder /install /usr/local
COPY locustfile.py .

ENV GEVENT_SUPPORT=True
ENTRYPOINT locust --host=http://${FRONTEND_ADDR} --headless -u ${USERS:-10} -r 1