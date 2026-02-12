# How to Minimize Docker Image Size

## 1. Use Multi-Stage Build
Multi-stage builds allow you to use a "heavy" image (with compilers and tools) to build your app, then copy only the final binary or bundle to a "light" production image.
```dockerfile
FROM golang:1.16-alpine AS builder
WORKDIR /app
COPY . .
RUN go build -o main main.go

FROM alpine:latest
WORKDIR /app
COPY --from=builder /app/main .
CMD ["./main"]
```

```dockerfile
# STAGE 1: THE BUILDER (Heavy)
# Contains the SDK, compilers, and source code.
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build-stage
WORKDIR /src
COPY . .
RUN dotnet publish -c Release -o /app/publish

# STAGE 2: THE RUNTIME (Light)
# Contains ONLY the .NET runtime. No source code, no compilers.
FROM mcr.microsoft.com/dotnet/aspnet:8.0-alpine
WORKDIR /app
# We only take the compiled DLLs from the build-stage
COPY --from=build-stage /app/publish .
ENTRYPOINT ["dotnet", "MyApi.dll"]
```

```dockerfile
```

## 2. Use .dockerignore
This prevents large, unnecessary files from being sent to the Docker daemon during the build process, which reduces the "build context" size and prevents accidental secrets leakage.
```dockerfile
# .dockerignore
.git
node_modules
npm-debug.log
dist
Dockerfile
.dockerignore
.env
```
## 3. Use Lightweight Base Images
The base image is the foundation. Starting with ubuntu or node:latest adds hundreds of MBs of OS utilities you likely don't need in production.
Size Comparison:

node:latest: ~1.1 GB

node:slim: ~200 MB

node:alpine: ~50 MB

gcr.io/distroless/nodejs20: ~25 MB

```dockerfile
# Instead of: FROM python:3.11
FROM python:3.11-slim
```
## 4. Cache Dependencies
Docker caches each layer. If you copy your entire source code before running install, any tiny code change will force Docker to re-download all dependencies.

```dockerfile
# Instead of:
COPY . .
RUN npm install

# Do this:
# Copy ONLY the dependency files first
COPY package.json package-lock.json ./
RUN npm install

# Then copy the rest of the source code
COPY . .
```
## 5. Use Alpine Linux
Alpine is a security-oriented, lightweight Linux distribution based on musl libc and busybox. It is significantly smaller than Debian-based images.

```dockerfile
# A basic Alpine image is only 5MB
FROM alpine:latest
RUN apk add --no-cache curl
```
mcr.microsoft.com/dotnet/aspnet:8.0-alpine
python:3.11-alpine

## 6. Remove Unnecessary Files
Each RUN command creates a layer. If you download a file, extract it, and then delete the zip in a different RUN command, the zip file still exists in the previous layer's history.
```dockerfile

# Instead of:
RUN wget http://example.com/big-tool.tar.gz
RUN tar -xvf big-tool.tar.gz
RUN rm big-tool.tar.gz

# Do this:
RUN wget http://example.com/big-tool.tar.gz && \
    tar -xvf big-tool.tar.gz && \
    rm big-tool.tar.gz && \
    apt-get purge -y --auto-remove wget
```

