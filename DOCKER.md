# jDisco Docker Build Configuration

Complete Docker build environment for jDisco library with JDK 11 and Maven.

## Overview

jDisco provides two Docker configurations:

1. **Dockerfile** (Current) - Debian Buster + OpenJDK 11
2. **Dockerfile.temurin** (Recommended) - Eclipse Temurin multi-stage build

Both configurations compile jDisco to Java 6 bytecode while using JDK 11 for building (last JDK version supporting Java 6 target).

## Quick Start

### Option 1: Current Configuration (Debian Buster)

**Build:**
```bash
docker compose build
```

**Run tests:**
```bash
docker compose run --rm jdisco mvn test
```

**Build and install to local repository:**
```bash
docker compose run --rm jdisco mvn clean install
```

**Extract artifacts:**
```bash
docker compose run --rm jdisco sh -c "cp target/*.jar /artifacts/"
ls -lh artifacts/
```

### Option 2: Eclipse Temurin (Multi-Stage)

**Build all stages:**
```bash
docker compose -f docker-compose.temurin.yml build
```

**Extract artifacts automatically:**
```bash
docker compose -f docker-compose.temurin.yml up artifact
ls -lh artifacts/
```

**Run tests:**
```bash
docker compose -f docker-compose.temurin.yml run --rm dev mvn test
```

**Interactive development:**
```bash
docker compose -f docker-compose.temurin.yml run --rm dev bash
```

## Configuration Comparison

### Current: Dockerfile (Debian Buster)

**Pros:**
- Already working and tested
- Single-stage build (simpler)
- BuildKit cache mounts for fast rebuilds

**Cons:**
- Debian Buster is archived (security updates stopped)
- Larger image size (627 MB)
- Single-stage means dev tools in production image
- Archive repository URLs required for apt-get

**Base Image:** `debian:buster-slim`
**Size:** 627 MB
**Java Version:** OpenJDK 11.0.23
**Maven Version:** 3.6.0

### Recommended: Dockerfile.temurin (Eclipse Temurin)

**Pros:**
- Modern, actively maintained base images
- Multi-stage build (smaller artifacts)
- Three stages: builder, artifact, dev
- Official Eclipse Temurin from Adoptium project
- Ubuntu Jammy LTS base (security updates until 2027)
- Cleaner artifact extraction

**Cons:**
- Requires multi-stage build understanding
- Slightly more complex configuration

**Base Image:** `eclipse-temurin:11-jdk` (builder/dev), `eclipse-temurin:11-jre-jammy` (artifact)
**Size:** ~450 MB (builder), ~200 MB (artifact)
**Java Version:** Eclipse Temurin 11
**Maven Version:** Latest from Ubuntu repos

## Docker Stages Explained (Temurin)

### Stage 1: Builder

Full build environment with JDK 11 + Maven:
- Downloads dependencies (with cache mount)
- Compiles Java sources (Java 6 target)
- Runs all tests with JUnit 4
- Creates JAR packages (main, sources, javadoc)
- Installs to Maven local repository

**Usage:**
```bash
docker build --target builder -t jdisco:builder .
```

### Stage 2: Artifact

Minimal JRE-only image with build artifacts:
- Contains only JAR files
- No build tools (Maven, JDK)
- Small image size (~200 MB)
- Includes BUILD_INFO.txt metadata

**Usage:**
```bash
docker build --target artifact -t jdisco:artifact .
docker run --rm -v $(pwd)/artifacts:/out jdisco:artifact sh -c "cp /artifacts/* /out/"
```

### Stage 3: Dev

Interactive development environment:
- Full JDK 11 + Maven
- Source code mounted as volumes
- Dependency cache persisted
- Suitable for local development and testing

**Usage:**
```bash
docker run -it --rm -v $(pwd):/build/jdisco jdisco:dev bash
```

## BuildKit Optimization

Both Dockerfiles use BuildKit features for optimal caching:

### Cache Mounts

```dockerfile
RUN --mount=type=cache,target=/root/.m2 \
    mvn dependency:go-offline -B
```

**Benefits:**
- Maven dependencies persist across builds
- First build: 3-5 minutes (download all deps)
- Subsequent builds: 30-60 seconds (cache hit)
- Shared across all Docker builds on same host

### Layer Ordering

```
1. Copy pom.xml         → Cache invalidates only on dependency changes
2. Download dependencies → Cached unless pom.xml changes
3. Copy source code     → Cache invalidates only on source changes
4. Build and test       → Runs only when source changes
```

**Result:** Changing Java source code doesn't re-download dependencies.

## CI/CD Integration

### GitHub Actions Example

