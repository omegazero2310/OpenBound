# Build stage
FROM mcr.microsoft.com/dotnet/sdk:10.0 AS build
WORKDIR /src

# Copy solution and project files
COPY ["OpenBound.sln", "./"]
COPY ["Avatar API/Avatar API.csproj", "Avatar API/"]
COPY ["OpenBound Network Object Library/OpenBound Network Object Library.csproj", "OpenBound Network Object Library/"]

# Restore dependencies
RUN dotnet restore "Avatar API/Avatar API.csproj"

# Copy source code
COPY ["Avatar API/", "Avatar API/"]
COPY ["OpenBound Network Object Library/", "OpenBound Network Object Library/"]

# Build the project
WORKDIR "/src/Avatar API"
RUN dotnet build "Avatar API.csproj" -c Release -o /app/build

# Publish stage
FROM build AS publish
RUN dotnet publish "Avatar API.csproj" -c Release -o /app/publish /p:UseAppHost=false

# Runtime stage
FROM mcr.microsoft.com/dotnet/aspnet:10.0 AS final
WORKDIR /app
EXPOSE 80
EXPOSE 443

# Copy published files
COPY --from=publish /app/publish .

# Set environment variables
ENV ASPNETCORE_URLS=http://+:80

ENTRYPOINT ["dotnet", "Avatar API.dll"]
