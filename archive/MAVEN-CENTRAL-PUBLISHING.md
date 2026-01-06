# Publishing to Maven Central (Future Enhancement)

This document provides guidance for publishing jDisco to the Maven Central Repository in the future. This is currently not configured but can be enabled when ready.

## Current State

**Status**: Publishing to **GitHub Packages only** (configured and active)

**Why GitHub Packages first?**
- Immediate availability after merge to main/develop
- Automatic snapshot publishing
- No external service accounts required
- Suitable for active development

## Why Publish to Maven Central?

**Advantages**:
- Accessible worldwide without authentication
- Standard repository for open-source Java projects
- Better discoverability
- CDN-backed for faster downloads
- Long-term archival support

**Requirements**:
- Published project must meet OSS criteria
- Requires JIRA account with Sonatype
- PGP key setup for artifact signing
- Verification process (first-time only)

## Prerequisites for Maven Central

### 1. Sonatype Account

**Step 1**: Create account at https://oss.sonatype.org/
- Requires valid email
- Password complexity requirements

**Step 2**: Create JIRA ticket
- Navigate to https://issues.sonatype.org/
- Create issue:
  - Project: OSSRH (OSS Repository Hosting)
  - Type: New Project
  - Group Id: `dk.ruc.keld`
  - Project URL: `https://github.com/bedavs/jDisco`
  - SCM URL: `https://github.com/bedavs/jDisco.git`
  - License: Public Domain

**Step 3**: Verify ownership
- Sonatype requires proof of repository ownership
- Can be done by:
  - Creating DNS TXT record
  - Creating GitHub repo in your account
  - Adding comment on JIRA ticket from verified GitHub account

**Expected time**: 24-48 hours for approval

### 2. PGP Key Setup

**Create GPG key** (if not exists):
```bash
gpg --gen-key
# Select: (1) RSA and RSA
# Keysize: 4096
# Validity: 0 (no expiration) or 2y
# Real name: Your Name
# Email: your-email@example.com
# Passphrase: Strong password

# List keys
gpg --list-keys
gpg --list-secret-keys
```

**Export public key to keyserver**:
```bash
gpg --keyserver keys.openpgp.org --send-keys YOUR_KEY_ID
# Wait 5-10 minutes for propagation
```

**Configure Maven** (in `~/.m2/settings.xml`):
```xml
<profiles>
    <profile>
        <id>ossrh</id>
        <properties>
            <gpg.executable>gpg</gpg.executable>
            <gpg.passphrase>YOUR_PASSPHRASE</gpg.passphrase>
        </properties>
    </profile>
</profiles>
<activeProfiles>
    <activeProfile>ossrh</activeProfile>
</activeProfiles>
```

**Better approach** (use Maven GPG plugin environment variable):
```bash
export GPG_PASSPHRASE=your_passphrase
```

### 3. Update pom.xml

Add Maven Central distribution management:

```xml
<distributionManagement>
    <repository>
        <id>ossrh</id>
        <name>Sonatype OSSRH</name>
        <url>https://oss.sonatype.org/service/local/staging/deploy/maven2/</url>
    </repository>
    <snapshotRepository>
        <id>ossrh</id>
        <url>https://oss.sonatype.org/content/repositories/snapshots</url>
    </snapshotRepository>
</distributionManagement>
```

### 4. Update Build Configuration

Add plugins to pom.xml for Central requirements:

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-gpg-plugin</artifactId>
    <version>3.0.1</version>
    <executions>
        <execution>
            <id>sign-artifacts</id>
            <phase>verify</phase>
            <goals>
                <goal>sign</goal>
            </goals>
        </execution>
    </executions>
</plugin>

<plugin>
    <groupId>org.sonatype.plugins</groupId>
    <artifactId>nexus-staging-maven-plugin</artifactId>
    <version>1.6.13</version>
    <extensions>true</extensions>
    <configuration>
        <serverId>ossrh</serverId>
        <nexusUrl>https://oss.sonatype.org/</nexusUrl>
        <autoReleaseAfterClose>true</autoReleaseAfterClose>
    </configuration>
</plugin>
```

## Workflow Changes

### Update Maven CI Workflow

Add Maven Central publishing step (optional, only for releases):

```yaml
  publish-maven-central:
    name: Publish to Maven Central
    runs-on: ubuntu-latest
    needs: [build, integration-test]
    if: startsWith(github.ref, 'refs/tags/v')
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-java@v4
        with:
          distribution: 'temurin'
          java-version: '11'
          cache: 'maven'

      - name: Import GPG key
        env:
          GPG_SECRET_KEYS: ${{ secrets.GPG_SECRET_KEYS }}
          GPG_OWNERTRUST: ${{ secrets.GPG_OWNERTRUST }}
        run: |
          echo $GPG_SECRET_KEYS | base64 --decode | gpg --import --no-tty
          echo $GPG_OWNERTRUST | base64 --decode | gpg --import-ownertrust

      - name: Publish to Maven Central
        run: |
          mvn deploy \
            -P ossrh \
            -Dgpg.passphrase="${{ secrets.GPG_PASSPHRASE }}" \
            -DskipTests
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

### Configure GitHub Secrets

In repository Settings → Secrets and variables → Actions, add:

1. **GPG_SECRET_KEYS**: Base64-encoded GPG private key
   ```bash
   gpg --export-secret-keys YOUR_KEY_ID | base64 -w 0
   # Copy output to secret
   ```

