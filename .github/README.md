# GitHub Configuration for jDisco

This directory contains GitHub-specific configuration for the jDisco project, including CI/CD workflows and documentation.

## Contents

### Workflows (./workflows/)

**maven-ci.yml**
- Continuous Integration pipeline
- Triggers: push to main/develop/feature/fix branches, pull requests, weekly schedule
- Runs: build, test, code quality checks, snapshot publishing
- Status: [View Workflow](https://github.com/bedavs/jDisco/actions/workflows/maven-ci.yml)

**maven-release.yml**
- Release management pipeline
- Triggers: git tags (v*), manual workflow dispatch
- Runs: full build, creates GitHub release, publishes to GitHub Packages
- Status: [View Workflow](https://github.com/bedavs/jDisco/actions/workflows/maven-release.yml)

**settings.xml**
- Maven configuration for GitHub Packages authentication
- Used by workflows to publish artifacts
- Provides repository configuration and server credentials

### Documentation

**CI-CD-QUICKSTART.md** ← START HERE
- Quick reference for developers
- How to create releases
- Using jDisco in other projects
- Troubleshooting common issues

**GITHUB-ACTIONS-SETUP.md**
- Comprehensive workflow documentation
- Detailed explanation of each job
- Performance optimization details
- Debugging guide and best practices
- Complete configuration reference

## Quick Commands

### Create a Release
```bash
git tag -a v1.2.0 -m "Release version 1.2.0"
git push origin v1.2.0
# Workflow automatically builds, tests, and publishes
```

### Check Build Status
Visit: [GitHub Actions](https://github.com/bedavs/jDisco/actions)

### View Build Artifacts
1. Go to Actions tab
2. Click the latest workflow run
3. Scroll to Artifacts section
4. Download JAR files

### View Published Releases
Visit: [Releases Page](https://github.com/bedavs/jDisco/releases)

## Key Information

- **Build Environment**: Ubuntu Latest with JDK 11
- **Java Target**: 1.6 (Java 6 compatible bytecode)
- **Build Tool**: Apache Maven
- **CI Trigger**: Push, PR, schedule (weekly)
- **Release Trigger**: Git tags (v*)
- **Artifact Repository**: GitHub Packages Maven Registry

## Build Badge

Add this to your README.md to display build status:

```markdown
[![Maven Build](https://github.com/bedavs/jDisco/actions/workflows/maven-ci.yml/badge.svg)](https://github.com/bedavs/jDisco/actions/workflows/maven-ci.yml)
```

Result:
[![Maven Build](https://github.com/bedavs/jDisco/actions/workflows/maven-ci.yml/badge.svg)](https://github.com/bedavs/jDisco/actions/workflows/maven-ci.yml)

## File Structure

```
.github/
├── workflows/
│   ├── maven-ci.yml              # Continuous integration workflow
│   ├── maven-release.yml         # Release workflow
│   └── settings.xml              # Maven configuration
├── README.md                      # This file
├── CI-CD-QUICKSTART.md           # Quick start guide
└── GITHUB-ACTIONS-SETUP.md       # Detailed documentation
```

## First-Time Setup

1. Ensure `.github/workflows/` directory exists
2. Verify pom.xml has distributionManagement configured
3. Create first git tag: `git tag -a v1.2.0 -m "Initial release"`
4. Push tag: `git push origin v1.2.0`
5. Monitor Actions tab for workflow execution

## Support & Documentation

- **Quick Help**: See `CI-CD-QUICKSTART.md`
- **Detailed Docs**: See `GITHUB-ACTIONS-SETUP.md`
- **GitHub Actions Guide**: https://docs.github.com/en/actions
- **Maven with GitHub Actions**: https://github.com/actions/setup-java

## Related Files

- **Root**: `pom.xml` - Maven project configuration
- **Build**: `Dockerfile` (if containerized builds)
- **Project Docs**: `README.md` in repository root

---

**Last Updated**: 2026-01-06
**Status**: Active
**Maintainer**: Automated via GitHub Actions
