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
# Build Stage
# ========================
FROM golang:1.21.3-alpine AS builder
RUN apk add --no-cache ca-certificates git build-base

WORKDIR /src

# Restore dependencies
COPY go.mod go.sum ./
RUN go mod download
COPY . .

# Skaffold passes in debug-oriented compiler flags
ARG SKAFFOLD_GO_GCFLAGS
RUN go build -gcflags="${SKAFFOLD_GO_GCFLAGS}" -o /go/bin/shippingservice .

# ========================
# Runtime Stage
# ========================
FROM alpine:3.18
RUN apk add --no-cache ca-certificates

WORKDIR /src
COPY --from=builder /go/bin/shippingservice /src/shippingservice

# Service port
ENV APP_PORT=50051
EXPOSE 50051

# For Go debug traces
ENV GOTRACEBACK=single

ENTRYPOINT ["/src/shippingservice"]