# Repository Setup Instructions for jDisco GitHub Repository

This guide provides step-by-step instructions to set up a new GitHub repository for jDisco with all CI/CD workflows properly configured.

## Prerequisites

- GitHub account with repository owner access
- Git installed locally
- jDisco project files ready to push

## Step 1: Create GitHub Repository

### Via GitHub Web UI

1. Go to [GitHub New Repository](https://github.com/new)
2. **Repository name**: `jDisco`
3. **Description**: "Discrete-event and continuous simulation library for Java"
4. **Visibility**: Choose Public or Private based on needs
5. **Initialize repository**: Leave unchecked (will push existing code)
6. Click **Create repository**

### Via GitHub CLI

```bash
gh repo create jDisco \
  --description "Discrete-event and continuous simulation library for Java" \
  --public \
  --source=. \
  --remote=origin \
  --push
```

## Step 2: Configure Repository Settings

### Enable Required Features

1. Navigate to repository Settings
2. **Code security and analysis** section:
   - Enable "Dependabot alerts"
   - Enable "Dependency graph" (if available)
3. **Actions** settings:
   - Verify "Actions permissions" is set to "Allow all actions"
   - Check "Workflow permissions":
     - Select "Read and write permissions"
     - Check "Allow GitHub Actions to create and approve pull requests"

### Configure Branch Protection (Optional but Recommended)

1. Go to Settings → Branches
2. Click **Add rule** for branch protection
3. **Branch name pattern**: `main`
4. Configure:
   - Require pull request reviews: 1 review
   - Require status checks to pass: Check "maven-ci" jobs
   - Require up-to-date branches: Yes
   - Require conversation resolution: Yes
5. Click **Create**

## Step 3: Push Project Code

### From Existing Local Repository

```bash
cd /home/beda/work/interlockSim/jdisco

# Add remote (if not already configured)
git remote add origin https://github.com/your-username/jDisco.git

# Verify remote
git remote -v

# Push all branches
git push -u origin main develop

# Push tags (if any)
git push origin --tags
```

### Initialize New Repository

```bash
cd /path/to/jdisco

git init
git add .
git commit -m "Initial commit: jDisco discrete-event simulation library"
git branch -M main
git remote add origin https://github.com/your-username/jDisco.git
git push -u origin main
```

## Step 4: Verify Workflows Are Enabled

1. Navigate to repository **Actions** tab
2. You should see two workflows listed:
   - **Maven Build with Java 6 Compatibility** (maven-ci.yml)
   - **Release to GitHub Packages** (maven-release.yml)
3. If workflows not visible:
   - Verify `.github/workflows/*.yml` files are in repository
   - Workflows are automatically discovered by GitHub
   - Refresh page if still not visible

## Step 5: Test CI Workflow

### Create Test Commit

```bash
# Make a small change (e.g., add comment to README)
echo "# Testing GitHub Actions" >> README.md
git add README.md
git commit -m "test: verify CI workflow executes"
git push origin main
```

### Monitor Workflow

1. Go to **Actions** tab
2. Click the "Maven Build with Java 6 Compatibility" workflow
3. View the latest run
4. Watch jobs execute:
   - Build (compile, test, package)
   - Integration Test
   - Code Quality
   - Snapshot Publishing
   - Status Check
5. All jobs should pass (green checkmarks)

**Expected duration**: 15-25 minutes

## Step 6: Configure GitHub Packages (Maven Registry)

### Repository Permissions

1. Go to Settings → Actions → General
2. Under "Workflow permissions":
   - Ensure "Read and write permissions" is selected
   - This allows workflows to publish to GitHub Packages

### Test Publishing

The CI workflow automatically publishes snapshots on push to `main`/`develop`. To verify:

1. Monitor workflow completion in Actions tab
2. Once "Publish Snapshot" job succeeds, check:
   - Repository sidebar → Packages section
   - Should show `jdisco` package
   - Version shown as `1.2.0-SNAPSHOT`

### Access GitHub Packages

```bash
# List versions
gh api repos/your-username/jDisco/packages/maven/jdisco/versions

# Or view in web UI:
# GitHub.com → Repository → Packages → jdisco
```

## Step 7: Create First Release

### Create Release Tag

```bash
git tag -a v1.2.0 -m "Release version 1.2.0"
git push origin v1.2.0
```

### Monitor Release Workflow

1. Go to **Actions** tab
2. Click **Release to GitHub Packages** workflow
3. Watch jobs execute:
   - Build and test complete project
   - Create GitHub release with artifacts
   - Publish to GitHub Packages
4. Duration: ~20 minutes

### Verify Release

1. Check **Releases** tab:
   - Should show v1.2.0 release
   - Artifacts attached (JAR, sources, javadoc)
   - Release notes generated
2. Check **Packages**:
   - Version `1.2.0` now available
   - Artifact files visible

## Step 8: Configure Consuming Projects

### interlockSim Integration

Update interlockSim's pom.xml to use published jDisco:

```xml
<repositories>
    <repository>
        <id>github-jdisco</id>
        <url>https://maven.pkg.github.com/bedavs/jDisco</url>
    </repository>
</repositories>

<dependency>
    <groupId>dk.ruc.keld</groupId>
    <artifactId>jdisco</artifactId>
    <version>1.2.0</version>
</dependency>
```

### Update interlockSim workflows

If interlockSim's Gradle build consumes jDisco from Maven Central or GitHub Packages:

```gradle
repositories {
    mavenCentral()
    maven {
        url = "https://maven.pkg.github.com/bedavs/jDisco"
        credentials {
            username = project.findProperty("github.username")
            password = project.findProperty("github.token")
        }
    }
}
```

## Step 9: Add Repository Metadata

### Update README.md

Add build badge and badges:
```markdown
# jDisco

Discrete-event and continuous simulation library for Java.

[![Maven Build](https://github.com/your-username/jDisco/actions/workflows/maven-ci.yml/badge.svg)](https://github.com/your-username/jDisco/actions/workflows/maven-ci.yml)

## Usage

```xml
<dependency>
    <groupId>dk.ruc.keld</groupId>
    <artifactId>jdisco</artifactId>
    <version>1.2.0</version>
</dependency>
```

## Documentation

- [CI/CD Quick Start](/.github/CI-CD-QUICKSTART.md)
- [Detailed Workflow Setup](/.github/GITHUB-ACTIONS-SETUP.md)
```

### Add GitHub Topics

1. Go to Settings (bottom right of About section)
2. Add topics:
   - `simulation`
   - `discrete-event`
   - `java`
   - `continuous`
   - `maven`
   - `java6`

## Step 10: Set Up Personal Access Token (If Private)

### For Consumers of the Library

If the repository is private, consumers need PAT to access GitHub Packages:

1. Personal Settings → Developer settings → [Personal access tokens](https://github.com/settings/tokens)
2. Click **Generate new token (classic)**
3. Name: `GITHUB_TOKEN_PACKAGES`
4. Scopes:
   - `read:packages` - to download packages
   - `write:packages` - if also publishing
5. Copy token
6. Store in password manager or GitHub Codespaces secrets

### Configure in Consumer Project

**Option A: Environment Variable**
```bash
export GITHUB_TOKEN=ghp_xxxxxxxxxxxx
mvn clean install
```

**Option B: Maven settings.xml**
```xml
<servers>
    <server>
        <id>github-jdisco</id>
        <username>your-username</username>
        <password>ghp_xxxxxxxxxxxx</password>
    </server>
</servers>
```

**Option C: GitHub Codespaces Secrets**
```bash
gh secret set GITHUB_TOKEN_PACKAGES --body "ghp_xxxxxxxxxxxx"
```

## Troubleshooting Setup

### Workflows Not Running

**Issue**: Workflows don't trigger after push
- **Solution 1**: Verify `.github/workflows/` directory exists and files are committed
- **Solution 2**: Check Actions tab settings - verify workflows are enabled
- **Solution 3**: Try manual workflow dispatch: Actions → Select workflow → Run workflow

### Authentication Errors During Publishing

**Issue**: `401 Unauthorized` during Maven deploy
- **Solution**: Verify GitHub Actions has `packages: write` permission (Settings → Actions → Permissions)
- Check repository access is public or accessible to your account

### Branch Protection Blocking Pushes

**Issue**: Cannot push to main - branch protection active
- **Solution 1**: Push to feature branch, create PR, have it reviewed
- **Solution 2**: Temporarily disable branch protection to initial setup (not recommended for production)

### Maven Cache Issues

**Issue**: "Could not find artifact" despite correct pom.xml
- **Solution**: Clear Maven cache: `rm -rf ~/.m2/repository`
- Run: `mvn clean install`

## Ongoing Maintenance

### Regular Tasks

**Weekly**:
- Monitor Actions for failed builds
- Review test results

**Monthly**:
- Check for dependency updates
- Review GitHub Packages stats

**On Each Release**:
1. Create git tag: `git tag -a vX.Y.Z -m "Release X.Y.Z"`
2. Push tag: `git push origin vX.Y.Z`
3. Monitor Release workflow
4. Verify artifacts in Releases and Packages

### Updating Workflows

Workflows can be updated by:
1. Edit `.github/workflows/*.yml`
2. Test in feature branch with `workflow_dispatch`
3. Merge to main

## Reference Documentation

- [GitHub Actions Setup Guide](./GITHUB-ACTIONS-SETUP.md)
- [CI/CD Quick Start](./CI-CD-QUICKSTART.md)
- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [GitHub Packages Maven Registry](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-apache-maven-registry)

## Support

For issues:
1. Check GitHub Actions logs (Actions tab)
2. Review workflow configuration
3. Consult documentation files above
4. Check GitHub Status: https://www.githubstatus.com/

---

**Setup Date**: 2026-01-06
**Java Requirement**: JDK 11 (for Java 6 target)
**Build Tool**: Apache Maven 3.6+
**Expected Setup Time**: 30-45 minutes
