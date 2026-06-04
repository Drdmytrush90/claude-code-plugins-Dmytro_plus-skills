# Dmytro — Story 2.1 Progress Journal

> This file is the memory between sessions.
> Every time we start a new conversation, paste this file in and Claude can pick up exactly where we left off.

---

## Who I am

- GitHub: Drdmytrush90
- Project: Final project — platform engineering course
- Story assigned: **Story 2.1 — Metrics & SLOs**
- Repo: `Drdmytrush90/claude-code-plugins-Dmytro_plus-skills`

---

## What Story 2.1 requires me to build

I am the team's **metrics owner**. I need to:

1. Deploy **Prometheus + Grafana + Alertmanager** on the team's EKS cluster as a locally-vendored Helm chart under `eks-monitoring/`
2. Prometheus scrapes the **cluster + at least one teammate's app**
3. Grafana reads both **Prometheus** (cluster/app metrics) and **CloudWatch** (AWS-managed resources like RDS, ALB)
4. **Dashboards as code** — provisioned from the Helm chart, not clicked by hand
5. Grafana is **access-restricted** — IP allowlist + auth + credentials from AWS Secrets Manager
6. Define **at least one SLO** — SLI, target, measurement window, error budget, and a written response when budget burns
7. Wire my section into `deploy-platform-tools.yaml`
8. Write `README` + `WORKING-WITH-AI.md` memoir

The interview story: *"I owned the metrics stack and defined the SLO that the team treated as the contract between platform and product."*

---

## My 5-Phase Learning Plan

| Phase | What I do | Skills used | Status |
|-------|-----------|-------------|--------|
| 1 | Understand EKS, Helm, Prometheus, Grafana, Alertmanager, CloudWatch | Reading + questions | ✅ DONE |
| 2 | Design the `eks-monitoring/` Helm chart structure + per-env values | `helm-chart-generator`, `helm-values-manager` | ⬜ NEXT |
| 3 | Get metrics flowing — Prometheus scrape configs + CloudWatch IAM role | `prometheus-config-generator`, `iam-role-generator` | ⬜ TODO |
| 4 | Dashboards as code + security (IP allowlist, Secrets Manager) | `grafana-dashboard-creator`, `kubernetes-secrets-manager` | ⬜ TODO |
| 5 | Define and defend the SLO — SLI, target, error budget, response | `alertmanager-rules-config` + my own thinking | ⬜ TODO |

---

## Phase 1 — COMPLETED ✅

### What I learned

**EKS (Elastic Kubernetes Service)**
- Kubernetes = operating system for containers. Runs apps across many servers, restarts crashes, balances traffic.
- EKS = Amazon's managed Kubernetes. AWS runs the "brain" (control plane), I manage the apps.
- I do NOT create the cluster — Story 1.1 owns that. I deploy my monitoring stack ON TOP of it.

**Helm**
- Package manager for Kubernetes — bundles all YAML files into one reusable "chart"
- I need a **locally-vendored** chart (files live in my repo under `eks-monitoring/`) — NOT pulling from internet at deploy time
- Why locally-vendored? If upstream chart changes overnight, my deploy breaks. Owning it = I control the version.

**Prometheus**
- Time-series database. Works by **PULLING** — Prometheus goes to each app every ~15s and asks "what are your metrics?"
- Apps expose a `/metrics` HTTP endpoint with numbers like `http_requests_total`
- Different from Datadog which **pushes** data to Datadog's servers
- I run everything myself → cheaper, but I own the storage and reliability

**Grafana**
- Reads from Prometheus + CloudWatch and turns numbers into dashboards
- Dashboards must be **provisioned as code** (JSON in Git) — if Grafana pod is deleted, dashboards come back automatically

**Alertmanager**
- Receives firing alerts from Prometheus and routes them (Slack, PagerDuty, email)
- Story 2.3 adds its own alert rules ON TOP of my Alertmanager — that's why I deploy it even with minimal rules

**CloudWatch**
- AWS services (RDS, ALB) don't expose `/metrics` — they publish to CloudWatch
- Grafana needs a **CloudWatch data source** + an **IAM role** to read it
- Goal: one Grafana dashboard showing app metrics (Prometheus) AND database metrics (CloudWatch) side by side