2. **GPG_OWNERTRUST**: Base64-encoded trust settings
   ```bash
   gpg --export-ownertrust | base64 -w 0
   # Copy output to secret
   ```

3. **GPG_PASSPHRASE**: Your GPG passphrase
   - Keep this confidential
   - Use GitHub's secret masking

4. **SONATYPE_USERNAME**: Sonatype account username
   ```bash
   echo -n "your-sonatype-username" | base64 -w 0
   ```

5. **SONATYPE_PASSWORD**: Sonatype account password
   ```bash
   echo -n "your-sonatype-password" | base64 -w 0
   ```

## Publishing Process

### Initial Setup (One-time)

1. Create Sonatype account and JIRA ticket
2. Verify ownership (3-5 days for approval)
3. Create and export GPG key
4. Add secrets to GitHub Actions
5. Update pom.xml and workflows
6. Test with snapshot release

### On Each Release

```bash
# With automatic release (recommended):
git tag -a v1.2.1 -m "Release 1.2.1"
git push origin v1.2.1
# Workflow automatically:
# 1. Builds and tests
# 2. Signs artifacts with GPG
# 3. Deploys to Maven Central staging
# 4. Auto-releases to Central (if configured)

# Expected time to sync to Maven Central: 10-30 minutes
```

### Manual Release (If needed)

```bash
# Close staging repository
mvn nexus-staging:close -DstagingRepositoryId=YOUR_REPO_ID

# Release to Central
mvn nexus-staging:release -DstagingRepositoryId=YOUR_REPO_ID
```

## Verification

### Check Maven Central

After release syncs (10-30 min):

```bash
# Search Maven Central
# URL: https://central.sonatype.com/
# Search for: dk.ruc.keld:jdisco

# Or via Maven:
mvn dependency:get \
  -Dartifact=dk.ruc.keld:jdisco:1.2.0
```

### Test Consuming from Central

Remove GitHub Packages repository from pom.xml, use Maven Central only:

```xml
<!-- Remove GitHub repo, use default Central -->
<dependency>
    <groupId>dk.ruc.keld</groupId>
    <artifactId>jdisco</artifactId>
    <version>1.2.0</version>
</dependency>
```

Build should work without authentication:
```bash
mvn clean install
```

## Troubleshooting

### GPG Key Not Found
```
[ERROR] gpg: can't connect to the agent
```
**Solution**:
```bash
export GPG_TTY=$(tty)
gpg-agent --daemon
```

### Signature Verification Failed
```
[ERROR] Failed to execute goal org.apache.maven.plugins:maven-gpg-plugin:3.0.1:sign
```
**Solution**:
- Verify correct passphrase
- Check GPG key is valid: `gpg --list-secret-keys`
- Regenerate key if expired

### Repository Not Syncing to Central
- Check staging repository status in Sonatype OSSRH
- Verify no validation errors
- Manually release if auto-release disabled
- Wait longer (can take 2-4 hours in rare cases)

## Security Considerations

### GPG Passphrase in CI/CD

**Important**: Never hardcode passphrase in workflow files!

**Best practices**:
- Store passphrase as GitHub secret
- Use environment variables for injection
- Rotate passphrase annually
- Disable GPG key if compromised
- Review secret access logs monthly

### Key Backup

**Keep safe backups of**:
- GPG private key
- GPG passphrase
- Sonatype credentials
- Export key:
  ```bash
  gpg --export-secret-key YOUR_KEY_ID > private.key
  gpg --export-ownertrust > ownertrust.txt
  ```

## Migration from GitHub Packages

Once Maven Central is active:

1. **Gradual transition**:
   - Publish snapshots to GitHub Packages
   - Publish releases to both GitHub Packages and Central
   - Eventually deprecate GitHub Packages

2. **Update consumers**:
   - Switch pom.xml to use Maven Central
   - Remove GitHub Packages repository configuration
   - No auth token needed anymore

3. **Keep GitHub Packages**:
   - Can maintain for backward compatibility
   - Or deprecate after transition period

## References

- [Sonatype OSS Repository Hosting](https://central.sonatype.org/publish/publish-maven/)
- [Maven Signing Guide](https://maven.apache.org/plugins/maven-gpg-plugin/usage.html)
- [Maven Central Requirements](https://central.sonatype.org/publish/requirements/)
- [GitHub GPG/Secrets Best Practices](https://docs.github.com/en/actions/security-guides/encrypted-secrets)

## Timeline for Implementation

**Phase 1** (Current): GitHub Packages + Local development
**Phase 2** (When ready): Add Maven Central publishing workflow
**Phase 3** (Maintenance): Monitor Central syncing, handle issues

**Estimated effort**:
- Setup: 2-4 hours
- Initial approval: 1-3 days
- Integration: 1-2 hours
- Ongoing: Minimal (automated via CI/CD)

## Decision Points

This implementation is recommended when:
- Project is stable and production-ready
- Regular releases are planned
- Community adoption is growing
- Wide availability is important

**Current recommendation**: Continue with GitHub Packages for now. Transition to Maven Central when project reaches broader adoption or you need public access without authentication.

---

**Last Updated**: 2026-01-06
**Status**: Planned Enhancement
**Complexity**: Medium (4-6 hour setup including Sonatype approval)
