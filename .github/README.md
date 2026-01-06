# GitHub Configuration for jDisco

GitHub Actions workflows for CI/CD automation.

## Workflows

**maven-ci.yml** - Continuous Integration
- Triggers: push, PR, weekly schedule
- Jobs: build, test, quality, snapshot publishing
- Status: [View Workflow](https://github.com/bedavs/jDisco/actions/workflows/maven-ci.yml)

**maven-release.yml** - Release Management
- Triggers: version tags (v*)
- Creates GitHub Release with artifacts
- Publishes to GitHub Packages

**settings.xml** - Maven configuration for authentication

## Quick Commands

**Create a Release:**
```bash
git tag -a v1.2.0 -m "Release version 1.2.0"
git push origin v1.2.0
```

**Build Badge:**
```markdown
[![Maven Build](https://github.com/bedavs/jDisco/actions/workflows/maven-ci.yml/badge.svg)](https://github.com/bedavs/jDisco/actions/workflows/maven-ci.yml)
```

## Key Information

- Build: Ubuntu + JDK 11 (Java 6 bytecode target)
- Triggers: Push, PR, weekly schedule
- Artifacts: GitHub Packages Maven Registry

## Documentation

See `/GITHUB-ACTIONS-GUIDE.md` in repository root for complete setup and usage guide.

Detailed documentation archived in `/archive/` directory.
