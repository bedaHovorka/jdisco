# GitHub Actions CI/CD Setup for jDisco

This document provides comprehensive guidance on the GitHub Actions workflows configured for the jDisco project.

## Overview

The jDisco repository uses two complementary GitHub Actions workflows:

1. **maven-ci.yml** - Continuous Integration: Builds, tests, and quality checks
2. **maven-release.yml** - Release Management: Creates releases and publishes to GitHub Packages

## Workflow Triggers

### Continuous Integration (maven-ci.yml)

Automatically triggered on:
- Push to `main`, `develop`, `feature/**`, `fix/**` branches
- Pull requests to `main` and `develop` branches
- Manual dispatch via GitHub UI (`workflow_dispatch`)
- Weekly schedule: Every Monday at 00:00 UTC

### Release (maven-release.yml)

Automatically triggered on:
- Push to tags matching `v*` (e.g., `v1.2.0`, `v1.3.0`)
- Manual dispatch with version parameter

## Workflow Details

### CI Pipeline (maven-ci.yml)

The CI pipeline consists of 5 parallel/sequential jobs:

#### 1. Build Job
- **Environment**: Ubuntu latest with JDK 11 (Temurin distribution)
- **Timeout**: 20 minutes
- **Steps**:
  - Checkout repository
  - Setup JDK 11 (last version supporting Java 6 target)
  - Display Java version for verification
  - Compile project
  - Run JUnit 4 tests (auto-detected from `**/*Test.java`)
  - Generate JavaDoc documentation
  - Package JAR files
  - Upload artifacts (30-day retention)
  - Create test report with dorny/test-reporter

**Why JDK 11?**
- Java 17+ no longer supports Java 6 target compilation
- JDK 11 is the last version that can compile to Java 6 bytecode
- jDisco source code: Java 6 compatible
- jDisco bytecode: Java 6 target (1.6)

#### 2. Integration Test Job
- **Depends on**: Build job
- **Environment**: Ubuntu latest with JDK 11
- **Timeout**: 20 minutes
- **Steps**:
  - Checkout repository
  - Setup JDK 11 with Maven cache
  - Run full Maven install (clean install)
  - Verify JAR creation and contents
  - Upload JAR artifacts (90-day retention)

This job ensures the complete build-test-package cycle works end-to-end.

#### 3. Code Quality Job
- **Environment**: Ubuntu latest with JDK 11
- **Timeout**: 15 minutes
- **Steps**:
  - Checkout repository
  - Setup JDK 11
  - Check compilation warnings
  - Verify code formatting (tabs, line endings, encoding)

This job catches formatting issues and compilation warnings that might be missed by tests.

#### 4. Snapshot Publishing Job
- **Depends on**: Build, Integration Test, Code Quality
- **Runs only on**: Push to `main` or `develop` branches
- **Permissions**: `packages: write`
- **Environment**: JDK 11 with GitHub Packages authentication
- **Step**: Deploy artifacts to GitHub Packages Maven repository

This automatically publishes snapshot versions after successful CI.

#### 5. Status Check Job
- **Depends on**: Build, Integration Test, Code Quality (all jobs)
- **Runs**: Always, even if previous jobs fail
- **Purpose**: Provides clear pass/fail status for the entire CI pipeline

## Maven Configuration for GitHub Packages

### pom.xml Distribution Management

```xml
<distributionManagement>
    <repository>
        <id>github</id>
        <name>GitHub Packages</name>
        <url>https://maven.pkg.github.com/bedavs/jDisco</url>
    </repository>
    <snapshotRepository>
        <id>github</id>
        <name>GitHub Packages Snapshots</name>
        <url>https://maven.pkg.github.com/bedavs/jDisco</url>
    </snapshotRepository>
</distributionManagement>
```

### Authentication

GitHub Actions automatically provides authentication via:
- `GITHUB_TOKEN` environment variable (passed to Maven via `-Dgithub.token`)
- `GITHUB_ACTOR` environment variable (passed via `-Dgithub.username`)

