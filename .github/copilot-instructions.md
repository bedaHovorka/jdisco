# GitHub Copilot Instructions for jDisco

This file provides guidance to GitHub Copilot when working with code in this repository.

## Project Overview

**jDisco** is a Java library for discrete-event and continuous simulation, originally written by Keld Helsgaun (Roskilde University, Denmark) and released into the public domain (2001-2004). This is a **legacy library** extracted from the interlockSim project in 2025 as a standalone Maven library.

## Critical: Conservative Approach Required

**This is a legacy library from 2004 that has been in use for 20+ years. Be extremely conservative when modifying code.**

### Strict Rules for Modifications

1. **Do not touch Java code unless explicitly requested** - This is working, stable code
2. **Preserve Java 6 compatibility** - No Java 7+ features (no try-with-resources, no diamond operator, no lambdas, etc.)
3. **Maintain encoding** - Source files are ISO-8859-1 encoded (some contain Latin-1 characters)
4. **No modernization** - Do not refactor working code to "improve" it
5. **Tests required** - Any modifications must be covered by tests

### Allowed Modifications

- Fixing actual bugs (only if absolutely necessary)
- Fixing very critical security issues
- Adding tests to verify existing behavior
- Documentation improvements
- Build system updates (Maven plugins, Docker)
- Adding SLF4J logging dependencies and replacing System.out.println with logger calls
- Adding javax.inject for Singleton annotations (only if absolutely necessary)
- Adding new features only if absolutely necessary (with tests)
- Test-scoped dependencies are allowed; other dependencies require explicit justification

## Build System

This project uses Apache Maven with Java 6 source/target compatibility.

### Common Build Commands

```bash
# Clean build
mvn clean compile

# Run tests
mvn test

# Package JAR
mvn package

# Install to local Maven repository
mvn install

# Generate JavaDoc
mvn javadoc:javadoc

# Full build with tests
mvn clean install
```

### Docker Setup (Recommended)

Docker support eliminates the need to install JDK 6 or Maven on the host machine.

```bash
# Build library
docker compose build

# Run tests
docker compose run jdisco mvn test

# Install to artifacts directory
docker compose run jdisco mvn install

# Clean build
docker compose run jdisco mvn clean install
```

Build artifacts are available in `./artifacts/` directory on the host.

## Code Style

Follow `.editorconfig` configuration:

- **Java files**: tabs (width 4), max line length 120
- **Source encoding**: ISO-8859-1 (some files contain Latin-1 characters)
- **Documentation**: UTF-8
- **Line endings**: LF

## Architecture

### Core Components

- **Process** (`Process.java`) - Discrete event simulation core with coroutine-based implementation
- **Continuous** (`Continuous.java`) - Continuous process modeling with numerical integration
- **Variable** (`Variable.java`) - State variables that change continuously
- **Monitor** (`Monitor.java`) - Simulation controller managing continuous state updates
- **Random** (`Random.java`) - Random number generation with various distributions
- **Queue Management**: Head, Link, Linkage - Circular two-way list structures
- **Integration Methods**: 13 classes including Euler, RKF45, Adams, etc.
- **Reporting & Statistics**: Reporter, Condition, Accumulate, Tally, Histogram

### Package Structure

All classes are in the `jDisco` package located at `src/main/java/jDisco/`.

## Testing

- **Framework**: JUnit 4.13.2
- **Test location**: `src/test/java/jDisco/`
- **Run tests**: `mvn test` or `docker compose run jdisco mvn test`

Test coverage includes:
- Process lifecycle and event scheduling
- Continuous integration
- Random number distributions
- Queue operations
- Variable state management

## Maven Coordinates

```xml
<dependency>
    <groupId>dk.ruc.keld</groupId>
    <artifactId>jdisco</artifactId>
    <version>1.2.0</version>
</dependency>
```

## Development Guidance

This library is in **maintenance mode**. Modern alternatives exist for new projects (DSOL, Kalasim, SSJ), but jDisco remains valuable for:
- Projects requiring Java 6 compatibility
- Legacy code depending on jDisco API
- Educational purposes
- Systems that cannot be easily migrated

## License

Public domain. Originally written by Keld Helsgaun and released for any purpose whatsoever without acknowledgment.
