# 📘 Maatify CI Policy

---

## ⚡ TL;DR — Maatify CI Policy

* **Library CI is the default** for all libraries: unit tests, PHPStan, coverage, and no external services.
* **Integration CI is optional and explicit**: used only when real databases or services are required.
* **Never mix Library CI and Integration CI** in the same workflow.
* **Integration CI options** (PHPStan, coverage, services) must be enabled intentionally and justified.
* **If Docker, DSNs, or real connections are needed → Integration CI. Otherwise → Library CI.**

---

This repository contains **official, centralized GitHub Actions workflows**
used across the **Maatify ecosystem**.

Its purpose is to **standardize CI behavior**, eliminate duplication,
and enforce clear architectural boundaries between different project types.

---

## 🎯 Core Principle

> **CI must reflect the architectural nature of the project — not convenience.**

Different project types require **different CI responsibilities**.
Mixing them leads to flaky builds, unclear failures, and architectural drift.

---

## 🧱 CI Types in Maatify

Maatify defines **two official CI workflows**:

1. **Library CI**
2. **Integration CI**

Each serves a **distinct purpose** and **must not be merged**.

---

## 1️⃣ Library CI (Default)

### 📁 Workflow

```
.github/workflows/library-ci.yml
```

### 🧩 Intended For

* Infrastructure libraries
* Core libraries
* DTO / config / utility libraries

Examples:

* `infra-drivers`
* `data-adapters`
* `rate-limiter`
* `security-guard` (core phases)

---

### ✅ What Library CI Does

* Runs **PHPStan** (blocking)
* Runs **PHPUnit unit tests**
* Generates **code coverage**
* Publishes **GitHub Summary**
* Sends **Telegram notification**
* Calculates **execution duration**

---

### ❌ What Library CI Must NOT Do

* Start Docker services
* Use environment variables (DSN, credentials)
* Connect to real databases
* Run integration tests
* Perform schema or migration setup

> If any of the above is needed, **Library CI is the wrong workflow**.

---

### 🧠 Key Properties

| Property    | Value           |
| ----------- | --------------- |
| Determinism | High            |
| Speed       | Fast            |
| Network     | None            |
| Secrets     | Telegram only   |
| Trigger     | Every push / PR |

---

## 2️⃣ Integration CI (Explicit & Optional)

### 📁 Workflow

```
.github/workflows/integration.yml
```

### 🧩 Intended For

* Projects with **real external dependencies**
* Integration or end-to-end validation
* Adapter or application-level testing

Examples:

* Applications
* `data-adapters` (integration phase)
* `security-guard` (integration phase)

---

### ✅ What Integration CI Does

* Starts Docker services (MySQL / Redis / Mongo)
* Uses real DSNs and credentials
* Initializes schemas / collections
* Runs **integration tests only**
* Optionally runs PHPStan
* Optionally collects coverage
* Sends Telegram notification
* Publishes execution summary

---

### ❌ What Integration CI Is NOT

* A replacement for Library CI
* A quality gate for every commit
* A fast or deterministic workflow

---

### 🧠 Key Properties

| Property    | Value              |
| ----------- | ------------------ |
| Determinism | Medium             |
| Speed       | Slow               |
| Network     | Required           |
| Secrets     | Required           |
| Trigger     | Manual / Tags only |

---

## 🧭 Decision Rule (Simple)

> **If tests require Docker, DSNs, or real connections → use Integration CI.**
> **Otherwise → use Library CI.**

There is **no third option**.

---

## 📘 CI Usage Guide

Practical usage examples for all official Maatify CI workflows
are documented separately.

This includes:
- How to enable Library CI in a project
- How to enable Integration CI correctly
- Valid triggers and execution patterns
- Common misuses to avoid

➡️ See **[USAGE.md](./USAGE.md)** for full usage instructions.

> This README defines **policy and enforcement**.  
> `USAGE.md` focuses strictly on **how to apply the workflows**.

---

## ⚙️ Integration CI — Options & Configuration

Integration CI supports several **optional capabilities**.
These options exist to adapt the workflow to different project needs,
but **must be enabled intentionally**.

> Integration CI is not a quality gate by default.
> Each option must justify its execution cost.

---

### 🔹 PHPStan in Integration CI

#### When to enable

* Before a release
* When integration changes introduce new logic paths
* When additional confidence is required beyond Library CI

#### When to avoid

* If PHPStan already runs in Library CI
* If integration runtime becomes excessively slow

#### Rules

* If enabled, PHPStan **must be blocking**
* Non-blocking static analysis is **not allowed**

---

### 🔹 Code Coverage in Integration CI

#### When to enable

* To observe **real execution paths**
* To validate behavior not covered by unit tests
* In application or adapter-heavy projects

#### When to avoid

* For pure infrastructure libraries
* When unit test coverage is sufficient
* When CI runtime is a concern

#### Rules

* Integration coverage is **informational only**
* No minimum coverage thresholds
* Coverage results must not block the pipeline

---

### 🔹 Debugging External Services

Integration CI may include **non-blocking diagnostics**
to help troubleshoot service availability.

Examples:

* Redis ping
* MySQL connection check
* MongoDB ping

#### Rules

* Must use `continue-on-error: true`
* Must not affect CI success or failure

---

### 🔹 Schema & Data Initialization

Integration CI may initialize:

* Database schemas
* Indexes
* Collections

#### Rules

* Integration schemas must be isolated
* Production schemas must never be reused
* Initialization scripts must live under test fixtures

---

### 🔹 Telegram Notifications

Telegram notifications are **mandatory** for Integration CI.

They provide:

* Pass / Fail status
* Execution duration
* Repository and reference information
* Direct link to CI logs

---

### 🔹 Execution Duration Tracking

Execution time must always be calculated.

Purpose:

* Detect performance regressions
* Identify stalled or hanging integration runs
* Monitor infrastructure stability

---

## 🧭 Integration Options Summary

| Option         | Default  | Purpose                      |
| -------------- | -------- | ---------------------------- |
| PHPStan        | Off      | Extra static confidence      |
| Coverage       | Off      | Observe real execution paths |
| Debug services | Optional | Troubleshooting only         |
| Schema init    | Optional | Required for DB-backed tests |
| Telegram       | On       | Visibility & alerting        |
| Duration       | On       | Performance monitoring       |

---

## 🔒 Enforcement Rule

> **If an integration option is enabled, it must be justified.**
> Blindly enabling all options is considered misuse.

> Integration CI options exist to increase confidence — not to compensate for missing unit tests.
---

## 🔒 Governance Rules (LOCKED)

* Projects **must not modify** centralized workflows locally
* CI changes are made **only in this repository**
* Any exception requires explicit architectural approval
* Integration CI must never run on every push

---

## 📌 How to Use in a Project

In your project repository:

```yaml
jobs:
  ci:
    uses: maatify/.github/.github/workflows/library-ci.yml@main
    secrets:
      TELEGRAM_CI_BOT_TOKEN: ${{ secrets.TELEGRAM_CI_BOT_TOKEN }}
      TELEGRAM_CI_CHAT_ID: ${{ secrets.TELEGRAM_CI_CHAT_ID }}
```

---

## 🧠 Final Note

> **Library CI protects code quality.**
> **Integration CI validates reality.**

Confusing the two breaks both.

---

© 2025 Maatify.dev
CI Architecture maintained by **Mohamed Abdulalim**

---