The workflow uses Maven's default authentication mechanism. The `settings.xml` in `.github/workflows/` provides optional configuration if needed.

## Release Workflow (maven-release.yml)

The release workflow automates version releases:

### Trigger Methods

**Method 1: Git Tag (Recommended)**
```bash
git tag -a v1.2.0 -m "Release version 1.2.0"
git push origin v1.2.0
# Workflow automatically triggers and creates release
```

**Method 2: Manual Dispatch**
```
GitHub UI → Actions → Release to GitHub Packages → Run workflow → Enter version
```

### Release Process

1. **Checkout & Build**
   - Checkout tagged commit
   - Setup JDK 11
   - Run full Maven build with tests

2. **Version Extraction**
   - If triggered by tag: extract version from tag name (strips 'v' prefix)
   - If manual: use provided version parameter

3. **Release Notes Generation**
   - Create markdown release notes with:
     - Release version
     - Build information (date, Java version)
     - Artifact list
     - Maven dependency snippet

4. **GitHub Release Creation**
   - Create release on GitHub with generated notes
   - Attach JAR, sources, and documentation artifacts
   - Mark as stable release (not draft/prerelease)

5. **Maven Artifact Publishing**
   - Publish artifacts to GitHub Packages
   - Both release and snapshot repositories configured

6. **Build Summary**
   - Generate GitHub Actions job summary with release details

## Artifact Management

### Retention Policies

| Artifact Type | Retention | Location |
|---|---|---|
| Build artifacts (all files) | 30 days | Actions tab > Build > Artifacts |
| JAR files | 90 days | Actions tab > Integration Test > Artifacts |
| GitHub Packages | Permanent | GitHub Packages > bedavs/jDisco |

### Artifact Contents

**Build Job Uploads** (target/ directory):
- Compiled classes
- Test reports (XML and HTML)
- JAR files (main and sources)
- JavaDoc JAR
- POM file

**Integration Test Job Uploads**:
- `jdisco-1.2.0.jar` - Compiled library
- `jdisco-1.2.0-sources.jar` - Source code
- `jdisco-1.2.0-javadoc.jar` - API documentation

### Accessing Artifacts

1. **From GitHub UI**:
   - Navigate to Actions tab
   - Select workflow run
   - Scroll to Artifacts section
   - Download desired artifact

2. **From GitHub Packages**:
   - Navigate to Code > Packages (right sidebar)
   - Select jdisco package
   - View all versions and releases

## GitHub Packages Configuration

### For Repository Consumers

To use jDisco from GitHub Packages, add the repository to your `pom.xml`:

```xml
<repositories>
    <repository>
        <id>github-jdisco</id>
        <url>https://maven.pkg.github.com/bedavs/jDisco</url>
        <snapshots>
            <enabled>true</enabled>
        </snapshots>
    </repository>
</repositories>
```

Then add the dependency:
```xml
<dependency>
    <groupId>dk.ruc.keld</groupId>
    <artifactId>jdisco</artifactId>
    <version>1.2.0</version>
</dependency>
```

### Authentication for Consumers

If consuming from GitHub Packages (not public), provide credentials via:

**Option 1: GitHub Personal Token (Recommended)**
```bash
export GITHUB_USERNAME=your-username
export GITHUB_TOKEN=ghp_xxxxxxxxxxxx
mvn clean install
```

**Option 2: settings.xml Configuration**
```xml
<servers>
    <server>
        <id>github-jdisco</id>
        <username>your-username</username>
        <password>ghp_xxxxxxxxxxxx</password>
    </server>
</servers>
```

**Scope Required**: `read:packages`

## Performance Optimization

### Maven Dependency Caching

Both workflows use `actions/setup-java@v4` with `cache: 'maven'` to cache:
- Maven plugins
- Project dependencies (downloaded via Maven)

**Cache Hit Rate**: ~90% on subsequent runs within same week
**Impact**: Reduces build time from ~3 minutes (cold cache) to ~1 minute (warm cache)

**Cache Invalidation**:
- Automatic when `pom.xml` changes
- Manual via GitHub Actions Cache UI if needed