### Datadog vs open-source — the PM answer
| | Datadog | Prometheus + Grafana |
|---|---|---|
| Cost | Expensive (per-host billing) | Free (compute cost only) |
| Setup | Minutes | Days |
| Reliability | Vendor managed | I manage it |
| Flexibility | Limited | PromQL is very powerful |
| Data ownership | Datadog holds my metrics | I hold my metrics |

Honest answer: *"We gained cost savings and full control. We gave up a managed service. The bet is that our team can run it ourselves."*

### Key concept: Pull vs Push
- Prometheus **pulls** — it initiates the connection TO the app
- In Kubernetes this matters because apps can come and go dynamically
- Prometheus uses **ServiceMonitors** (a Kubernetes resource) to know which apps to scrape
- The app doesn't need to know about Prometheus at all — it just exposes `/metrics` and waits

---

## Skills saved to this repo

Location: `my-project/skills-reference/`

| Skill pack | What it helps with |
|---|---|
| `02-devops-advanced/` | Helm, Prometheus, Grafana, Alertmanager, Kubernetes configs |
| `13-aws-skills/` | EKS, IAM roles, CloudWatch, security groups |

Individual skills I will use most:
- `helm-chart-generator` — Phase 2: build the eks-monitoring chart
- `helm-values-manager` — Phase 2: per-env values (dev/staging/prod)
- `prometheus-config-generator` — Phase 3: scrape configs and ServiceMonitors
- `iam-role-generator` — Phase 3: IAM role for CloudWatch → Grafana
- `cloudwatch-alarm-creator` — Phase 3: CloudWatch data source wiring
- `grafana-dashboard-creator` — Phase 4: dashboards as code
- `kubernetes-secrets-manager` — Phase 4: Secrets Manager injection for Grafana creds
- `alertmanager-rules-config` — Phase 5: Alertmanager config + SLO alert rule

---

## What is next — Phase 2

Before writing any YAML I need to answer these questions (my own thinking, not Claude's):

1. **What folder structure should `eks-monitoring/` have?** (Helm chart layout)
2. **What changes between dev, staging, and prod?** (hostname, resource limits, retention?)
3. **Which teammate's app will I scrape?** (Need to coordinate with Story 3.1, 3.3, or 3.5 owner)
4. **What is my SLO candidate?** (Start thinking now — what user journey, what target number, why?)

The Phase 2 checkpoint question Claude will ask:
> *"Sketch the folder structure of `eks-monitoring/` on paper before we write a single file. What goes inside a Helm chart?"*

---

## Repo structure (what exists so far)

```
claude-code-plugins-Dmytro_plus-skills/
├── my-project/                         ← Dmytro's working space
│   ├── README.md                       ← project overview
│   ├── PROGRESS.md                     ← THIS FILE — session memory
│   └── skills-reference/
│       ├── 02-devops-advanced/         ← Helm, Prometheus, Grafana, Alertmanager skills
│       └── 13-aws-skills/             ← EKS, IAM, CloudWatch skills
├── skills/                             ← full repo skill library (not mine specifically)
├── plugins/                            ← full plugin library
└── ...                                 ← rest of repo
```

The **real deliverable** will be built in a separate `eks-monitoring/` folder at the repo root (or in the platform-tools repo — to confirm with team).

---

## Security reminder

- **Never paste a GitHub PAT into chat more than once per session**
- Always rotate the token after each session
- Token location on my machine: `~/.zshrc` as `export GITHUB_PAT="..."`
- Generate new token at: `github.com → Settings → Developer settings → Personal access tokens → Fine-grained tokens`
- Scope: Contents Read & Write on `claude-code-plugins-Dmytro_plus-skills` only
- Expiry: 30 days max

---

## How to resume a session with Claude

1. Open a new Claude chat
2. Paste your GitHub token (fresh one, not the old one)
3. Say: *"Here is my progress file"* and paste the contents of this `PROGRESS.md`
4. Claude will know exactly where we are and what to do next

---

*Last updated: 2026-06-04 — completed Phase 1, ready to start Phase 2*
