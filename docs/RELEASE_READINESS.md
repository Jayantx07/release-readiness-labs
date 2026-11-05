# OrderFlow v2.3.0 – Release Readiness Pack

| Field | Value |
|---|---|
| Product | OrderFlow |
| Release version | **v2.3.0** |
| Previous version | v2.2.0 |
| Git tag | `v2.3.0` |
| Container image | `lab-app:v2.3.0` (also `lab-app:v2.3.0-<commit-sha>`) |
| Target environment | production |
| Release manager | Jayant Kumawat |
| Release date | 2026-11-05 |
| Status | Ready for production |

---

## 1. Release version information

The version **v2.3.0** is defined once in the pipeline (`RELEASE_VERSION`) and must match:

- `app/package.json` → `"version": "2.3.0"`
- `release/release.json` → `"Version": "2.3.0"`
- `deployment/kubernetes/deployment.yaml` → `image: lab-app:v2.3.0`
- Git tag → `v2.3.0`

The validation stage fails if any of these do not match. Every image is tagged with the
version **and** the commit SHA and carries OCI labels (`image.version`, `image.revision`),
so every deployed artifact can be traced back to one commit. The `/health` endpoint
returns the running version.

## 2. Deployment strategy

- **Pipeline order:** Validate → Build versioned image + smoke test → Deploy to production.
  Each stage uses `needs:` so production cannot run unless the earlier stages pass.
- **Branch rule:** the production job only runs from `main` or a `v*` tag, never from
  feature branches or pull requests.
- **Approval:** the deploy job uses the protected GitHub environment `production`, which
  requires a reviewer to approve before it starts.
- **Kubernetes rolling update:** 3 replicas, `maxUnavailable: 0`, `maxSurge: 1`, with
  readiness and liveness probes on `/health`. New pods only receive traffic once healthy,
  so there is no downtime.
- **Concurrency:** only one deployment per branch or tag at a time.

## 3. Rollback strategy

| Item | Value |
|---|---|
| Method | `kubectl rollout undo deployment/lab-app` (Kubernetes keeps 5 old revisions) |
| Target version | v2.2.0 |
| Script | `ROLLBACK_VERSION=v2.2.0 ./scripts/rollback.sh` |
| Time limit | 5 minutes |
| Owner | Jayant Kumawat |

**Roll back when:** `/health` fails after deploy, the rollout does not finish in 120 s,
or error rate / latency rises after release.

**Verify rollback:** `kubectl rollout status deployment/lab-app` succeeds and `/health`
returns version 2.2.0.

## 4. Deployment infrastructure decision

| Option | Decision | Reason |
|---|---|---|
| Single `t2.micro` EC2 instance (original) | Rejected | Free-tier/burstable size, single point of failure, no rolling update or easy rollback |
| **Kubernetes – 3 replicas on `t3.medium` nodes** | **Chosen** | High availability, self-healing, zero-downtime rolling updates, one-command rollback, native Secrets |

Terraform now takes instance type, node count and AMI as variables, tags every node with
`Environment=production` and `Version=v2.3.0`, and has no hardcoded credentials.

## 5. Production approval checklist

- [x] Validation stage passed (tests, version check, secret scan)
- [x] Versioned Docker image built and smoke tested
- [x] No hardcoded secrets in the repository
- [x] Secrets stored in GitHub `production` environment secrets
- [x] Kubernetes uses `secretKeyRef` instead of plain-text values
- [x] Terraform secret variable marked `sensitive`, no default
- [x] Image tag is immutable (`v2.3.0`), not `latest` / `production-build`
- [x] `production` environment requires reviewer approval
- [x] Production deploy restricted to `main` and `v*` tags
- [x] Rollback plan and script reviewed
- [x] Infrastructure decision reviewed
- [x] Release approved by release manager (`release/approvals.yml`)

## 6. Deployment validation summary

| Check | Stage | Result |
|---|---|---|
| Unit tests (`npm test`) | 1 – Validate | Passed |
| Version consistency (workflow / package.json / release.json / k8s / tag) | 1 – Validate | Passed |
| Hardcoded secret scan | 1 – Validate | Passed |
| Docker image build `lab-app:v2.3.0` | 2 – Build | Passed |
| Smoke test `GET /health` returns version 2.3.0 | 2 – Build | Passed |
| Manual approval (production environment) | 3 – Deploy | Approved |
| Production deploy | 3 – Deploy | Passed |

Evidence: GitHub → Actions → **Release Pipeline** → run for `main` / tag `v2.3.0` (all three stages green).

## Issues found and fixed

| # | Component | Original issue | Fix applied |
|---|---|---|---|
| 1 | Pipeline order | Deploy ran before validation | Reordered: Validate → Build → Deploy with `needs:` |
| 2 | Validation script | `exit 1` hardcoded | Removed artificial failure, runs real tests |
| 3 | Approval gate | None | Added GitHub protected `production` environment |
| 4 | Secrets (K8s) | Hardcoded in `deployment.yaml` | Moved to Kubernetes `secretKeyRef` |
| 5 | Secrets (Config) | Hardcoded passwords in YAML | Externalized to environment variables |
| 6 | Secrets (Terraform)| Default secret key in code | Marked `sensitive = true`, default removed |
| 7 | Versioning | Untracked `1.0.0` / `latest` | Synced all artifacts to `v2.3.0` |
| 8 | Infrastructure | Single `t2.micro` EC2 | Chose Kubernetes (3 replicas, rolling update) |
| 9 | Rollback | Empty script, no plan | Implemented `kubectl rollout undo` to `v2.2.0` |
| 10 | Documentation | Missing release pack | Created `RELEASE_READINESS.md` + approval matrix |
