# jDisco Docker Quick Reference

One-page cheat sheet for Docker operations.

## Build Commands

### Current Configuration (Debian Buster)
```bash
docker compose build                              # Build image
docker compose run --rm jdisco mvn test          # Run tests
docker compose run --rm jdisco mvn install       # Build JARs
docker compose run --rm jdisco sh -c "cp target/*.jar /artifacts/"  # Extract
```

### Temurin Configuration (Recommended)
```bash
docker compose -f docker-compose.temurin.yml build           # Build all
docker compose -f docker-compose.temurin.yml build builder   # Build only
docker compose -f docker-compose.temurin.yml build artifact  # Artifact only
docker compose -f docker-compose.temurin.yml build dev       # Dev only
```

## Test Commands

### Current
```bash
docker compose run --rm jdisco mvn test                      # Unit tests
docker compose run --rm jdisco mvn test -X                   # Verbose
```

### Temurin
```bash
docker compose -f docker-compose.temurin.yml run --rm dev mvn test       # Tests
docker compose -f docker-compose.temurin.yml run --rm dev mvn test -X    # Verbose
```

## Artifact Extraction

### Current (Manual)
```bash
docker compose run --rm jdisco mvn clean install
docker compose run --rm jdisco sh -c "cp target/*.jar /artifacts/"
ls -lh artifacts/
```

### Temurin (Automatic)
```bash
docker compose -f docker-compose.temurin.yml up artifact
ls -lh artifacts/
```

## Interactive Development

### Current
```bash
docker compose run --rm jdisco bash
# Inside container:
mvn clean test
mvn compile
exit
```

### Temurin
```bash
docker compose -f docker-compose.temurin.yml run --rm dev bash
# Inside container:
mvn clean test
mvn javadoc:javadoc
exit
```

## One-Liners

### Build Everything (Current)
```bash
docker compose build && \
docker compose run --rm jdisco mvn clean install && \
docker compose run --rm jdisco sh -c "cp target/*.jar /artifacts/"
```

### Build Everything (Temurin)
```bash
docker compose -f docker-compose.temurin.yml build && \
docker compose -f docker-compose.temurin.yml up artifact
```

### Clean and Rebuild
```bash
# Current
docker compose down && docker compose build --no-cache

# Temurin
docker compose -f docker-compose.temurin.yml down && \
docker compose -f docker-compose.temurin.yml build --no-cache
```

## Image Management

### List Images
```bash
docker images | grep jdisco
```

### Remove Images
```bash
docker rmi jdisco-jdisco:latest              # Current
docker rmi jdisco:builder jdisco:artifact    # Temurin
```

### Check Image Size
```bash
docker images jdisco-jdisco --format "{{.Size}}"  # Current
docker images jdisco:artifact --format "{{.Size}}" # Temurin
```

### Prune Old Images
```bash
docker image prune -f                # Remove dangling images
docker builder prune -f              # Remove build cache
```

## Troubleshooting

### View Build Logs
```bash
docker compose build 2>&1 | tee build.log
docker compose -f docker-compose.temurin.yml build 2>&1 | tee build.log
```

### Check Java Version
```bash
docker compose run --rm jdisco java -version
docker compose -f docker-compose.temurin.yml run --rm dev java -version
```

### Check Maven Version
```bash
docker compose run --rm jdisco mvn --version
docker compose -f docker-compose.temurin.yml run --rm dev mvn --version
```

### Inspect Container
```bash
docker compose run --rm jdisco ls -lh target/
docker compose -f docker-compose.temurin.yml run --rm dev ls -lh target/
```

### Fix Artifact Permissions
```bash
sudo chown -R $(whoami):$(whoami) artifacts/
# Or
docker compose run --rm --user $(id -u):$(id -g) jdisco mvn install
```

## CI/CD Snippets

### GitHub Actions
```yaml
- name: Build
  run: docker compose -f docker-compose.temurin.yml build builder

- name: Test
  run: docker compose -f docker-compose.temurin.yml run --rm dev mvn test

- name: Extract
  run: docker compose -f docker-compose.temurin.yml up artifact

- name: Upload
  uses: actions/upload-artifact@v4
  with:
    name: jdisco-jars
    path: artifacts/*.jar
```

### GitLab CI
```yaml
build:
  script:
    - docker compose -f docker-compose.temurin.yml build builder
    - docker compose -f docker-compose.temurin.yml run --rm dev mvn test
    - docker compose -f docker-compose.temurin.yml up artifact
  artifacts:
    paths:
      - artifacts/*.jar
```

