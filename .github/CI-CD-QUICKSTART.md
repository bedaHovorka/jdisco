# CI/CD Quick Start Guide for jDisco Repository

## Quick Reference

### Build Badge
```markdown
[![Maven Build](https://github.com/bedavs/jDisco/actions/workflows/maven-ci.yml/badge.svg)](https://github.com/bedavs/jDisco/actions/workflows/maven-ci.yml)
```

### Maven Coordinates
```xml
<dependency>
    <groupId>dk.ruc.keld</groupId>
    <artifactId>jdisco</artifactId>
    <version>1.2.0</version>
</dependency>
```

### Build This Project Locally
```bash
# Prerequisites: Java 11+, Maven 3.6+
mvn clean install
```

## Workflows at a Glance

### Continuous Integration (On Every Push/PR)
- **File**: `.github/workflows/maven-ci.yml`
- **Runs**: ~20 minutes
- **Tests**: JUnit 4 test suite
- **Quality**: Compilation checks, code formatting verification
- **Artifacts**: JAR files, test reports
- **Java**: JDK 11 (for Java 6 target compatibility)

### Release Management (On Version Tags)
- **File**: `.github/workflows/maven-release.yml`
- **Trigger**: Push git tag like `v1.2.0`
- **Creates**: GitHub Release with artifacts
- **Publishes**: Artifacts to GitHub Packages

## Creating a Release

### Step 1: Tag the Release
```bash
git tag -a v1.2.0 -m "Release version 1.2.0"
git push origin v1.2.0
```

### Step 2: Watch the Workflow
- GitHub automatically triggers `maven-release.yml`
- Visit Actions tab to monitor progress
- Takes ~20 minutes

### Step 3: Verify Release
- Check [Releases page](https://github.com/bedavs/jDisco/releases)
- Download artifacts from release notes
- Verify artifacts in [GitHub Packages](https://github.com/bedavs/jDisco/packages)

## Using jDisco in Your Project

### Add Repository (pom.xml)
```xml
<repositories>
    <repository>
        <id>github-jdisco</id>
        <url>https://maven.pkg.github.com/bedavs/jDisco</url>
    </repository>
</repositories>
```

### Add Dependency (pom.xml)
```xml
<dependency>
    <groupId>dk.ruc.keld</groupId>
    <artifactId>jdisco</artifactId>
    <version>1.2.0</version>
</dependency>
```

### If Private GitHub Token Required
```bash
export GITHUB_ACTOR=your-username
export GITHUB_TOKEN=ghp_xxxxxxxxxxxx
mvn clean install
```

## Checking Build Status

### GitHub UI
1. Navigate to Actions tab
2. View all workflow runs
3. Click run to see detailed logs
4. Download artifacts from run summary

### Command Line
```bash
# Using GitHub CLI (gh)
gh run list --workflow maven-ci.yml
gh run view <run-id> --log
```

## Troubleshooting

### Build Fails Locally But Passes in CI
- Ensure you have JDK 11+
- Run `mvn clean install` (not just `mvn install`)
- Check Maven version: `mvn -v` (3.6.0+)

### JAR Not Created
- Check test failures in Maven output
- All tests must pass for JAR creation
- Review test output in GitHub Actions

### Can't Consume from GitHub Packages
- Verify you added the repository to pom.xml
- Check your GitHub token has `read:packages` scope
- Verify token is not expired

## File Structure
```
.github/
├── workflows/
│   ├── maven-ci.yml              # Continuous integration
│   ├── maven-release.yml         # Release management
│   └── settings.xml              # Maven configuration
├── GITHUB-ACTIONS-SETUP.md       # Detailed workflow documentation
└── CI-CD-QUICKSTART.md           # This file
```

## Key Facts About This Project

- **Java Source**: 1.6 (Java 6 compatible code)
- **Java Target**: 1.6 (compiles to Java 6 bytecode)
- **Build Tool**: Apache Maven
- **Build Java**: JDK 11 (last version supporting Java 6 target)
- **Tests**: JUnit 4.13.2
- **License**: Public Domain

## Need Help?

1. **Workflow not running?** → Check Actions → Workflows → Verify enabled
2. **Build failing?** → Check detailed logs in Actions tab
3. **Publishing issues?** → See `.github/GITHUB-ACTIONS-SETUP.md`
4. **Java version mismatch?** → Ensure local JDK 11+ for development

---
For comprehensive documentation, see [`.github/GITHUB-ACTIONS-SETUP.md`](./GITHUB-ACTIONS-SETUP.md)