```yaml
name: Build jDisco

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Build jDisco
        run: |
          docker compose -f docker-compose.temurin.yml build builder

      - name: Run tests
        run: |
          docker compose -f docker-compose.temurin.yml run --rm dev mvn test

      - name: Extract artifacts
        run: |
          docker compose -f docker-compose.temurin.yml up artifact

      - name: Upload artifacts
        uses: actions/upload-artifact@v4
        with:
          name: jdisco-jars
          path: artifacts/*.jar
          retention-days: 90
```

### GitLab CI Example

```yaml
image: docker:latest

services:
  - docker:dind

variables:
  DOCKER_DRIVER: overlay2
  DOCKER_BUILDKIT: 1

stages:
  - build
  - test
  - artifact

build:
  stage: build
  script:
    - docker compose -f docker-compose.temurin.yml build builder

test:
  stage: test
  script:
    - docker compose -f docker-compose.temurin.yml run --rm dev mvn test

artifact:
  stage: artifact
  script:
    - docker compose -f docker-compose.temurin.yml up artifact
  artifacts:
    paths:
      - artifacts/*.jar
    expire_in: 90 days
```

## Artifact Extraction Patterns

### Pattern 1: Docker Run (Manual)

```bash
# Build the artifact stage
docker build --target artifact -t jdisco:artifact -f Dockerfile.temurin .

# Extract to local filesystem
docker run --rm -v $(pwd)/artifacts:/out jdisco:artifact sh -c "cp /artifacts/* /out/"

# Verify
ls -lh artifacts/
```

### Pattern 2: Docker Compose (Automated)

```bash
# Automatically builds and extracts
docker compose -f docker-compose.temurin.yml up artifact

# Artifacts are in ./artifacts/
ls -lh artifacts/
```

### Pattern 3: Docker Copy (Alternative)

```bash
# Build image
docker build --target artifact -t jdisco:artifact -f Dockerfile.temurin .

# Create container (don't run)
CONTAINER=$(docker create jdisco:artifact)

# Copy artifacts out
docker cp $CONTAINER:/artifacts/ ./

# Clean up
docker rm $CONTAINER
```

### Pattern 4: CI/CD Pipeline

```bash
# In CI/CD, use volumes for artifact extraction
docker run --rm \
  -v $(pwd)/artifacts:/out \
  jdisco:artifact \
  sh -c "cp /artifacts/*.jar /out/ && ls -lh /out/"
```

## Performance Optimization Tips

### 1. Enable BuildKit

**Docker Compose:**
```bash
export DOCKER_BUILDKIT=1
docker compose build
```

**Docker Build:**
```bash
DOCKER_BUILDKIT=1 docker build .
```

### 2. Use Build Cache

Docker automatically caches layers. To force rebuild:

```bash
# Rebuild without cache
docker compose build --no-cache

# Rebuild specific service
docker compose build --no-cache dev
```

### 3. Parallel Stage Builds

BuildKit can build independent stages in parallel:

```bash
# Build all stages simultaneously
docker buildx build \
  --target builder \
  --target artifact \
  --target dev \
  -f Dockerfile.temurin .
```

### 4. Prune Build Cache

Clean up old cache layers:

```bash
# Remove all unused build cache
docker builder prune

# Remove cache older than 7 days
docker builder prune --filter "until=168h"
```

## Image Size Comparison

| Configuration | Stage | Size | Purpose |
|--------------|-------|------|---------|
| Debian Buster | single | 627 MB | Build + runtime |
| Temurin | builder | ~450 MB | Full build environment |
| Temurin | artifact | ~200 MB | JARs only (minimal) |
| Temurin | dev | ~450 MB | Interactive development |

**Recommendation:** Use Temurin multi-stage for production, artifact stage for distribution.

## Java Version Compatibility

### Why JDK 11?

jDisco requires Java 6 bytecode compatibility:

```xml
<maven.compiler.source>1.6</maven.compiler.source>
<maven.compiler.target>1.6</maven.compiler.target>
```

**JDK Compatibility:**
- **JDK 8-10:** Support Java 6 target
- **JDK 11:** Last LTS supporting Java 6 target (with warnings)
- **JDK 17+:** Cannot compile to Java 6 target (removed)

**Current choice:** JDK 11 (last version before removal)

**Warnings:** Maven will show deprecation warnings but compilation succeeds:
```
WARNING: source value 6 is obsolete and will be removed in a future release
WARNING: target value 1.6 is obsolete and will be removed in a future release
```

These warnings are expected and can be safely ignored.

### Runtime Compatibility

