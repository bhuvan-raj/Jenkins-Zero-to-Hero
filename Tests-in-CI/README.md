<img src="https://github.com/bhuvan-raj/Github-Actions/blob/main/Tests%20in%20CI/assets/test.png" alt="Banner" />

<div align="center">

![CI/CD](https://img.shields.io/badge/CI%2FCD-Testing-brightgreen?style=for-the-badge)
![DevSecOps](https://img.shields.io/badge/DevSecOps-Security-red?style=for-the-badge)
![Jenkins](https://img.shields.io/badge/Jenkins-D24939?style=for-the-badge&logo=jenkins&logoColor=white)
![SonarQube](https://img.shields.io/badge/SonarQube-4E9BCD?style=for-the-badge&logo=sonarqube&logoColor=white)

# Automated Testing in CI

> **Shift left. Catch bugs early. Ship with confidence.**

</div>

---

## 📋 Table of Contents

- [Why Testing in CI/CD?](#-why-testing-in-cicd)
- [The Test Pyramid](#-the-test-pyramid)
- [Functional Test Types](#-functional-test-types)
  - [Unit Tests](#1-unit-tests)
  - [Integration Tests](#2-integration-tests)
  - [Component Tests](#3-component-tests)
  - [End-to-End Tests](#4-end-to-end-e2e-tests)
  - [Smoke Tests](#5-smoke-tests)
- [Security Tests — DevSecOps](#-security-tests--devsecops)
  - [SAST](#1-sast--static-application-security-testing)
  - [DAST](#2-dast--dynamic-application-security-testing)
  - [SCA](#3-sca--software-composition-analysis)
  - [IAST](#4-iast--interactive-application-security-testing)
  - [Secret Detection](#5-secret-detection)
- [Tools Reference Table](#-tools-reference-table)
- [Key Takeaways](#-key-takeaways)

---

## 🚀 Why Testing in CI/CD?

In modern software delivery, testing is not a final gate — it is an **ongoing, automated process** woven into every stage of the pipeline. The goal is to **shift left**: integrate quality and security checks as early as possible so that defects are caught when they are cheapest to fix.

| Benefit | Why It Matters |
|---------|---------------|
| 🐛 Early Bug Detection | Bugs caught at commit time cost a fraction of bugs caught in production |
| 🛡️ Prevent Regressions | Automated tests ensure new changes don't silently break existing functionality |
| ⚡ Faster Feedback Loops | Developers know within minutes if their code works correctly |
| 🏗️ Code Quality & Reliability | A strong test suite builds confidence for bold refactors and new features |
| 🔐 Security by Design | Integrating security tests early (DevSecOps) eliminates vulnerabilities before they reach users |

---

## 📈 The Test Pyramid

The **Test Pyramid** is a strategy for balancing your automated testing efforts. The rule is simple: have many fast low-level tests and progressively fewer slow high-level tests.

```
              ▲
             /|\
            / | \
           /  |  \
          / E2E \        ← Few | Slow | Broad scope | Expensive per test
         /-------|
        /Component\      ← Moderate | Medium speed | Focused on one service
       /-----------|
      / Integration \    ← More | Medium speed | Inter-component communication
     /---------------|
    /      Unit       \  ← Many | Fast | Narrow scope | Cheap per test
   /-------------------|
```

**The core rule:** Find and fix bugs at the **lowest possible level**. Unit tests are the fastest, cheapest, and most precise. Over-relying on slow E2E tests (the "Ice Cream Cone" anti-pattern) leads to slow, fragile, expensive pipelines.

---

## 🔍 Functional Test Types

### 1. Unit Tests

The **microscope** of testing — verifies a single function or method in complete isolation.

- **Focus:** Internal logic of one unit of code
- **Speed:** Extremely fast (milliseconds per test)
- **Dependencies:** Mocked or simulated — no real DBs, no real APIs
- **Value in CI:** Immediate, precise feedback. Should make up the largest portion of your test suite.

**Example — Java with JUnit + Mockito:**
```java
@Test
void shouldReturnDiscountedPrice() {
    PricingService service = new PricingService();
    double result = service.applyDiscount(100.0, 20);
    assertEquals(80.0, result);
}
```

---

### 2. Integration Tests

The **connecting blocks** test — verifies that different modules or services interact correctly with each other and with external systems like databases or APIs.

- **Focus:** Interfaces and data flow between components
- **Speed:** Slower than unit tests — involves real communication
- **Dependencies:** Real or containerized external systems (e.g., Testcontainers)
- **Value in CI:** Catches interface mismatches and data flow errors that unit tests can't detect.

**Example — Testing a REST API endpoint with REST Assured (Java):**
```java
@Test
void shouldReturn200ForHealthCheck() {
    given()
        .when().get("/api/health")
        .then().statusCode(200);
}
```

---

### 3. Component Tests

The **mini-system** test — tests a significant, independent part of your application (e.g., a microservice) in isolation from other major services, but with its own internal dependencies (e.g., its own database).

- **Focus:** Overall behavior and API contract of a single service
- **Speed:** Moderate — slower than integration, faster than full E2E
- **Dependencies:** The service's own internal dependencies (real or containerized), other services mocked
- **Value in CI:** Critical for microservice architectures. Strong confidence in a single service without the overhead of the full system.

---

### 4. End-to-End (E2E) Tests

The **user's journey** test — simulates a complete user workflow across the entire application stack, from the UI through the backend and all integrated systems.

- **Focus:** Full user experience validation
- **Speed:** Slowest and most complex — involves real browsers, real services, real data flows
- **Dependencies:** The fully deployed application
- **Value in CI:** Highest confidence in the complete system. Essential before every production release.

**Example — Cypress E2E test:**
```javascript
it('user can log in and view dashboard', () => {
    cy.visit('/login');
    cy.get('[data-cy=email]').type('user@example.com');
    cy.get('[data-cy=password]').type('password123');
    cy.get('[data-cy=submit]').click();
    cy.url().should('include', '/dashboard');
});
```

---

### 5. Smoke Tests

The **quick health check** — a small, fast set of tests that verify the most critical functionality is working after a new build or deployment.

- **Focus:** Basic system stability and core feature availability
- **Speed:** Very fast — designed to be a rapid go/no-go gate
- **Dependencies:** The deployed application
- **Value in CI:** Acts as a gatekeeper. If smoke tests fail, the pipeline stops immediately — no point running a full test suite against a broken build.

---

## 🔐 Security Tests — DevSecOps

**DevSecOps** is the practice of integrating security into every stage of the CI/CD pipeline — baking it in from the start rather than scanning at the very end.

```
  Code Commit
      │
      ├──▶ Secret Detection      (pre-commit / PR gate)
      ├──▶ SAST                  (build phase — static analysis)
      ├──▶ SCA                   (build phase — dependency scan)
      ├──▶ IAST                  (during functional/integration tests)
      └──▶ DAST                  (against deployed staging environment)
```

---

### 1. SAST — Static Application Security Testing

Analyzes **source code without executing it** to find security vulnerabilities like SQL injection, XSS, and hardcoded secrets.

- **Type:** White-box testing (requires source code)
- **When in CI:** Build phase — run on every commit or pull request
- **Key Tools:** SonarQube, Semgrep, Checkmarx, Bandit (Python), ESLint security rules (JS)

---

### 2. DAST — Dynamic Application Security Testing

Analyzes a **running application from the outside** by simulating attacks — probing for misconfigurations, injection flaws, and runtime vulnerabilities.

- **Type:** Black-box testing (no source code needed)
- **When in CI:** Against a deployed staging or testing environment
- **Key Tools:** OWASP ZAP, Burp Suite, Invicti, Acunetix

---

### 3. SCA — Software Composition Analysis

Identifies and audits **open-source and third-party libraries** for known vulnerabilities (CVEs) and license compliance issues.

- **Type:** Dependency / supply chain analysis
- **When in CI:** Early in the pipeline during the build phase when dependencies are resolved
- **Key Tools:** Snyk, OWASP Dependency-Check, Dependabot, Black Duck, Sonatype Nexus Lifecycle

---

### 4. IAST — Interactive Application Security Testing

A **hybrid approach** — an agent runs inside the application and observes code execution in real time during functional tests, identifying vulnerabilities with precise runtime context.

- **Type:** Hybrid (combines SAST + DAST)
- **When in CI:** During integration or functional testing phases
- **Advantage:** High accuracy, fewer false positives than SAST or DAST alone
- **Key Tools:** Contrast Security, HCL AppScan, Veracode IAST

---

### 5. Secret Detection

Scans the **codebase and commit history** for accidentally hardcoded secrets — API keys, passwords, tokens, and certificates.

- **Type:** Pattern-matching / entropy analysis on code
- **When in CI:** Pre-commit hooks or as a mandatory early pipeline step
- **Key Tools:** GitGuardian, TruffleHog, Gitleaks, AWS Git Secrets, SpectralOps

---

## 🛠️ Tools Reference Table

| Test Type | Language / Platform | Key Tools |
|-----------|--------------------|-----------| 
| **Unit Tests** | Java | JUnit, TestNG, Mockito |
| | Python | pytest, unittest, mock |
| | JavaScript / TypeScript | Jest, Vitest, Mocha + Chai |
| | C# / .NET | xUnit.net, NUnit, Moq |
| | Go | testing (built-in), Testify |
| **Integration Tests** | Java | JUnit + Spring Test, REST Assured, Testcontainers |
| | Python | pytest + requests, Testcontainers |
| | JavaScript | Supertest, Jest, Testcontainers |
| | API (any language) | Postman / Newman CLI, Karate DSL, SoapUI |
| **Component Tests** | All (API-based) | WireMock, Testcontainers, Docker, Spring Boot Test |
| **E2E Tests** | JavaScript / TypeScript | Cypress, Playwright, Puppeteer |
| | Multi-language | Selenium WebDriver, Robot Framework, TestCafe |
| | BDD | Cucumber, SpecFlow |
| **Smoke Tests** | All | Postman / Newman, curl, basic Cypress / Playwright scripts |
| **SAST** | Multi-language | SonarQube, Semgrep, Checkmarx |
| | Python | Bandit |
| | JavaScript | ESLint (security plugins) |
| | IaC | KICS, Checkov |
| **DAST** | Language-agnostic | OWASP ZAP, Burp Suite, Invicti, Acunetix |
| **SCA** | Multi-language | Snyk, OWASP Dependency-Check, Dependabot, Black Duck |
| **IAST** | Multi-language | Contrast Security, HCL AppScan, Veracode IAST |
| **Secret Detection** | Multi-language | GitGuardian, TruffleHog, Gitleaks, AWS Git Secrets |

---

## ✨ Key Takeaways

**Tests are your safety net** — they give you the confidence to move fast without breaking things.

**Follow the pyramid** — maximize unit tests, minimize E2E tests. Find bugs at the lowest, fastest, cheapest level possible.

**Automate everything** — in CI, tests run automatically on every commit, saving time and enforcing consistency across the team.

**Fix failures immediately** — a failing CI test is a stop-the-line event. Don't merge broken builds. This is the core discipline of CI.

**Security is everyone's responsibility** — SAST, DAST, SCA, and secret detection are not optional extras. They are standard gates in a mature CI/CD pipeline.

---

<div align="center">

*Part of the [Jenkins Zero to Hero](../README.md) course*

</div>