## Environment Variables

### Enable BuildKit (Faster Builds)
```bash
export DOCKER_BUILDKIT=1
export COMPOSE_DOCKER_CLI_BUILD=1
```

### Set Maven Options
```bash
docker compose run --rm \
  -e MAVEN_OPTS="-Xmx512m" \
  jdisco mvn clean install
```

## Advanced Usage

### Build Specific Stage (Temurin)
```bash
docker build --target builder -t jdisco:builder -f Dockerfile.temurin .
docker build --target artifact -t jdisco:artifact -f Dockerfile.temurin .
docker build --target dev -t jdisco:dev -f Dockerfile.temurin .
```

### Extract Single JAR
```bash
docker run --rm -v $(pwd)/artifacts:/out jdisco:artifact \
  sh -c "cp /artifacts/jdisco-1.2.0.jar /out/"
```

### Run Maven Goal
```bash
# Current
docker compose run --rm jdisco mvn javadoc:javadoc

# Temurin
docker compose -f docker-compose.temurin.yml run --rm dev mvn javadoc:javadoc
```

### Build with Custom Maven Settings
```bash
docker compose run --rm \
  -v $(pwd)/settings.xml:/root/.m2/settings.xml \
  jdisco mvn clean install
```

## Quick Aliases (Add to ~/.bashrc or ~/.zshrc)

```bash
# Current configuration
alias jdisco-build="docker compose build"
alias jdisco-test="docker compose run --rm jdisco mvn test"
alias jdisco-install="docker compose run --rm jdisco mvn install"
alias jdisco-shell="docker compose run --rm jdisco bash"

# Temurin configuration
alias jdisco-t-build="docker compose -f docker-compose.temurin.yml build"
alias jdisco-t-test="docker compose -f docker-compose.temurin.yml run --rm dev mvn test"
alias jdisco-t-artifact="docker compose -f docker-compose.temurin.yml up artifact"
alias jdisco-t-shell="docker compose -f docker-compose.temurin.yml run --rm dev bash"
```

## File Locations

```
jdisco/
├── Dockerfile                      # Current config
├── Dockerfile.temurin              # Temurin config
├── docker-compose.yml              # Current compose
├── docker-compose.temurin.yml      # Temurin compose
├── .dockerignore                   # Build context filter
├── artifacts/                      # Output directory
│   ├── jdisco-1.2.0.jar           # Main library
│   ├── jdisco-1.2.0-sources.jar   # Sources
│   └── jdisco-1.2.0-javadoc.jar   # Javadoc
├── pom.xml                         # Maven config
└── src/                            # Source code
```

## Common Patterns

### Full Clean Build
```bash
# Remove old artifacts and images
rm -rf artifacts/*.jar
docker compose down
docker compose build --no-cache
docker compose run --rm jdisco mvn clean install
docker compose run --rm jdisco sh -c "cp target/*.jar /artifacts/"
```

### Quick Rebuild (With Cache)
```bash
docker compose build
docker compose run --rm jdisco mvn clean install
docker compose run --rm jdisco sh -c "cp target/*.jar /artifacts/"
```

### Test-Only (No Build)
```bash
docker compose run --rm jdisco mvn test
```

### Build + Test + Extract (One Command)
```bash
# Temurin (recommended)
docker compose -f docker-compose.temurin.yml build && \
docker compose -f docker-compose.temurin.yml run --rm dev mvn test && \
docker compose -f docker-compose.temurin.yml up artifact
```

## Performance Tips

1. **Enable BuildKit:** `export DOCKER_BUILDKIT=1`
2. **Use artifact stage:** Smaller, faster for distribution
3. **Cache Maven deps:** BuildKit cache mounts handle this automatically
4. **Prune regularly:** `docker builder prune` to free space
5. **Use .dockerignore:** Already configured, reduces build context

## Getting Help

```bash
# Docker help
docker --help
docker compose --help

# Maven help in container
docker compose run --rm jdisco mvn --help
docker compose run --rm jdisco mvn dependency:help

# Image inspection
docker inspect jdisco-jdisco:latest
docker history jdisco-jdisco:latest
```

---

**See Also:**
- `DOCKER.md` - Complete documentation
- `DOCKER-SUMMARY.md` - Configuration comparison
- `CLAUDE.md` - Development guidelines
- `.github-workflows-example.yml` - CI/CD examples
