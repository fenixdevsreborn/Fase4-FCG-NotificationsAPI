FROM mcr.microsoft.com/dotnet/sdk:10.0 AS build
WORKDIR /app

# Copy solution and project files first for better layer caching
COPY NotificationsAPI.slnx ./
COPY src/Shared/Shared.csproj ./src/Shared/
COPY src/Shared.Contracts/Shared.Contracts.csproj ./src/Shared.Contracts/
COPY src/NotificationsAPI/NotificationsAPI.Worker.csproj ./src/NotificationsAPI/
COPY src/NotificationsAPI.Application/NotificationsAPI.Application.csproj ./src/NotificationsAPI.Application/
COPY src/NotificationsAPI.Domain/NotificationsAPI.Domain.csproj ./src/NotificationsAPI.Domain/
COPY src/NotificationsAPI.Infrastructure/NotificationsAPI.Infrastructure.csproj ./src/NotificationsAPI.Infrastructure/
COPY src/NotificationsAPI.Application.Tests/NotificationsAPI.Application.Tests.csproj ./src/NotificationsAPI.Application.Tests/
COPY src/NotificationsAPI.Domain.Tests/NotificationsAPI.Domain.Tests.csproj ./src/NotificationsAPI.Domain.Tests/

# Restore dependencies
RUN dotnet restore "src/NotificationsAPI/NotificationsAPI.Worker.csproj"

# Copy remaining source code
COPY src ./src

# Build and publish
RUN dotnet publish "src/NotificationsAPI/NotificationsAPI.Worker.csproj" -c Release -o /out --no-restore

FROM mcr.microsoft.com/dotnet/aspnet:10.0
WORKDIR /app

# Install curl for health checks
RUN apt-get update && apt-get install -y curl && rm -rf /var/lib/apt/lists/*

COPY --from=build /out .
EXPOSE 8080
ENV ASPNETCORE_URLS=http://+:8080
HEALTHCHECK --interval=30s --timeout=10s --start-period=60s --retries=3 \
    CMD curl -f http://localhost:8080/health || exit 1

ENTRYPOINT ["dotnet", "NotificationsAPI.Worker.dll"]
