# 📘 DevOps Production Prep – Day 2

## Git in Production: Branching, Merging, Rebasing & Recovery

> **Objective:**
> Learn Git the way it is **actually used in production**, including **mistakes, recovery, collaboration issues, and CI-friendly workflows**.

This day focuses on **real incidents**, not theory.

---

## 🧠 Why Git Is Critical in Production

Git is not just version control — it is:

* The **source of truth**
* The **trigger for CI/CD**
* The **audit trail** for incidents
* A **collaboration contract** between teams

One wrong Git command can:

* Break pipelines
* Lose production code
* Block deployments

---

## 🛠️ Topics Covered (Day 2)

* Branch naming & standards
* Merge vs Rebase (real usage)
* Non-fast-forward errors
* Fixing wrong repo / wrong branch pushes
* Submodule mistakes
* Safe recovery strategies
* Production Git best practices

---

## 🌱 Branch Naming (Production Standard)

```text
main        → stable, protected
develop     → integration
feature/*   → new work
hotfix/*    → production fixes
release/*   → release prep
```

⚠️ **Avoid**

```text
Main / MASTER / MAIN
```

Git is **case-sensitive**.

---

## 🚨 Production Issue 1: Non-Fast-Forward Push Rejection

### ❌ Error Seen

```text
! [rejected] Main -> Main (non-fast-forward)
Updates were rejected because the tip of your current branch is behind
```

### 🔍 Root Cause

* Remote branch has commits your local branch does not
* Possibly due to:

  * PR merge on GitHub
  * Repo initialized twice
  * Branch rename (`main` ↔ `Main`)

### ✅ Correct Fix (Safe)

```bash
git checkout Main
git pull origin Main
git push origin Main
```

📌 **Production Rule:**

> Never force push on shared branches.

---

## 🚨 Production Issue 2: Merging Wrong Branch into Itself

### ❌ Mistake

```bash
git merge main
```

while already on `Main`.

### 🧠 Why It Happened

Git merged the branch into itself → **no effect**.

### ✅ Correct Merge

```bash
git checkout Main
git merge source-main
git push origin Main
```

---

## 🚨 Production Issue 3: Wrong Repository Pushed

### ❌ Symptom

Push goes to:

```text
Raksh-backend.git
```

instead of:

```text
Test-Devops.git
```

### 🔍 Root Cause

Incorrect `origin` remote URL.

### ✅ Fix

```bash
git remote -v
git remote set-url origin https://github.com/USERNAME/Test-Devops.git
```

---

## 🚨 Production Issue 4: Folder Added as Submodule (160000 mode)

### ❌ Git Output

```text
create mode 160000 Day1-Linux
```

### 🔍 Root Cause

`git init` was run **inside the folder earlier**.

### ✅ Fix

```bash
rm -rf Day1-Linux/.git
git rm --cached Day1-Linux
git add Day1-Linux
git commit -m "Fix: remove accidental submodule"
```

📌 **Production Impact if ignored**

* CI won’t clone code
* Builds fail silently
* Missing files in deployments

---

## 🔁 Merge vs Rebase (Production Truth)

### Merge

```bash
git merge feature/login
```

✔ Keeps history
✔ Safe for shared branches
❌ Messy graph

### Rebase

```bash
git rebase main
```

✔ Clean history
✔ Linear commits
❌ Dangerous on shared branches

📌 **Rule:**

> Rebase locally, merge remotely.

---

## 🚑 Emergency Recovery Scenarios

### 🔹 Undo last commit (not pushed)

```bash
git reset --soft HEAD~1
```

### 🔹 Undo pushed commit (safe)

```bash
git revert <commit-id>
```

### 🔹 View history safely

```bash
git log --oneline --graph --all
```

---

## 🔐 Production Git Best Practices

* Protect `main`
* Require PR reviews
* Disallow force push
* Enable CI checks
* Use meaningful commit messages

---

## 🎯 Interview Questions & Answers

### Q1. Why does Git reject non-fast-forward pushes?

**Answer:**
To prevent overwriting remote commits and losing history.

---

### Q2. When is force push acceptable?

**Answer:**
Only on personal branches or during early repo bootstrapping.

---

### Q3. Difference between revert and reset?

**Answer:**
`reset` rewrites history, `revert` creates a new undo commit.

---

### Q4. Why are submodules risky?

**Answer:**
They require extra cloning steps and often break CI if misused.

---

### Q5. Merge or rebase in production?

**Answer:**
Merge for shared branches, rebase only for local cleanup.

---


