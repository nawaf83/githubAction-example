# GitHub Actions Example — Java Hello World

A minimal Java project that demonstrates a complete **GitHub Actions CI/CD pipeline** with build, test, code-coverage, and deployment stages.

---

## Table of Contents

- [Overview](#overview)
- [Project Structure](#project-structure)
- [Key Technologies](#key-technologies)
- [Source Code](#source-code)
- [Build & Test Locally](#build--test-locally)
- [CI/CD Pipeline](#cicd-pipeline)

---

## Overview

This project is a "Hello World" Java application used as a learning example for GitHub Actions. It shows how to:

- Build a Java project with Gradle
- Run unit tests and generate a code-coverage report
- Package the application as a runnable JAR
- Upload build artifacts and test reports
- Deploy (run) the application in a separate pipeline job

---

## Project Structure

```
.
├── .github/
│   └── workflows/
│       └── ci.yml              # GitHub Actions CI/CD workflow
├── src/
│   ├── main/
│   │   ├── java/com/example/
│   │   │   └── HelloWorld.java # Main application class
│   │   └── resources/
│   │       └── sample.txt      # Sample resource file
│   └── test/
│       └── java/com/example/
│           └── HelloWorldTest.java  # JUnit unit tests
└── build.gradle                # Gradle build configuration
```

---

## Key Technologies

| Technology | Version | Purpose |
|---|---|---|
| Java | 8 (source/target), built with JDK 17 | Application language |
| Gradle | — | Build tool |
| JUnit | 4.13.2 | Unit testing |
| JaCoCo | 0.8.7 | Code coverage reporting |
| GitHub Actions | — | CI/CD automation |

---

## Source Code

### `HelloWorld.java`

The main application class in the `com.example` package.

- **`main(String[] args)`** — Prints `"Hello, World!"` to standard output. This is the entry point when running the JAR.
- **`getGreeting()`** — Returns the string `"Hello, class!"`. This method is used by the unit tests.

### `HelloWorldTest.java`

A JUnit 4 test class that verifies the behaviour of `HelloWorld`.

- **`testGetGreeting()`** — Asserts that `getGreeting()` returns the expected greeting string.

### `build.gradle`

Configures the Gradle build:

- Applies the `java` and `jacoco` plugins.
- Sets Java 8 source/target compatibility.
- Declares a JUnit 4 test dependency from Maven Central.
- Configures the `jar` task to produce `build/libs/hello-world-java.jar` with `HelloWorld` as the main class.
- Configures the `jacocoTestReport` task to generate both XML and HTML coverage reports after every test run.

---

## Build & Test Locally

**Prerequisites:** JDK 17+ and Gradle installed (or use the Gradle wrapper if present).

```bash
# Compile and package (skips tests)
gradle clean assemble

# Run unit tests and generate coverage report
gradle test

# View the HTML coverage report
open build/reports/jacoco/test/html/index.html

# Run the application
java -jar build/libs/hello-world-java.jar
```

---

## CI/CD Pipeline

The workflow defined in `.github/workflows/ci.yml` triggers on every **push** or **pull request** to the `master` branch. It consists of three jobs:

```
push / pull_request
        │
        ▼
   ┌─────────┐
   │  build  │  ── compiles the project, uploads JAR artifact
   └────┬────┘
        │
   ┌────┴────────────────────┐
   ▼                         ▼
┌──────┐              ┌────────────┐
│ test │              │ deployment │
└──────┘              └────────────┘
```

### `build` job

1. Checks out the code.
2. Sets up JDK 17 (Adoptium distribution).
3. Runs `gradle clean assemble` to compile and package without running tests.
4. Uploads the built JAR as the `build-artifacts` artifact.

### `test` job *(depends on `build`)*

1. Checks out the code.
2. Sets up JDK 17.
3. Runs `gradle test` to execute unit tests.
4. Runs `gradle jacocoTestReport` to generate the coverage report.
5. Uploads test reports as the `test-reports` artifact.
6. Uploads the JaCoCo HTML coverage report as the `code-coverage-report` artifact.

### `deployment` job *(depends on `build`)*

1. Downloads the `build-artifacts` artifact produced by the `build` job.
2. Executes the JAR with `java -jar build/libs/*.jar` to simulate a deployment/run step.
