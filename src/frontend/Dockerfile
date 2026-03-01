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
# Builder Stage
# ========================
FROM golang:1.21.3-alpine AS builder

RUN apk add --no-cache ca-certificates git build-base

WORKDIR /src

# Restore dependencies
COPY go.mod go.sum ./
RUN go mod download
COPY . .

# Skaffold passes debug flags if needed
ARG SKAFFOLD_GO_GCFLAGS
RUN go build -gcflags="${SKAFFOLD_GO_GCFLAGS}" -o /go/bin/frontend .

# ========================
# Runtime Stage
# ========================
FROM alpine:3.18 AS release

RUN apk add --no-cache ca-certificates \
    busybox-extras net-tools bind-tools

WORKDIR /src

# Copy binary and assets
COPY --from=builder /go/bin/frontend /src/server
COPY ./templates ./templates
COPY ./static ./static

# Go stack trace setting
ENV GOTRACEBACK=single

EXPOSE 8080
ENTRYPOINT ["/src/server"]