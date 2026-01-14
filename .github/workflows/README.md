# GitHub Actions Workflows

This directory contains CI/CD workflows for the jDisco project.

## Workflows Overview

### 1. Maven CI (`maven-ci.yml`)

**Purpose:** Continuous integration for Java builds and tests

**Triggers:**
- Push to `main`, `develop`, or feature branches
- Pull requests to `main` or `develop`
- Manual dispatch
- Weekly schedule (Mondays at 00:00 UTC)

**Jobs:**
- **build** - Compile Java code with JDK 11 (Java 6 target compatibility)
- **integration-test** - Run full integration tests
- **code-quality** - Check code formatting and compilation warnings
- **publish-snapshot** - Deploy to GitHub Packages (main/develop only)

**Configuration:**
- Java Version: JDK 11 (compiles to Java 6 bytecode)
- Maven settings: `.m2/settings.xml`
- Test reports: Uploaded as artifacts
- Timeout: 20 minutes per job

### 2. Claude Code (`claude.yml`)

**Purpose:** AI-assisted development via `@claude` mentions

**Triggers:**
- Issue comments containing `@claude`
- Pull request review comments containing `@claude`
- Pull request reviews containing `@claude`
- New issues with `@claude` in title or body

**Features:**
- Interactive AI assistant for code questions
- Can read CI results on PRs
- Responds directly in comments
- Timeout: 20 minutes
- Error handling: Non-blocking failures

**Usage Examples:**
```
@claude please explain how the Process class works
@claude can you suggest improvements for this code?
@claude help debug this test failure
```

**Requirements:**
- `ANTHROPIC_API_KEY` secret must be configured
- See main README.md for setup instructions

**Permissions:**
- `contents: read` - Read repository code
- `pull-requests: read` - Read PR context
- `issues: read` - Read issue context
- `actions: read` - Read CI results
- `id-token: write` - OIDC authentication

### 3. Claude Code Review (`claude-code-review.yml`)

**Purpose:** Automated AI code review on pull requests

**Triggers:**
- Pull request opened, synchronized, reopened, or marked ready for review
- Only on relevant file changes (Java, pom.xml, Dockerfile, workflows)

**Path Filters:**
```yaml
- "src/main/java/**/*.java"
- "src/test/java/**/*.java"
- "pom.xml"
- "CLAUDE.md"
- "Dockerfile"
- ".github/workflows/**"
```

**Review Focus:**
- Potential bugs or logic errors
- Java 6 compatibility (no Java 7+ features)
- Error handling
- Security vulnerabilities
- Code style consistency
- Improvement suggestions

**Configuration:**
- Timeout: 15 minutes
- Fetch depth: 50 commits (for context)
- Error handling: Non-blocking (continues on failure)
- Concurrency: One review per PR at a time

**Permissions:**
- `contents: read` - Read repository contents
- `pull-requests: write` - Post review comments
- `issues: read` - Read issue context
- `id-token: write` - OIDC authentication

**Optional Filters:**
You can uncomment the `if` condition to run reviews only for external contributors:
```yaml
if: |
  github.event.pull_request.author_association == 'FIRST_TIME_CONTRIBUTOR' ||
  github.event.pull_request.author_association == 'CONTRIBUTOR' ||
  github.event.pull_request.author_association == 'NONE'
```

This saves API credits by skipping reviews for trusted maintainers.

### 4. Maven Release (`maven-release.yml`)

**Purpose:** Automated release workflow

**Triggers:** Manual dispatch or release tags

**Features:**
- Builds release artifacts
- Publishes to Maven repositories
- Creates GitHub releases

## Cost Management

### Claude Code API Usage

Claude Code workflows use the Anthropic API, which incurs costs:

**Estimated Costs:**
- Interactive `@claude` requests: ~$0.01-0.10 per request (varies by complexity)
- Automated code reviews: ~$0.05-0.50 per PR (depends on diff size)

**Cost Reduction Strategies:**

1. **Path Filters** (enabled) - Only review relevant files
2. **Author Filters** (optional) - Skip reviews for maintainers
3. **Timeouts** (enabled) - Prevent runaway costs (15-20 min limits)
4. **Error Handling** (enabled) - Graceful failures don't retry indefinitely

**Monitoring:**
- Check GitHub Actions logs for API call counts
- Monitor your Anthropic Console for usage metrics
- Set up billing alerts in Anthropic Console

