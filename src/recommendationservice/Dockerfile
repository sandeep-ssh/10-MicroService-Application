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

# Base Stage
FROM python:3.10.8-slim AS base

# Builder Stage
FROM base AS builder

RUN apt-get -qq update \
    && apt-get install -y --no-install-recommends wget g++ \
    && rm -rf /var/lib/apt/lists/*

# Install Python dependencies
COPY requirements.txt .
RUN pip install -r requirements.txt

# Runtime Stage
FROM base

# Enable unbuffered logging
ENV PYTHONUNBUFFERED=1

WORKDIR /recommendationservice

# Copy installed packages from builder
COPY --from=builder /usr/local/lib/python3.10/ /usr/local/lib/python3.10/

# Add application code
COPY . .

# Set service port
ENV PORT=8080
EXPOSE 8080

ENTRYPOINT ["python", "recommendation_server.py"]