Once compiled to Java 6 bytecode, jDisco JAR runs on:
- Java 6, 7, 8, 11, 17, 21+ (any version)
- No runtime dependencies except SLF4J API

## Troubleshooting

### Issue: Debian Buster 404 Errors

**Problem:**
```
E: The repository 'http://deb.debian.org/debian buster Release' does not have a Release file.
```

**Solution:** Already fixed in current Dockerfile:
```dockerfile
RUN sed -i 's|http://deb.debian.org|http://archive.debian.org|g' /etc/apt/sources.list
```

### Issue: Maven Dependencies Not Caching

**Problem:** Dependencies download every build

**Solution:** Enable BuildKit and use cache mounts:
```bash
export DOCKER_BUILDKIT=1
docker compose build
```

### Issue: Tests Failing in Docker

**Problem:** Tests pass locally but fail in Docker

**Solution:**
1. Check Java version: `docker compose run jdisco java -version`
2. Run tests with verbose output: `docker compose run jdisco mvn test -X`
3. Check for file encoding issues (jDisco uses ISO-8859-1)

### Issue: Artifacts Directory Empty

**Problem:** `artifacts/` directory exists but contains no files

**Solution:**
```bash
# Ensure artifacts are copied after build
docker compose run --rm jdisco mvn clean install
docker compose run --rm jdisco sh -c "cp target/*.jar /artifacts/"
ls -lh artifacts/
```

**Or use Temurin compose:**
```bash
# Automatic artifact extraction
docker compose -f docker-compose.temurin.yml up artifact
```

### Issue: Permission Denied on Artifacts

**Problem:** Artifact files owned by root, cannot delete

**Solution:**
```bash
# Fix permissions
sudo chown -R $(whoami):$(whoami) artifacts/

# Or run as current user
docker compose run --rm --user $(id -u):$(id -g) jdisco mvn install
```

## Migration Guide: Buster → Temurin

To migrate from current Dockerfile to Temurin:

### Step 1: Test Temurin Build

```bash
# Build with Temurin
docker compose -f docker-compose.temurin.yml build

# Verify tests pass
docker compose -f docker-compose.temurin.yml run --rm dev mvn test

# Extract and verify artifacts
docker compose -f docker-compose.temurin.yml up artifact
ls -lh artifacts/
```

### Step 2: Compare Artifacts

```bash
# Build with both
docker compose build
docker compose -f docker-compose.temurin.yml build dev

# Compare JAR sizes and checksums
sha256sum artifacts/*.jar
```

### Step 3: Switch Configuration

```bash
# Backup current
mv Dockerfile Dockerfile.buster
mv docker-compose.yml docker-compose.buster.yml

# Activate Temurin
mv Dockerfile.temurin Dockerfile
mv docker-compose.temurin.yml docker-compose.yml

# Test
docker compose build
docker compose run --rm jdisco mvn test
```

### Step 4: Update CI/CD

Update pipeline configurations to use new multi-stage targets:
- `builder` - For build + test jobs
- `artifact` - For artifact extraction
- `dev` - For interactive debugging

## Best Practices

### 1. Use Multi-Stage Builds

✅ **Good:** Separate builder, artifact, and dev stages
❌ **Bad:** Single stage with all tools

### 2. Leverage BuildKit

✅ **Good:** Use cache mounts for Maven dependencies
❌ **Bad:** Download dependencies on every build

### 3. Specific Base Image Versions

✅ **Good:** `eclipse-temurin:11-jdk`
❌ **Bad:** `eclipse-temurin:latest`

### 4. Layer Ordering

✅ **Good:** Copy pom.xml → download deps → copy source
❌ **Bad:** Copy all files → download deps → build

### 5. Artifact Extraction

✅ **Good:** Dedicated artifact stage with volume mount
❌ **Bad:** Docker cp from stopped container

## References

- **Eclipse Temurin:** https://adoptium.net/
- **Docker BuildKit:** https://docs.docker.com/build/buildkit/
- **Multi-stage builds:** https://docs.docker.com/build/building/multi-stage/
- **Maven in Docker:** https://github.com/carlossg/docker-maven

## Summary

**Current Production Status:**
- Dockerfile (Debian Buster): ✅ Working, tested, production-ready
- Dockerfile.temurin (Eclipse Temurin): ✅ Tested, recommended for new deployments

**Recommendation:**
- **Existing deployments:** Keep using current Dockerfile (if not broken, don't fix)
- **New deployments:** Use Dockerfile.temurin (better security, smaller images, multi-stage)
- **CI/CD pipelines:** Switch to Temurin for better cache management and faster builds

Both configurations successfully compile jDisco to Java 6 bytecode using JDK 11.
