# OrderFlow

This repository contains the Release Readiness Investigation & Deployment Approval Lab.

## Application

A simple Node.js Express application.

### Endpoints
- `GET /`
- `GET /health`

## Current Release
v2.3.0
Production Deployment

## Getting Started

### Clone
```bash
git clone https://github.com/kalvium-devops/release-readiness-lab.git
cd release-readiness-lab
```

### Install
```bash
cd app
npm install
```

### Run
```bash
npm start
```
Alternatively, use Docker Compose:
```bash
docker-compose up --build
```

### Push
Make your changes and push them back to the repository.

## Repository Structure
```
├── .github/
│   └── workflows/
│       └── release.yml
├── app/
│   ├── server.js
│   └── package.json
├── deployment/
│   ├── kubernetes/
│   └── terraform/
├── release/
├── scripts/
├── Dockerfile
├── docker-compose.yml
└── README.md
```

# 🚀 OrderFlow v2.3.0 — Complete Assignment Guide (Hinglish)
**Author:** Jayant Kumawat  
**Target:** Release Readiness Engineering (Full Marks Guide)

---

## 📌 Index & Steps Overview

1. [Assignment Ka Goal Aur 7 Problems](#1-assignment-ka-goal-aur-7-problems)
2. [5 Required Screenshots & Exact Captions](#2-5-required-screenshots--exact-captions)
3. [Step 0 & 1: Initial Investigation (Screenshot 1)](#step-0--1-initial-investigation-screenshot-1) — *(Done)*
4. [Step 2 & 3: Pipeline Repair (Screenshot 2)](#step-2--3-pipeline-repair-screenshot-2) — *(Done & Pushed)*
5. [Step 4: GitHub Browser Setup (`production` Environment)](#step-4-github-browser-setup-production-environment) — **(Abhi Yeh Karna Hai)**
6. [Step 5: Secure Release Configuration (Screenshot 3)](#step-5-secure-release-configuration-screenshot-3)
7. [Step 6: Release Readiness Pack & Documentation](#step-6-release-readiness-pack--documentation)
8. [Step 7: Pull Request & Merge](#step-7-pull-request--merge)
9. [Step 8: Tag `v2.3.0` & Production Deploy (Screenshot 4)](#step-8-tag-v230--production-deploy-screenshot-4)
10. [Step 9: Final Release Pack (Screenshot 5)](#step-9-final-release-pack-screenshot-5)
11. [Step 10: Single PDF Kaise Banayein](#step-10-single-pdf-kaise-banayein)

---

## 1. Assignment Ka Goal Aur 7 Problems

Is assignment ka maksad application ko live deploy karna nahi hai, balki repo ko **100% Production-Ready** banana hai. Original repo me ye 7 galtiyan thin:
1. **Broken Pipeline Order**: Deploy pehle chal raha tha, testing baad me.
2. **Hardcoded Failure**: Validation job me intentionally `exit 1` likha tha.
3. **Insecure Secrets**: Plaintext passwords (`password123`, `test-secret`) code me daale the.
4. **Missing Approval Gates**: Bina manager ke approval ke production deploy ho raha tha.
5. **Wrong Versioning**: Version `1.0.0` aur image tag `latest`/`production-build` tha.
6. **Wrong Infrastructure**: Single `t2.micro` EC2 choose kiya tha jabki containerized multi-replica Kubernetes chahiye.
7. **Missing Rollback Plan**: Rollback files aur checklists khali the.

---

## 2. 5 Required Screenshots & Exact Captions

| Screenshot | Kahan Se Lena Hai | Caption (PDF Me Copy-Paste Karein) |
| :--- | :--- | :--- |
| **Screenshot 1** | VS Code (Original `.github/workflows/release.yml`) | `The original pipeline deploys to production before validation, validation always fails, there is no approval gate, secrets are hardcoded, images use latest/production-build, and there is no rollback plan.` |
| **Screenshot 2** | VS Code (Updated `.github/workflows/release.yml`) | `The pipeline now runs Validate → Build → Deploy in order, production only starts after tests and secret scans pass, it uses fixed version v2.3.0, and requires manual approval.` |
| **Screenshot 3** | VS Code Split (`deployment.yaml` + `deployment-config.yml` + Git panel) | `All secrets are removed from code and pulled from GitHub/Kubernetes Secrets, production needs approval through a protected environment, and every file uses traceable version v2.3.0.` |
| **Screenshot 4** | GitHub Actions Browser (`v2.3.0` tag run all 3 green) | `Every stage passed in order – tests, version check, secret scan, versioned image build and smoke test – and production was deployed only after manual approval, making v2.3.0 deployment-ready.` |
| **Screenshot 5** | VS Code Split (`RELEASE_READINESS.md` + `rollback.yml` + `git log`) | `The release now has a single traceable version (v2.3.0), a tested rollback plan to v2.2.0, a completed approval checklist, and a documented Kubernetes infrastructure decision.` |

---

## Step 0 & 1: Initial Investigation (Screenshot 1)
- **Status:** ✅ Completed! Aapne Screenshot 1 le liya hai.

---

## Step 2 & 3: Pipeline Repair (Screenshot 2)
- **Status:** ✅ Completed! Pipeline repair ho chuki hai, GitHub par push ho gayi hai, aur Actions me expected failure (version block) aa chuka hai jo prove karta hai ki validation gate sahi kaam kar raha hai.

---

## Step 4: GitHub Browser Setup (`production` Environment)
👉 **Yeh step abhi browser me karein (2 minute ka kaam hai):**

1. Browser me open karein: `https://github.com/Jayantx07/release-readiness-labs`
2. Upar **Settings** tab par click karein.
3. Left sidebar me **Environments** par click karein.
4. Click **New environment**:
   - Name me likhein: `production` (exact lowercase)
   - Click **Configure environment**
5. **Required reviewers**:
   - Checkbox ko tick karein.
   - Search box me apna username search karein: `Jayantx07` aur select karein.
   - *(Note: "Prevent self-review" ko uncheck hi rehne dein).*
6. **Deployment branches and tags**:
   - Dropdown me **Selected branches and tags** select karein.
   - Click **Add deployment branch or tag rule**:
     - Ref type: **Branch**, Name pattern: `main` ➔ Click **Add rule**.
   - Phir se click **Add deployment branch or tag rule**:
     - ⚠️ Ref type dropdown ko change karke **Tag** karein!
     - Name pattern me likhein: `v*` ➔ Click **Add rule**.
   - Click **Save protection rules**.
7. **Environment secrets** (Page ke niche, Secrets section me):
   - Click **Add secret**:
     - Name: `API_KEY` | Value: `demo-key-value` ➔ Add secret.
   - Click **Add secret**:
     - Name: `DATABASE_PASSWORD` | Value: `demo-db-value` ➔ Add secret.

---

## Step 5: Secure Release Configuration (Screenshot 3)
*Jab aap Step 4 kar lenge, tab hum Step 5 execute karenge.*

Is step me:
- `deployment/kubernetes/deployment.yaml` me secrets ko `secretKeyRef` se link kiya jayega aur image `lab-app:v2.3.0` set hogi.
- `release/deployment-config.yml` me secrets ko `${...}` placeholders me badla jayega.
- `app/package.json` aur `release/release.json` me version `2.3.0` aur Author `Jayant Kumawat` set hoga.
- `deployment/terraform/` me 3 nodes `t3.medium` aur sensitive variables set honge.

### 📸 Screenshot 3 Lene Ka Tarika:
1. VS Code me split view banayein (`Ctrl + \`):
   - Left side: `deployment/kubernetes/deployment.yaml`
   - Right side: `release/deployment-config.yml`
2. Left side me **Source Control** icon par click karein (`Ctrl + Shift + G`) jisme modified files ki list dikhe.
3. **`Windows + Shift + S`** dabakar full window screenshot lein aur `screenshot3.png` save karein.
4. Caption likhein:
   > `All secrets are removed from code and pulled from GitHub/Kubernetes Secrets, production needs approval through a protected environment, and every file uses traceable version v2.3.0.`

Phir terminal me run karenge:
```bash
git add .
git commit -m "fix(release): remove hardcoded secrets, pin version v2.3.0, production infrastructure"
git push
```
*(GitHub Actions me ab Stage 1 Validate ✅ aur Stage 2 Build ✅ dono GREEN ho jayenge!)*

---

## Step 6: Release Readiness Pack & Documentation
Is step me operational release documentation inject ki jayegi:
- `docs/RELEASE_READINESS.md` (Version info, deployment strategy, rollback strategy, infrastructure decision, approval checklist, validation summary).
- `release/approvals.yml` (Jayant Kumawat dwara approved).
- `release/rollback.yml` (Automated rollback to v2.2.0).
- `release/release-summary.yml` (Status: ready-for-production).

Commit & Push:
```bash
git add .
git commit -m "docs(release): add v2.3.0 release readiness pack, rollback plan and approvals"
git push
```

---

## Step 7: Pull Request & Merge (Browser)
1. Browser me jayein: `https://github.com/Jayantx07/release-readiness-labs/pulls`
2. Click **New pull request**.
3. ⚠️ **Base repository** me apna hi repo hona chahiye: `Jayantx07/release-readiness-labs`, base: `main`, compare: `investigation/release-readiness`.
4. Click **Create pull request** ➔ Check pass hone dein.
5. Click **Merge pull request** ➔ **Confirm merge**.
6. Actions tab me `main` branch ka run aayega: Stage 3 par **Waiting** dikhega.
7. Click **Review deployments** ➔ tick **production** ➔ click **Approve and deploy** ➔ all green!

---

## Step 8: Tag `v2.3.0` & Production Deploy (Screenshot 4)
Terminal me run karein:
```bash
git checkout main
git pull
git tag -a v2.3.0 -m "OrderFlow v2.3.0"
git push origin v2.3.0
```

### 📸 Screenshot 4 Lene Ka Tarika:
1. GitHub ➔ **Actions** tab me jayein.
2. Run for tag **`v2.3.0`** open karein.
3. Deploy job par **Review deployments** par click karke **production** tick karein aur **Approve and deploy** karein.
4. Teeno jobs Green hone ke baad browser ka full screenshot lein jisme:
   - Status: **Success**
   - Tag label: **v2.3.0**
   - Teeno connected green jobs: `1 - Validate release` ✅ ➔ `2 - Build versioned image` ✅ ➔ `3 - Deploy to production` ✅
   - Deployment protection rules box me aapka approval dikhe.
5. Save karein: `screenshot4.png`.
6. Caption:
   > `Every stage passed in order – tests, version check, secret scan, versioned image build and smoke test – and production was deployed only after manual approval, making v2.3.0 deployment-ready.`

---

## Step 9: Final Release Pack (Screenshot 5)
Terminal me run karein:
```bash
git log --oneline --decorate -5
```

### 📸 Screenshot 5 Lene Ka Tarika:
1. VS Code split view:
   - Left side: `docs/RELEASE_READINESS.md` ka Markdown Preview (`Ctrl + Shift + V`), scrolled to **5. Production approval checklist**.
   - Right side: `release/rollback.yml`.
   - Bottom me Terminal open rahe jisme `git log --oneline --decorate -5` ka output dikhe (`tag: v2.3.0`).
2. **`Windows + Shift + S`** dabakar full window snapshot lein aur `screenshot5.png` save karein.
3. Caption:
   > `The release now has a single traceable version (v2.3.0), a tested rollback plan to v2.2.0, a completed approval checklist, and a documented Kubernetes infrastructure decision.`

---

## Step 10: Single PDF Kaise Banayein

1. MS Word ya Google Docs open karein.
2. Order me 1 se 5 tak headings daalein:
   - **Screenshot 1 — Initial Deployment Investigation**
     *(Insert `screenshot1.png`)*
     *Caption: The original pipeline deploys to production before validation, validation always fails, there is no approval gate, secrets are hardcoded, images use latest/production-build, and there is no rollback plan.*
   - **Screenshot 2 — Deployment Pipeline Repairs**
     *(Insert `screenshot2.png`)*
     *Caption: The pipeline now runs Validate → Build → Deploy in order, production only starts after tests and secret scans pass, it uses fixed version v2.3.0, and requires manual approval.*
   - **Screenshot 3 — Secure Release Configuration**
     *(Insert `screenshot3.png`)*
     *Caption: All secrets are removed from code and pulled from GitHub/Kubernetes Secrets, production needs approval through a protected environment, and every file uses traceable version v2.3.0.*
   - **Screenshot 4 — Successful Validation**
     *(Insert `screenshot4.png`)*
     *Caption: Every stage passed in order – tests, version check, secret scan, versioned image build and smoke test – and production was deployed only after manual approval, making v2.3.0 deployment-ready.*
   - **Screenshot 5 — Final Release Readiness Pack**
     *(Insert `screenshot5.png`)*
     *Caption: The release now has a single traceable version (v2.3.0), a tested rollback plan to v2.2.0, a completed approval checklist, and a documented Kubernetes infrastructure decision.*
3. **File ➔ Save As ➔ PDF** (ya Google Docs me **File ➔ Download ➔ PDF Document**).

Yeh PDF submit karte hi aapko **Full Marks** milenge! 🎯
