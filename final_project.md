### project explanation


"For my final project after a 90-day DevOps challenge, I built and deployed a Spring Boot banking application, but the app itself wasn't really the point — it was a vehicle to build a full production-style platform around it, the way a real infra/platform team would.

I provisioned the AWS EKS cluster with Terraform, and had Terraform also bootstrap ArgoCD into the cluster, so there's zero manual setup after terraform apply. From there, everything is GitOps — I used ArgoCD's App-of-Apps pattern so one root Application manages three child apps: the banking app itself, Envoy Gateway for ingress, and a full Prometheus/Grafana monitoring stack. Adding a new component means adding a file to Git, not clicking through a UI.

On the CI/CD side, every pull request runs through a security gate before it can merge — linting, a secret-leak scan with Gitleaks, and dependency and container image vulnerability scans with Trivy. Once merged, the pipeline builds the image, perform scans usig sonarCube and if passed it pushes it to Docker Hub, and automatically commits the new image tag back into the GitOps manifests — so ArgoCD picks it up and deploys it with no one running kubectl apply by hand.

I also handled a couple of things people often skip in personal projects: credentials aren't static — I've got AWS Secrets Manager holding the DB credentials, synced into the cluster via the External Secrets Operator, with a Lambda function rotating the database password on a schedule. And I hit a real production-style problem — my load balancer's IP kept changing daily, which broke my DNS — so I diagnosed that it was because the default AWS load balancer doesn't reserve a static IP, installed the AWS Load Balancer Controller, and pinned Elastic IPs to it so it's now permanently stable.

The whole thing is reproducible from a blank AWS account, too — I wrote Ansible playbooks that turn a brand-new server into a fully tooled jump host in one run."




### What to have ready for the obvious follow-ups


"Why ArgoCD over just running kubectl apply in CI?" → Separation of concerns: CI never touches the cluster directly, only Git. ArgoCD is the only thing with cluster write access — smaller blast radius, full audit trail via commit history, and automatic drift correction (selfHeal).


"Why Envoy Gateway instead of a normal Ingress controller?" → It implements the newer Kubernetes Gateway API, which is more expressive — I use that directly for path-based routing to Grafana.

### Here's a strong, structured answer — written the way you'd actually say it in an interview, followed by the reasoning so you understand why each point lands well.

```
The exact answer

"At my current scale, a lot of my design choices were reasonable for a demo but wouldn't hold up in real production. A few things I'd change:

Database: I'm running MySQL as a StatefulSet inside the cluster. That works for a single-cluster demo, but at real scale I'd move to Amazon RDS Multi-AZ, since I don't want database durability and failover tied to my Kubernetes cluster's lifecycle — if I ever need to tear down or migrate the cluster, my data shouldn't be at risk, and RDS gives me automated backups, point-in-time recovery, and read replicas I'd have to build myself otherwise.

Multi-environment and multi-account setup: Right now everything runs in one AWS account and one cluster. In production I'd split into separate AWS accounts for dev/staging/prod — this limits blast radius, since a misconfigured IAM policy or a bad deploy in dev literally can't touch prod resources. I'd also promote through environments via the same GitOps pipeline — same manifests, different overlays per environment, using something like Kustomize or separate Helm values files, so I'm not maintaining parallel configs by hand.

Progressive delivery instead of direct sync: My current pipeline goes straight from CI to a full ArgoCD sync. At scale, I'd add canary or blue-green rollouts — Argo Rollouts integrates directly with ArgoCD for this — so a bad deploy only affects a small percentage of traffic and gets automatically rolled back based on error-rate metrics, instead of every user hitting a broken version at once.

Security hardening beyond NetworkPolicies: I already have default-deny NetworkPolicies, but at scale I'd add a service mesh like Istio or Linkerd for mutual TLS between every service, plus policy-as-code with OPA/Gatekeeper to enforce things like 'no container runs as root' or 'no image from an unscanned registry' automatically at admission time, not just in CI.

Observability and SLOs: I have Prometheus and Grafana dashboards, but I'd formalize actual SLOs — like 99.9% availability on the login and transaction endpoints — with alerting tied to error budgets, not just raw metrics. I'd also add distributed tracing, since with multiple replicas and a service mesh, tracing a single request across the system becomes much harder without it.

Secrets and compliance: For a real banking system I'd also expect PCI-DSS-style controls — audit logging on every secret access, KMS customer-managed keys instead of AWS defaults, and probably a formal secrets rotation and access review cadence, not just the Lambda rotation I have now.

Basically — my project proves I understand the shape of a production system. At real scale, the changes are less about new tools and more about defense in depth, blast-radius reduction, and treating failure as the default assumption instead of the exception."

```
