# See https://aka.ms/customizecontainer to learn how to customize your debug container and how Visual Studio uses this Dockerfile to build your images for faster debugging.
# 1. This stage is used when running from VS in fast mode (Default for Debug configuration)
FROM mcr.microsoft.com/dotnet/aspnet:8.0-alpine AS base

USER root
RUN apk add --no-cache icu-libs icu-data-full tzdata

ENV DOTNET_SYSTEM_GLOBALIZATION_INVARIANT=false \
    LANG=en_US.UTF-8 \
    LC_ALL=en_US.UTF-8

WORKDIR /app

EXPOSE 8080
EXPOSE 8081

# 2. This stage is used to build the service project
FROM mcr.microsoft.com/dotnet/sdk:8.0-alpine AS build
ARG BUILD_CONFIGURATION=Release
ARG HTTP_BASIC_ARTIFACTORY_USERNAME
ARG HTTP_BASIC_ARTIFACTORY_PASSWORD

RUN dotnet nuget list source \
 && dotnet nuget disable source nuget.org \
 && dotnet nuget add source https://artifactorygcuk.jfrog.io/artifactory/api/nuget/v3/nuget-snapshot/index.json \
       --name ArtifactoryCloud \
       --username "${HTTP_BASIC_ARTIFACTORY_USERNAME}" \
       --password "${HTTP_BASIC_ARTIFACTORY_PASSWORD}" \
       --store-password-in-clear-text

WORKDIR /src

COPY ["src/Glencore.TacticalAuth.API.Permissions/Glencore.TacticalAuth.API.Permissions.csproj", "Glencore.TacticalAuth.API.Permissions/"]
COPY ["src/Glencore.TacticalAuth.API.Permissions.Abstractions/Glencore.TacticalAuth.API.Permissions.Abstractions.csproj", "Glencore.TacticalAuth.API.Permissions.Abstractions/"]
COPY ["src/Glencore.TacticalAuth.API.Permissions.Database.Sql/Glencore.TacticalAuth.API.Permissions.Database.Sql.csproj", "Glencore.TacticalAuth.API.Permissions.Database.Sql/"]
COPY ["src/Glencore.TacticalAuth.API.Permissions.LimitedMemoryCache/Glencore.TacticalAuth.API.Permissions.LimitedMemoryCache.csproj", "Glencore.TacticalAuth.API.Permissions.LimitedMemoryCache/"]
COPY ["src/Glencore.TacticalAuth.API.Permissions.Database.Sybase/Glencore.TacticalAuth.API.Permissions.Database.Sybase.csproj", "Glencore.TacticalAuth.API.Permissions.Database.Sybase/"]
COPY ["src/Glencore.TacticalAuth.API.Permissions.Ingestion.Service/Glencore.TacticalAuth.API.Permissions.Ingestion.Service.csproj", "Glencore.TacticalAuth.API.Permissions.Ingestion.Service/"]

RUN dotnet restore "Glencore.TacticalAuth.API.Permissions/Glencore.TacticalAuth.API.Permissions.csproj"

COPY src/ .

RUN dotnet publish "Glencore.TacticalAuth.API.Permissions/Glencore.TacticalAuth.API.Permissions.csproj" \
    -c $BUILD_CONFIGURATION \
    -o /app/publish \
    /p:UseAppHost=false \
    --no-restore

# 3. This stage is used in production or when running from VS in regular mode (Default when not using the Debug configuration)
FROM base AS final

WORKDIR /app
COPY --from=build /app/publish .

USER $APP_UID 
ENTRYPOINT ["dotnet", "Glencore.TacticalAuth.API.Permissions.dll"]