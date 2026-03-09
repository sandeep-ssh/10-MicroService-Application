FROM mcr.microsoft.com/dotnet/sdk:8.0 AS builder

WORKDIR /src

RUN apt-get update && apt-get install -y wget && rm -rf /var/lib/apt/lists/* \
    && wget -qO/bin/grpc_health_probe \
    https://github.com/grpc-ecosystem/grpc-health-probe/releases/download/v0.4.19/grpc_health_probe-linux-arm64 \
    && chmod +x /bin/grpc_health_probe

COPY cartservice.csproj .
RUN dotnet restore

COPY . .
RUN dotnet publish -c Release -o /app/publish --no-restore

FROM mcr.microsoft.com/dotnet/aspnet:8.0

WORKDIR /app

RUN apt-get update && apt-get install -y wget && rm -rf /var/lib/apt/lists/* \
    && wget -qO/bin/grpc_health_probe \
    https://github.com/grpc-ecosystem/grpc-health-probe/releases/download/v0.4.19/grpc_health_probe-linux-arm64 \
    && chmod +x /bin/grpc_health_probe

RUN useradd --uid 1000 --create-home appuser

COPY --from=builder /app/publish .

EXPOSE 7070

ENV ASPNETCORE_URLS=http://+:7070 \
    DOTNET_EnableDiagnostics=0

USER appuser

ENTRYPOINT ["dotnet", "cartservice.dll"]