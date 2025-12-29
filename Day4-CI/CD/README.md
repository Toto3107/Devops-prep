# 📘 DevOps Production Prep – Day 4

## CI/CD Pipelines in Production: Real Failures, Bugs & Fixes

> **Objective:**
> Learn how CI/CD pipelines break in real companies, why outages happen due to pipelines, and how DevOps engineers debug, fix, and prevent them.

This is **not Jenkins/GitHub Actions syntax learning** — this is **incident handling**.

---

## 🧠 Why CI/CD Is the #1 Production Risk

In production:

* Code ≠ Deployment
* A **bad pipeline can deploy broken code perfectly**
* 90% outages are **automation mistakes**

---

## 🏗️ Typical Production CI/CD Flow

1. Developer pushes code
2. CI:

   * Build
   * Test
   * Dockerize
3. CD:

   * Deploy to staging
   * Deploy to production
4. Monitoring + rollback

Failures can occur at **any step**.

---

## 🚨 Incident 1: Pipeline Passed but App Crashed in Prod

### ❌ Symptom

* CI green ✅
* Prod app down ❌

### 🧠 Root Cause

* Tests mocked environment
* No production-like config in CI

### 🔍 Diagnose

Check pipeline logs:

```bash
CI=true npm test
```

### ✅ Fix

* Add **integration tests**
* Use production-like ENV in CI

📌 **Lesson:**

> Green pipelines don’t guarantee working production.

---

## 🚨 Incident 2: Secrets Leaked via CI Logs

### ❌ Symptom

```text
DB_PASSWORD=supersecret123
```

### 🧠 Root Cause

* `echo $DB_PASSWORD`
* Debug logging enabled

### 🔍 Diagnose

Review pipeline logs

### ✅ Fix

* Mask secrets
* Disable debug in prod

📌 **Lesson:**

> CI logs are public inside organizations.

---

## 🚨 Incident 3: Pipeline Works on Main, Fails on Release Branch

### 🧠 Root Cause

* Branch-specific configs
* Hardcoded branch names

### 🔍 Diagnose

```yaml
if: branch == "main"
```

### ✅ Fix

```yaml
if: startsWith(github.ref, 'refs/heads/')
```

📌 **Lesson:**

> Pipelines must be branch-agnostic.

---

## 🚨 Incident 4: Half Deployment (Partial Rollout)

### ❌ Symptom

* Some users see new version
* Others see old version

### 🧠 Root Cause

* No atomic deployment
* Manual restart

### ✅ Fix

* Blue/Green or Rolling deployment
* Health checks before traffic switch

📌 **Lesson:**

> Partial deployments are worse than downtime.

---

## 🚨 Incident 5: Rollback Failed During Outage

### ❌ Symptom

* New version broken
* Rollback also broken

### 🧠 Root Cause

* Rollback not tested
* DB migrations irreversible

### 🔍 Diagnose

```bash
kubectl rollout history
```

### ✅ Fix

* Always test rollback
* Version DB migrations

📌 **Lesson:**

> If rollback isn’t tested, it doesn’t exist.

---

## 🚨 Incident 6: Pipeline Too Slow (45–60 mins)

### 🧠 Root Cause

* No caching
* Rebuilding everything every run

### 🔍 Diagnose

Check build times per stage

### ✅ Fix

* Cache dependencies
* Parallel jobs

📌 **Lesson:**

> Slow pipelines reduce developer productivity.

---

## 🚨 Incident 7: Pipeline Failed Due to Git Conflict (Your Case)

### ❌ Symptom

```text
! [rejected] non-fast-forward
```

### 🧠 Root Cause

* Remote branch ahead
* Local branch outdated

### ✅ Fix (Production-safe)

```bash
git pull origin Main --rebase
git push origin Main
```

📌 **Lesson:**

> CI/CD assumes a clean git history.

---

## 🚨 Incident 8: Wrong Code Deployed to Production

### 🧠 Root Cause

* No environment separation
* Same pipeline for all envs

### ✅ Fix

* Separate pipelines:

  * dev
  * staging
  * production
* Manual approval for prod

📌 **Lesson:**

> Production must always require human confirmation.

---

## 🧠 CI/CD Production Rules

1. Pipelines must fail fast
2. Secrets must never appear in logs
3. Rollbacks must be automated
4. Prod deployments need approvals
5. Pipelines must be reproducible
6. CI ≠ Production environment
7. Logs are part of security

---

## 🎯 Interview Questions & Answers

**Q: Why do pipelines pass but prod fails?**
A: CI environment differs from production.

**Q: What is a safe deployment strategy?**
A: Blue/Green or Rolling with health checks.

**Q: Why are rollbacks critical?**
A: Deployments fail more often than code.

**Q: How do you protect secrets in CI?**
A: Secret managers + masked logs.

