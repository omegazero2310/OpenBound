# Build stage
FROM mcr.microsoft.com/dotnet/sdk:10.0 AS build
WORKDIR /src

# Copy solution and project files
COPY ["OpenBound.sln", "./"]
COPY ["OpenBound Login Server/OpenBound Login Server.csproj", "OpenBound Login Server/"]
COPY ["OpenBound Network Object Library/OpenBound Network Object Library.csproj", "OpenBound Network Object Library/"]

# Restore dependencies
RUN dotnet restore "OpenBound Login Server/OpenBound Login Server.csproj"

# Copy source code
COPY ["OpenBound Login Server/", "OpenBound Login Server/"]
COPY ["OpenBound Network Object Library/", "OpenBound Network Object Library/"]

# Build the project
WORKDIR "/src/OpenBound Login Server"
RUN dotnet build "OpenBound Login Server.csproj" -c Release -o /app/build

# Publish stage
FROM build AS publish
RUN dotnet publish "OpenBound Login Server.csproj" -c Release -o /app/publish /p:UseAppHost=false

# Runtime stage
FROM mcr.microsoft.com/dotnet/runtime:10.0 AS final
WORKDIR /app
EXPOSE 8022

# Copy published files
COPY --from=publish /app/publish .

ENTRYPOINT ["dotnet", "OpenBound Login Server.dll"]