### Parallel Job Execution

- Build, Integration Test, Code Quality run in parallel
- Snapshot Publishing and Status Check are sequential
- Total pipeline time: ~20 minutes (limited by longest job)

## Concurrency Control

Both workflows include concurrency configuration:

```yaml
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true
```

This ensures:
- Only one workflow run per branch at a time
- Outdated workflow runs are automatically cancelled
- Reduces resource usage and cost
- Prevents race conditions during deployment

## Build Badges

Add build status badge to README.md:

```markdown
[![Maven Build](https://github.com/bedavs/jDisco/actions/workflows/maven-ci.yml/badge.svg)](https://github.com/bedavs/jDisco/actions/workflows/maven-ci.yml)

[![Release](https://github.com/bedavs/jDisco/actions/workflows/maven-release.yml/badge.svg)](https://github.com/bedavs/jDisco/actions/workflows/maven-release.yml)
```

## Debugging Failed Workflows

### Common Issues and Solutions

#### 1. Compilation Error with Java 6 Target
```
error: Source option 1.6 is no longer supported
```
**Solution**: This occurs with Java 17+. The workflow uses JDK 11 to avoid this.
- Verify workflow uses `java-version: '11'`
- Check `maven-compiler-plugin` has `<source>1.6</source>` and `<target>1.6</target>`

#### 2. Maven Cache Not Working
```
Could not find artifact...
```
**Solution**: Cache invalidation due to pom.xml changes
- Push again - cache rebuilds automatically
- Or manually clear cache via GitHub Actions settings

#### 3. GitHub Packages Authentication Failure
```
401 Unauthorized when deploying
```
**Solution**: Token permissions issue
- Verify `GITHUB_TOKEN` is available in job context
- Check GitHub Actions job has `packages: write` permission
- Check repository allows GitHub Packages access

#### 4. JAR File Not Created
```
JAR not found in verify step
```
**Solution**: Package step failed silently
- Check test output for failures
- Review Maven compiler output
- Verify pom.xml has correct JAR plugin configuration

### Enable Debug Logging

To enable verbose Maven output:

```yaml
- name: Build with debug logging
  run: mvn clean compile -X  # -X enables debug mode
```

Or enable GitHub Actions debug logging:
```bash
ACTIONS_STEP_DEBUG=true
ACTIONS_RUNNER_DEBUG=true
```

## Repository Secrets

The workflows use only GitHub-provided tokens:

- **GITHUB_TOKEN** - Automatically provided, scoped to repository
  - Used for: Maven authentication, package publishing, release creation
  - Permissions: `contents: read`, `packages: write`

No manual secrets need to be configured. The `GITHUB_TOKEN` is automatically available in workflow contexts.

## First-Time Setup Checklist

1. **Enable GitHub Packages**:
   - Verify repository has Packages enabled (usually default)
   - Visit repository → Settings → Check "Packages" is visible

2. **Configure Workflow Permissions** (if repository requires):
   - Settings → Actions → General → Workflow Permissions
   - Select "Read and write permissions"
   - Check "Allow GitHub Actions to create and approve pull requests"

3. **Test Workflows**:
   - Push a commit to `main` or `develop`
   - Monitor Actions tab for workflow runs
   - Verify all jobs pass

4. **Create First Release**:
   ```bash
   git tag -a v1.2.0 -m "Initial release"
   git push origin v1.2.0
   ```
   - Workflow triggers automatically
   - Verify release appears in Releases tab
   - Check GitHub Packages has artifacts

## References

- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [Maven with GitHub Actions](https://github.com/actions/setup-java)
- [GitHub Packages Maven Documentation](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-apache-maven-registry)
- [JDK 11 Java Compiler Documentation](https://docs.oracle.com/en/java/javase/11/tools/javac.html)

## Support

For issues with the workflows:

1. Check the Actions tab for detailed logs
2. Review this document for the specific job/step
3. Check the jDisco GitHub Issues for similar problems
4. Consult GitHub Actions documentation linked above
