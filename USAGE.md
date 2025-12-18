# 📘 Maatify CI — Usage Guide

This document explains **how to use** the official Maatify CI workflows
defined in this repository.

> For architectural rules and enforcement, see **README.md**.

---

## 🎯 Purpose of This File

* Show **how to consume** centralized workflows
* Eliminate ambiguity for contributors
* Provide copy-paste safe examples
* Avoid duplicating CI logic in projects

This is **not** a GitHub Actions tutorial.

---

## 🧩 Available CI Workflows

| Workflow             | Purpose                                     |
| -------------------- | ------------------------------------------- |
| `library-ci.yml`     | Default CI for libraries (unit-only)        |
| `integration-ci.yml` | Optional CI for real services & integration |

---

## ✅ Using Library CI (Default)

Use **Library CI** for:

* Infrastructure libraries
* Core packages
* DTO / config / utility libraries
* Any project with **unit tests only**

---

### 📌 Example — Standard Library CI

Create the following file in your project:

```
.github/workflows/ci.yml
```

```yaml
name: CI

on:
  push:
    branches: [ main, dev ]
  pull_request:
    branches: [ main, dev ]

jobs:
  ci:
    uses: maatify/.github/.github/workflows/library-ci.yml@main
    secrets:
      TELEGRAM_CI_BOT_TOKEN: ${{ secrets.TELEGRAM_CI_BOT_TOKEN }}
      TELEGRAM_CI_CHAT_ID: ${{ secrets.TELEGRAM_CI_CHAT_ID }}
```

---

### ✔ What This Enables

* PHPStan (blocking)
* PHPUnit unit tests
* Coverage reporting
* GitHub Summary
* Telegram notification
* Execution duration tracking

---

### ❌ What You Must NOT Add

Do **not** add:

* Docker services
* Database credentials
* Environment variables
* Integration test suites

> If you need any of the above, use **Integration CI instead**.

---

## ✅ Using Integration CI (Explicit & Optional)

Use **Integration CI** only when:

* Tests require real databases or services
* Integration or end-to-end behavior must be validated
* Validation is needed before release

---

### 📌 Example — Integration CI

Create a separate workflow:

```
.github/workflows/integration.yml
```

```yaml
name: Integration Tests

on:
  workflow_dispatch:
  push:
    tags: [ "v*" ]

jobs:
  integration:
    uses: maatify/.github/.github/workflows/integration-ci.yml@main
    secrets:
      TELEGRAM_CI_BOT_TOKEN: ${{ secrets.TELEGRAM_CI_BOT_TOKEN }}
      TELEGRAM_CI_CHAT_ID: ${{ secrets.TELEGRAM_CI_CHAT_ID }}
```

---

### ✔ What This Enables

* Real Docker services (MySQL / Redis / Mongo)
* Real DSNs and credentials
* Schema / data initialization
* Integration test suites
* Telegram notifications
* Execution duration tracking

---

### ⚠ Optional Capabilities

Integration CI **may** additionally enable:

* PHPStan
* Coverage reporting
* Debug service checks

These options must be:

* Explicit
* Justified
* Used sparingly

See **README.md** for rules governing these options.

---

## ❌ Invalid Usage Examples

### 🚫 Mixing CI Types

```yaml
jobs:
  ci:
    uses: maatify/.github/.github/workflows/library-ci.yml@main
    services:
      mysql:
        image: mysql:8.0
```

❌ **Invalid**
Library CI must never start services.

---

### 🚫 Modifying Centralized Workflows

```yaml
jobs:
  ci:
    uses: maatify/.github/.github/workflows/library-ci.yml@main
    steps:
      - run: echo "custom step"
```

❌ **Invalid**
Centralized workflows must not be modified locally.

---

## 🧭 Quick Decision Guide

Use **Library CI** if:

* Tests are unit-only
* No real services are required

Use **Integration CI** if:

* Docker, DSNs, or real connections are required

There is **no third option**.

---

## 🔒 Important Rule

> **If you feel the need to modify a centralized workflow to make it work,
> you are using the wrong workflow.**

---

## 🧠 Final Note

* **README.md** defines policy and enforcement
* **USAGE.md** shows correct consumption patterns

Both are required for a stable CI ecosystem.

---

© 2025 Maatify.dev
CI Usage maintained by **Mohamed Abdulalim**

---

