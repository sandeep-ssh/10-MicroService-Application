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
FROM golang:1.21-alpine AS builder

RUN apk add --no-cache ca-certificates git build-base

WORKDIR /src

# Restore dependencies first (better caching)
COPY go.mod go.sum ./
RUN go mod download

COPY . .

# Ensure static binary for alpine
ENV CGO_ENABLED=0

# Skaffold debug flags (optional)
ARG SKAFFOLD_GO_GCFLAGS

RUN GOOS=linux GOARCH=$(go env GOARCH) \
    go build -gcflags="${SKAFFOLD_GO_GCFLAGS}" \
    -o /checkoutservice .


# ========================
# Runtime Stage
# ========================
FROM alpine:3.18

RUN apk add --no-cache ca-certificates

WORKDIR /app

COPY --from=builder /checkoutservice .

ENV GOTRACEBACK=single

EXPOSE 5050

ENTRYPOINT ["./checkoutservice"]