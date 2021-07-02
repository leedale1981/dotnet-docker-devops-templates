FROM mcr.microsoft.com/dotnet/core/sdk:3.1 AS build-env
WORKDIR /app

ARG FEED_URL
ARG USERNAME
ARG SHARED_FEED_URL
ARG PAT

ENV Package=1

# download and install latest credential provider. Not required after https://github.com/dotnet/dotnet-docker/issues/878
RUN wget -qO- https://raw.githubusercontent.com/Microsoft/artifacts-credprovider/master/helpers/installcredprovider.sh | bash

# Optional: Sometimes the http client hangs because of a .NEt issue. Setting this in dockerfile helps 
ENV DOTNET_SYSTEM_NET_HTTP_USESOCKETSHTTPHANDLER=0

# Environment variable to enable seesion token cache. More on this here: https://github.com/Microsoft/artifacts-credprovider#help
ENV NUGET_CREDENTIALPROVIDER_SESSIONTOKENCACHE_ENABLED true

# Environment variable for adding endpoint credentials. More on this here: https://github.com/Microsoft/artifacts-credprovider#help
# Add "FEED_URL" AND "PAT" using --build-arg in docker build step. "endpointCredentials" field is an array, you can add multiple endpoint configurations.
# Make sure that you *do not* hard code the "PAT" here. That is a sensitive information and must not be checked in.
ENV VSS_NUGET_EXTERNAL_FEED_ENDPOINTS {\"endpointCredentials\": [{\"endpoint\":\"${FEED_URL}\", \"username\":\"${USERNAME}\", \"password\":\"${PAT}\"}]}

# Copy csproj and restore as distinct layers
COPY . ./
RUN dotnet restore My.Web.Api/My.Web.Api.csproj --source $FEED_URL --source $SHARED_FEED_URL --source "https://api.nuget.org/v3/index.json"

# Copy everything else and build
RUN dotnet publish My.Web.Api/My.Web.Api.csproj -c Release -o out

# Build runtime image
FROM mcr.microsoft.com/dotnet/core/aspnet:3.1
WORKDIR /app

COPY --from=build-env /app/out .

ARG DATABASE_CONNECTIONSTRING
ARG LOGGING_DATABASE_CONNECTIONSTRING
ARG CLIENT_URI
ARG AZUREAD_CLIENTID
ARG AZUREAD_DOMAIN
ARG AZUREAD_INSTANCE
ARG AZUREAD_TENANTID

ENV ConnectionStrings__Enduring=$DATABASE_CONNECTIONSTRING \
    ConnectionStrings__Logging=$LOGGING_DATABASE_CONNECTIONSTRING \
    ClientUri=$CLIENT_URI \
    ASPNETCORE_URLS=http://0.0.0.0:8000;https://0.0.0.0:8001 \
    AzureAd__ClientId=$AZUREAD_CLIENTID \
    AzureAd__Domain=$AZUREAD_DOMAIN \
    AzureAd__Instance=$AZUREAD_INSTANCE \
    AzureAd__TenantId=$AZUREAD_TENANTID

ENTRYPOINT ["dotnet", "My.Web.Api.dll"]