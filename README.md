# 📘 Maatify CI Policy

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
