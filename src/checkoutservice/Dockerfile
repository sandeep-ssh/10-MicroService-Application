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

FROM golang:1.21-alpine AS builder

RUN apk add --no-cache ca-certificates git build-base wget

WORKDIR /src

RUN wget -qO/go/bin/grpc_health_probe \
    https://github.com/grpc-ecosystem/grpc-health-probe/releases/download/v0.4.19/grpc_health_probe-linux-arm64 \
    && chmod +x /go/bin/grpc_health_probe

COPY go.mod go.sum ./
RUN go mod download

COPY . .

ARG SKAFFOLD_GO_GCFLAGS
RUN go build -gcflags="${SKAFFOLD_GO_GCFLAGS}" -o /go/bin/checkoutservice .

FROM alpine:3.18

RUN apk add --no-cache ca-certificates

WORKDIR /src

COPY --from=builder /go/bin/checkoutservice /src/checkoutservice
COPY --from=builder /go/bin/grpc_health_probe /bin/grpc_health_probe

EXPOSE 5050

ENTRYPOINT ["/src/checkoutservice"]