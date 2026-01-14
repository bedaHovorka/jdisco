# jDisco - Discrete-Event and Continuous Simulation Library

A Java library for combined discrete-event and continuous simulation, originally written by Keld Helsgaun (Roskilde University, Denmark) and released into the public domain.

## Overview

jDisco provides a framework for building simulation models that combine:

- **Discrete-event simulation** - Process-oriented modeling with coroutines
- **Continuous simulation** - Numerical integration of differential equations
- **State-event detection** - Automatic detection of condition changes
- **Statistical collection** - Built-in reporting and data collection

## Features

- Process-oriented discrete-event simulation with coroutines
- Multiple numerical integration methods (Euler, RKF45, Adams, etc.)
- State-event detection via `waitUntil` mechanism
- Built-in random number generation with multiple distributions
- Queue/list management for process scheduling
- Statistical collection (histograms, tallies, accumulators)
- Zero external dependencies (pure Java SE)

## Requirements

- Java 6 or higher
- Maven 3.x (for building)

## Quick Start

### Maven Dependency

```xml
<dependency>
    <groupId>dk.ruc.keld</groupId>
    <artifactId>jdisco</artifactId>
    <version>1.2.0</version>
</dependency>
```

### Building from Source

**Native build:**
```bash
mvn clean install
```

**Docker build:**
```bash
docker compose build
docker compose run jdisco mvn install
```

## Example: Simple Discrete-Event Simulation

```java
import jDisco.*;

public class SimpleSimulation {
    public static void main(String[] args) {
        // Create a process
        new Process() {
            protected void actions() {
                System.out.println("Process started at time " + time());
                hold(10.0);  // Hold for 10 time units
                System.out.println("Process resumed at time " + time());
            }
        }.activate();  // Activate the main process
    }
}
```

## Example: Continuous Simulation

```java
import jDisco.*;

public class ContinuousExample {
    static Variable x = new Variable(1.0);  // Initial value

    public static void main(String[] args) {
        // Define continuous dynamics
        new Continuous() {
            protected void derivatives() {
                x.rate = -x.state * 0.1;  // dx/dt = -0.1*x
            }
        }.start();

        // Create process to observe
        new Process() {
            protected void actions() {
                for (int i = 0; i < 10; i++) {
                    System.out.println("Time: " + time() + ", x: " + x.state);
                    hold(1.0);
                }
            }
        }.activate();
    }
}
```

## Core Classes

- **Process** - Abstract class for discrete processes
  - `activate()` - Start process execution
  - `hold(t)` - Suspend for time t
  - `passivate()` - Suspend indefinitely
  - `waitUntil(condition)` - Wait for state condition

- **Continuous** - Abstract class for continuous processes
  - `derivatives()` - Define differential equations
  - `start()` - Activate continuous process
  - `stop()` - Deactivate continuous process

- **Variable** - State variable
  - `state` - Current value
  - `rate` - Derivative (dx/dt)

- **Random** - Random number generation
  - `uniform(a, b)` - Uniform distribution
  - `exponential(mean)` - Exponential distribution
  - `normal(mean, std)` - Normal distribution

- **Head, Link** - Queue/list management

## GitHub Actions & CI/CD

This repository uses GitHub Actions for automated builds and AI-assisted code review.

### Claude Code Integration

The repository includes AI-assisted development via Claude Code:

#### 🤖 Automatic Code Review
Pull requests automatically receive AI code review on:
- Java source code changes (`src/main/java/**/*.java`)
- Test code changes (`src/test/java/**/*.java`)
- Build configuration changes (`pom.xml`)

#### 💬 Interactive Claude Assistant
Mention `@claude` in issues, pull requests, or comments to get AI assistance with:
- Code suggestions and improvements
- Bug investigation
- Documentation generation
- Refactoring guidance

Example: "@claude please explain how the Process class handles coroutines"

### Setting Up Claude Code (For Repository Maintainers)

**Required:** Configure the `ANTHROPIC_API_KEY` secret to enable Claude Code workflows.

#### Step 1: Obtain an Anthropic API Key

1. Visit [Anthropic Console](https://console.anthropic.com/)
2. Sign in or create an account
3. Navigate to **API Keys** section
4. Click **Create Key** and copy the key (starts with `sk-ant-`)
5. Keep this key secure - it provides access to Claude API

#### Step 2: Configure GitHub Secret

1. Go to your repository on GitHub
2. Navigate to **Settings** → **Secrets and variables** → **Actions**
3. Click **New repository secret**
4. Name: `ANTHROPIC_API_KEY`
5. Value: Paste your Anthropic API key
6. Click **Add secret**

#### Step 3: Verify Setup

After configuring the secret:

1. Create a test pull request or issue
2. Mention `@claude` in a comment
3. Check the **Actions** tab to verify the workflow runs successfully
4. Claude should respond within a few minutes

**Note:** Claude Code requires an active Anthropic API subscription. Check [Anthropic pricing](https://www.anthropic.com/pricing) for current rates.

### CI/CD Workflows

- **Maven CI** (`maven-ci.yml`) - Builds and tests on every push and PR
- **Claude Code** (`claude.yml`) - Responds to `@claude` mentions
- **Claude Code Review** (`claude-code-review.yml`) - Automated PR reviews
- **Maven Release** (`maven-release.yml`) - Release automation

See `.github/workflows/README.md` for detailed workflow documentation.

## Documentation

- **Development Guidelines**: See `CLAUDE.md`
- **GitHub Workflows**: See `.github/workflows/README.md`
- **API Documentation**: Build with `mvn javadoc:javadoc`, output in `target/site/apidocs/`
- **Original Website**: http://webhotel4.ruc.dk/~keld/research/JDISCO/ (archived)

Detailed technical documentation archived in `/archive/` directory.

## History

- **2001**: Initial release by Keld Helsgaun
- **2004**: Last update (v1.2)
- **2025**: Extracted as standalone Maven library from interlockSim project

## License

Public domain. Originally written by Keld Helsgaun and released into the public domain. This may be used for any purposes whatsoever without acknowledgment.

## Author

**Keld Helsgaun**
Roskilde University, Denmark
Email: keld@ruc.dk

## Modern Alternatives

For new projects, consider these actively maintained alternatives:

- **DSOL** (Distributed Simulation Object Library) - Java 17+, multi-formalism
- **Kalasim** - Native Kotlin with coroutines
- **SSJ** (Stochastic Simulation in Java) - For stochastic simulation

jDisco remains valuable for:
- Java 6 compatibility requirements
- Legacy code maintenance
- Educational purposes
- Systems that cannot be easily migrated
