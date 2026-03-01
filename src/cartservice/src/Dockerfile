# ========================
# Build Stage
# ========================
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS builder
WORKDIR /src

COPY cartservice.csproj .
RUN dotnet restore

COPY . .

# 🔥 Remove runtime-specific publish
RUN dotnet publish -c Release -o /app/publish --no-restore


# ========================
# Runtime Stage
# ========================
FROM mcr.microsoft.com/dotnet/aspnet:8.0
WORKDIR /app

# Create non-root user
RUN adduser -u 1000 -D appuser

COPY --from=builder /app/publish .

EXPOSE 7070

ENV ASPNETCORE_URLS=http://+:7070 \
    DOTNET_EnableDiagnostics=0

USER appuser

ENTRYPOINT ["dotnet", "cartservice.dll"]