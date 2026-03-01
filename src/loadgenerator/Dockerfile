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

# ========================
# Base & Builder Stage
# ========================
FROM python:3.11-slim AS base

FROM base AS builder

WORKDIR /install

# Copy and install dependencies
COPY requirements.txt .
RUN pip install --prefix="/install" -r requirements.txt

# ========================
# Runtime Stage
# ========================
FROM base

WORKDIR /loadgen

# Copy installed packages
COPY --from=builder /install /usr/local

# Copy application code
COPY locustfile.py .

# Enable gevent support in debugger
ENV GEVENT_SUPPORT=True

# Start Locust in headless mode
ENTRYPOINT ["locust", "--host=http://${FRONTEND_ADDR}", "--headless", "-u", "${USERS:-10}"]