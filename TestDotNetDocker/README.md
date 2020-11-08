

FROM mcr.microsoft.com/dotnet/core/sdk:3.1 AS build-env
WORKDIR /app

# Copy csproj and restore as distinct layers
COPY *.csproj ./
RUN dotnet restore

# Copy everything else and build
COPY . ./
RUN dotnet publish -c Release -o out

# Build runtime image
FROM mcr.microsoft.com/dotnet/core/aspnet:3.1
WORKDIR /app
COPY --from=build-env /app/out .
ENTRYPOINT ["dotnet", "aspnetapp.dll"]


------------------------------------------------------

FROM mcr.microsoft.com/dotnet/core/sdk:3.1 AS build
COPY . ./app
WORKDIR ./app
RUN dotnet build -c Release -o output


FROM mcr.microsoft.com/dotnet/core/aspnet:3.1 AS runtime
WORKDIR ./app
EXPOSE 5001
COPY --from=build /app/output .
ENTRYPOINT ["dotnet", "TestDotNetDocker.dll"]


# How to run
docker build -t test-dotnet-docker .
docker run -p 5001:80  -e env=prodution test-dotnet-docker

Or

docker-compose up