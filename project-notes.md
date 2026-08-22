# Project Notes — DevSecOps Banking Application
*(Interview / talking-points cheat sheet — not part of the app's public README)*

Use this as prep material: a one-line pitch, the architecture in your own words, the reasoning behind each decision, and answers to the questions an interviewer is likely to ask.

---

## 1. The 30-second pitch

"I forked a demo banking application and used it as a vehicle to build a full production-style DevSecOps platform around it: Spring Boot app on EKS, GitOps deployment with ArgoCD's App-of-Apps pattern, a security-gated CI pipeline (lint, secret scan, dependency scan, image scan) that pushes straight through to a running rollback-able deployment, automated secret rotation with Lambda + Secrets Manager, free automated HTTPS via DuckDNS + cert-manager, and Ansible to reproducibly bootstrap the one jump server an operator needs."

---

## 2. Why each piece exists (the "why", not just the "what")

| Component | Why it's there |
|---|---|
| Spring Boot + MySQL + Redis | A realistic stateful app: auth, money movement, and horizontally-scaled session handling — not a toy stateless demo |
| Ollama / tinyllama | Shows integrating a self-hosted AI feature without sending banking data to a third-party API — data stays in-cluster |
| Envoy Gateway (Gateway API) | Modern replacement for Ingress — more expressive routing (used later for the `/monitoring` path rewrite), and it's where I terminate TLS |
| ArgoCD App of Apps | One Git-tracked root object manages every other Application — I never touch the ArgoCD UI to add a component, I just add a file |
| Terraform bootstrapping ArgoCD | Removes the last manual step (`helm install argocd`) — `terraform apply` alone gets you from nothing to a self-reconciling cluster |
| AWS Secrets Manager + Lambda rotation | Credentials aren't static forever — the DB password actually rotates on a schedule without anyone touching a Kubernetes Secret by hand |
| DuckDNS + cert-manager | Real HTTPS without paying for a domain — shows I understand cert-manager/ACME, not just "I turned on HTTPS in a cloud console" |
| Ansible for the jump server | New EC2 instance → fully tooled operator box in one playbook run, no manually SSH-ing in and installing five CLIs |
| GitHub Actions security gate | Shows a "shift-left" mindset — vulnerable code and leaked secrets are caught *before* merge, not discovered in production |
| NetworkPolicy default-deny | Defense in depth inside the cluster, not just at the edge — even if a pod is compromised, its blast radius is limited |

---

## 3. Request lifecycle (say this out loud fluently)

1. Browser hits the DuckDNS hostname → Envoy Gateway's load balancer, TLS terminated there.
2. `HTTPRoute` sends `/` traffic to `bankapp-svc` → one of N `bankapp` pods.
3. Spring Security filter chain checks the session; unauthenticated users bounce to `/login`.
4. On login, Spring Security queries MySQL, checks the BCrypt hash, and puts the `SecurityContext` into a Redis-backed session — so the *next* request can land on a *different* pod and still be authenticated.
5. Every balance-changing action (`deposit`/`withdraw`/`transfer`) re-reads the account from MySQL inside a `@Transactional` block — never trusts a cached balance.
6. The AI chat widget calls `/api/chat`, which builds a read-only context (username, balance, last 5 transactions) and forwards it to Ollama running in-cluster.

---

## 4. GitOps flow, end to end

1. Developer opens a PR → GitHub Actions runs Checkstyle, Gitleaks, Trivy dependency scan, and a Trivy image scan on a trial build. PR is blocked if anything fails.
2. PR merges to `main` → the image is rebuilt, scanned using sonar cube, tagged, and pushed to Docker Hub.
3. CI commits the new image tag back into the GitOps manifests (`k8s/bankapp-deployment.yml`).
4. ArgoCD's reconciliation loop sees the Git diff and syncs the `bankapp` Application — new pods roll out.
5. If something manual changes the cluster directly (`kubectl edit`), ArgoCD's `selfHeal` reverts it on the next sync — Git, not the cluster, is the source of truth.

**Key line for interviews:** "CI never touches the cluster directly — it only ever writes to Git. ArgoCD is the only thing with write access to the cluster. That separation is the whole point of GitOps."

---

## 5. Likely interview questions + how to answer them

**Q: Why ArgoCD App of Apps instead of just three separate Applications?**
A: One Git-tracked root object is the single place you register new components — you edit a folder instead of clicking through the ArgoCD UI, and it composes cleanly with an AppProject that scopes what those child apps are allowed to touch.

**Q: How does the app stay stateless/horizontally scalable if it uses sessions?**
A: Sessions live in Redis, not pod memory — Spring Session + Redis means any pod can serve any authenticated user's next request.

**Q: How do you avoid leaking the MySQL password in Git?**
A: It's never in Git — it lives in AWS Secrets Manager, is rotated on a schedule by a Lambda function, and is synced into the cluster as the `mysql-secret` value the app reads.
*(Be ready to name the exact sync mechanism you used — e.g. External Secrets Operator vs. a custom Lambda/CI step — since that's the natural follow-up.)*

**Q: What happens if the CI security scans find a critical CVE?**
A: The pipeline fails the PR (or the build-and-push step on merge) — a vulnerable image is never pushed to Docker Hub or referenced in the GitOps manifests, so it can never reach the cluster.

**Q: Why Envoy Gateway instead of a plain nginx Ingress?**
A: It implements the Kubernetes Gateway API, which is more expressive than Ingress — I use that directly for the `/monitoring` path-based route to Grafana with a URL rewrite, which would be clunkier with classic Ingress annotations.

**Q: How do you handle a bad deploy?**
A: `argocd app rollback bankapp <REVISION>`, or just `git revert` the commit that changed the image tag — either way Git stays the source of truth and ArgoCD reconciles to match.

**Q: Why Terraform *and* Ansible — isn't that redundant?**
A: Different jobs. Terraform provisions cloud infrastructure (EKS, IAM/OIDC, Secrets Manager, Lambda) and bootstraps ArgoCD into the cluster. Ansible configures a plain server's *software* (CLIs, credentials, repo clone) so there's a reproducible operator machine to run Terraform/kubectl/ArgoCD commands from.

**Q: What's your default-deny NetworkPolicy actually blocking?**
A: All pod-to-pod traffic not explicitly allowed — so even if one component were compromised, it can't freely reach MySQL, Redis, or Ollama unless a rule explicitly permits it. I apply it last, after confirming the app works, since debugging a default-deny policy from scratch is painful.

**Q: What would you change/improve if you kept working on this?**
Good honest answers to have ready:
- Move from a single shared MySQL StatefulSet to a managed RDS instance for real production use (StatefulSet-in-cluster is fine for a demo, not ideal for a real bank's durability/backups story).
- Add mutual TLS between services with a service mesh, on top of the NetworkPolicy layer.
- Add SAST beyond linting (e.g. Semgrep) alongside the existing Trivy/Gitleaks/Checkstyle checks.
- Externalize the AI assistant's context building/prompt so it's easier to swap `tinyllama` for a larger local model without code changes.

---

## 6. Quick facts to have on the tip of your tongue

- **Stack:** Java 21, Spring Boot 3.4, Spring Security, Spring Data JPA/Hibernate, MySQL 8, Redis, Thymeleaf.
- **Infra:** AWS EKS, Terraform, Envoy Gateway (Gateway API), ArgoCD (App of Apps), Helm, cert-manager, DuckDNS, AWS Secrets Manager, AWS Lambda.
- **CI/CD:** GitHub Actions — Checkstyle, Gitleaks, Trivy (dependency + image scan), Docker Hub, automated GitOps manifest bump.
- **Observability:** kube-prometheus-stack (Prometheus, Grafana, Alertmanager), custom dashboards, `ServiceMonitor`s for app/MySQL/Redis.
- **Security layers:** BCrypt password hashing, PR-gated security scans, image scanning before push, default-deny NetworkPolicy, rotated DB credentials, TLS at the edge.