# Stage 1: Build the application
FROM mcr.microsoft.com/dotnet/sdk:9.0 AS build
ARG BUILD_CONFIGURATION=Release
WORKDIR /src

# Copy the solution file and project files first to leverage Docker layer caching.
# This ensures that 'dotnet restore' is only re-run when project dependencies change.
COPY ["DockerApp.sln", "."]
COPY ["API/API.csproj", "API/"]

# Restore dependencies for the entire solution
RUN dotnet restore "DockerApp.sln"

# Copy the rest of the application's source code
COPY . .

# Set the working directory to the API project folder to run the build and publish commands
WORKDIR "/src/API"
RUN dotnet build "API.csproj" -c $BUILD_CONFIGURATION -o /app/build

# Publish the application
FROM build AS publish
WORKDIR "/src/API" # Ensure we are in the correct directory for publish
RUN dotnet publish "API.csproj" -c $BUILD_CONFIGURATION -o /app/publish /p:UseAppHost=false

# Stage 2: Create the final, lean production image
FROM mcr.microsoft.com/dotnet/aspnet:9.0 AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "API.dll"]
