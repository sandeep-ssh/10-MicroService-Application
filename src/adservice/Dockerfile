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

FROM gradle:8.5-jdk21 AS builder

WORKDIR /app

COPY ["build.gradle", "gradlew", "./"]
COPY gradle gradle
RUN chmod +x gradlew
RUN ./gradlew downloadRepos
COPY . .
RUN chmod +x gradlew
RUN ./gradlew installDist

RUN wget -qO/bin/grpc_health_probe \
    https://github.com/grpc-ecosystem/grpc-health-probe/releases/download/v0.4.19/grpc_health_probe-linux-arm64 \
    && chmod +x /bin/grpc_health_probe

FROM eclipse-temurin:21-jre

RUN apt-get update && apt-get install -y wget && rm -rf /var/lib/apt/lists/* \
    && wget -qO/bin/grpc_health_probe \
    https://github.com/grpc-ecosystem/grpc-health-probe/releases/download/v0.4.19/grpc_health_probe-linux-arm64 \
    && chmod +x /bin/grpc_health_probe

WORKDIR /app

COPY --from=builder /app .

EXPOSE 9555

ENTRYPOINT ["/app/build/install/hipstershop/bin/AdService"]