**Monthly Estimates:**
- Small project (5-10 PRs/month): $1-5
- Medium project (20-50 PRs/month): $10-25
- Active project (100+ PRs/month): $50-100

Actual costs depend on:
- PR size (lines changed)
- Complexity of code
- Number of files modified
- Frequency of `@claude` mentions

## Setup Requirements

### Required Secrets

| Secret | Purpose | Setup Instructions |
|--------|---------|-------------------|
| `ANTHROPIC_API_KEY` | Claude Code authentication | See main README.md |
| `GITHUB_TOKEN` | Maven publishing | Automatically provided by GitHub |

### Optional Configuration

**Maven Settings** (`.m2/settings.xml`)
- Local repository path
- Mirror configurations
- Server credentials

**Environment Variables:**
- `MAVEN_OPTS` - JVM options for Maven

## Troubleshooting

### Claude Code Workflow Failures

**Problem:** "ANTHROPIC_API_KEY not found"
- **Solution:** Configure the secret in repository settings (see README.md)

**Problem:** "Rate limit exceeded"
- **Solution:** Wait a few minutes and try again, or upgrade your Anthropic plan

**Problem:** "Timeout after 15 minutes"
- **Solution:** This is expected for very large PRs. Claude will post partial results.

**Problem:** "Network error"
- **Solution:** Temporary issue. The workflow will automatically fail gracefully.

### Maven CI Failures

**Problem:** "Java version mismatch"
- **Solution:** Workflow uses JDK 11 but targets Java 6 bytecode. This is correct.

**Problem:** "Test failures"
- **Solution:** Check test reports in workflow artifacts

**Problem:** "Maven settings not found"
- **Solution:** Ensure `.m2/settings.xml` exists in repository

## Best Practices

### For Contributors

1. **Run tests locally** before pushing
   ```bash
   mvn clean test
   ```

2. **Use `@claude` wisely** - Be specific in your requests
   - ✅ Good: "@claude explain the Continuous class integration mechanism"
   - ❌ Vague: "@claude help"

3. **Wait for CI** before requesting review
   - Let maven-ci.yml complete first
   - Claude can read CI results to provide better context

### For Maintainers

1. **Monitor API costs** in Anthropic Console

2. **Consider author filters** to reduce costs:
   - Uncomment the `if` condition in `claude-code-review.yml`
   - Reviews will only run for external contributors

3. **Review Claude's suggestions** before accepting
   - Claude is helpful but not infallible
   - Always verify suggested changes

4. **Update path filters** as project structure changes
   - Add new source directories
   - Include relevant configuration files

## Workflow Dependencies

```
maven-ci.yml
  ├── build (always runs)
  ├── integration-test (depends on: build)
  ├── code-quality (independent)
  └── publish-snapshot (depends on: build, only main/develop)

claude.yml (triggered by mentions, independent)

claude-code-review.yml (triggered by PRs, independent)
  └── Runs concurrently with maven-ci.yml
      (does not block PR merge)

maven-release.yml (manual/tags only)
```

## Security Considerations

### Permissions Model

All workflows follow the **principle of least privilege**:
- Read-only access to repository contents by default
- Write access only where necessary (posting comments)
- No workflows can push code or modify repository structure

### Secret Management

- `ANTHROPIC_API_KEY` - Stored as encrypted GitHub secret
- Never logged or exposed in workflow output
- Only accessible to workflow steps that need it

### External Actions

All external actions are pinned to specific versions:
- `actions/checkout@v4`
- `actions/setup-java@v4`
- `actions/upload-artifact@v4`
- `anthropics/claude-code-action@v1`

## Contributing to Workflows

When modifying workflows:

1. **Test changes** on a feature branch first
2. **Document** any new secrets or configuration
3. **Update** this README with changes
4. **Consider costs** of new Claude Code features
5. **Review permissions** - use minimum necessary

## Additional Resources

- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [Maven Build Documentation](../BUILD-FIXES.md)
- [Claude Code Action](https://github.com/anthropics/claude-code-action)
- [Anthropic API Documentation](https://docs.anthropic.com/)
- [Java 6 Compatibility Guide](../CLAUDE.md)

## Support

For workflow issues:
1. Check GitHub Actions logs
2. Review this documentation
3. Check [GitHub Actions status](https://www.githubstatus.com/)
4. Open an issue with workflow logs attached
