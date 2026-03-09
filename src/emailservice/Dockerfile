# Copyright 2020 Google LLC
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#      http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

# Base Image
FROM python:3.11-slim AS base

# Builder Stage
FROM base AS builder

RUN apt-get -qq update \
    && apt-get install -y --no-install-recommends \
        wget g++ gcc python3-dev \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /app
COPY requirements.txt .

# Skip google-cloud-profiler as it doesn't support ARM64
RUN grep -v "google-cloud-profiler" requirements.txt \
    | grep -v "^#" \
    | grep -v "^$" \
    > /tmp/requirements_filtered.txt \
    && pip install --prefix=/install -r /tmp/requirements_filtered.txt

# Runtime Stage
FROM base

RUN apt-get update && apt-get install -y wget && rm -rf /var/lib/apt/lists/*

# Install grpc_health_probe for ARM64
RUN wget -qO/bin/grpc_health_probe \
    https://github.com/grpc-ecosystem/grpc-health-probe/releases/download/v0.4.19/grpc_health_probe-linux-arm64 \
    && chmod +x /bin/grpc_health_probe

ENV PYTHONUNBUFFERED=1
ENV ENABLE_PROFILER=0

WORKDIR /email_server
COPY --from=builder /install /usr/local
COPY . .

EXPOSE 8080
ENTRYPOINT ["python", "email_server.py"]