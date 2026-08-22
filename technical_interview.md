##### Here's a full interview question bank, organized the way a recruiter/hiring panel would actually structure a DevOps/SRE loop against your resume — starting with your work experience, then your two projects, then core technical depth checks. Expected answers are tailored to what you've actually claimed, so they're consistent with what you'd need to defend under follow-up.

#### Section 1 — Professional Experience (LTIMindtree)

**Q1: You mention maintaining 98% SLA compliance across 150+ applications — walk me through what SLA compliance actually meant day-to-day, and what happens when you're about to breach one.**

```
Expected answer: Explain the mechanics — SLAs tied to ticket severity/response and resolution time windows tracked in ServiceNow, dashboards showing time-remaining-to-breach, and your process for triaging/escalating tickets nearing SLA to avoid breach (reassigning, escalating to L3/vendor, communicating proactively with the business). Mention that "98%" implies you tracked breaches and did root-cause on the 2% — recruiters will probe if that number is real or vague.
```

**Q2: Tell me about the critical production printing outage you resolved — what was your actual troubleshooting process, not just the outcome.**

Expected answer: Structured RCA walkthrough: reproduced/confirmed the issue, checked logs/metrics (Grafana/Prometheus) to isolate whether it was infra, network, or application-layer, identified the failing service, restarted it as a mitigation, then worked with the vendor for the actual root cause, and implemented a longer-term fix or monitoring check to prevent recurrence — tie the "zero recurrence over 6 months" claim to a specific preventive action (e.g., added an alert, added a health check, or documented a runbook).

Q3: You automated patch management across 80+ servers with Ansible — what did the playbook actually do, and how did you handle a server where the playbook failed partway through?

Expected answer: Describe idempotent playbook design (safe to re-run), the actual tasks (OS patching, firewall/CIDR provisioning), and your failure-handling approach — Ansible's --limit to target just the failed host, checking ansible-playbook output/logs for the failing task, and whether you used tags or serial batching to avoid patching all 80 servers simultaneously and risking a fleet-wide outage.

Q4: What's the difference between a P1 and a P2 incident in your world, and how did escalation actually work across a 24/7 global support model?

Expected answer: P1 = full outage/critical business impact, immediate all-hands response, executive visibility; P2 = degraded but not fully down, standard escalation path. Describe handoff between shifts/regions (e.g., APAC → EMEA → AMER) and how you ensured continuity — shared runbooks, ticket notes, shift handover calls.

Section 2 — DevBoard Project

Q5: Why ArgoCD specifically for this project, instead of just running kubectl apply in a CI pipeline?

Expected answer: ArgoCD gives continuous drift detection and self-healing — if someone manually changes something in the cluster, ArgoCD reverts it back to match Git. A CI-driven kubectl apply is a one-time push with no ongoing reconciliation loop and no single source of truth if the cluster drifts.

Q6: You mention Loki and Promtail for centralized logging — why not just use kubectl logs or CloudWatch?

Expected answer: kubectl logs only works per-pod and logs disappear when pods are recycled; you need cross-service, searchable, retained logs. Loki indexes logs by label (cheaper than full-text indexing like Elasticsearch) and pairs naturally with Prometheus/Grafana for a unified observability stack, while Promtail is the agent that ships container logs into Loki.

Q7: Your CI/CD pipeline went from "manual multi-step" to "single push to deploy" — what are the actual stages in between the push and the deploy?

Expected answer: Should be able to name the concrete stages: build → test → containerize/push image → update manifest/tag → ArgoCD sync — and explain what would block the pipeline (failing tests, build errors) versus what happens automatically (ArgoCD detecting the Git change).

Section 3 — BankApp AI Project (this is where they'll go deepest)

Q8: Walk me through what happens, end-to-end, when a user logs in and hits "transfer" — including where state is stored and why.

Expected answer: Request flow through Envoy Gateway → Service → pod, Spring Security auth against MySQL with BCrypt, session persisted to Redis (not pod memory) for horizontal scalability, transfer re-reads balance fresh from MySQL inside a transaction rather than trusting cached/session data — you should be able to explain why re-reading fresh matters (avoiding stale-balance race conditions).

Q9: Why does your CI pipeline block a PR instead of just scanning after merge?

Expected answer: Shift-left principle — catching a vulnerability or leaked secret before it merges to main means it never enters the Git history or gets built into an image at all; scanning after merge only tells you something's already wrong, it doesn't prevent it.

Q10: You automated MySQL credential rotation with Lambda and Secrets Manager — what would break if the rotation happened while the app was actively serving traffic, and how did you account for that?

Expected answer: This is a real gap-test question. Honest answer: connection pools holding old credentials could fail momentarily until they refresh; mitigations include the app re-reading credentials from the External Secrets Operator sync on a refresh interval, connection pool retry/backoff logic, or coordinating rotation during low-traffic windows. If you haven't fully solved this, say so honestly and describe what you'd add — that's a stronger answer than pretending it's airtight.

Q11: Your load balancer's IP kept changing — explain why, and why "restart the service" isn't a real fix.

Expected answer: Default AWS NLBs don't reserve static IPs — a new one gets provisioned whenever the LB's underlying network interfaces change. Restarting doesn't address the root cause; the actual fix was installing the AWS Load Balancer Controller and binding pre-allocated Elastic IPs to the NLB so the IP is permanently reserved, not dynamically assigned.

Q12: Why Terraform to bootstrap ArgoCD instead of just installing ArgoCD manually once?

Expected answer: Reproducibility — if the cluster is ever destroyed and rebuilt (which you should be honest happens in a demo/learning context), a manual step is something you'll forget or do inconsistently. Codifying it in Terraform means terraform apply alone gets you to a fully self-reconciling state every time, with no tribal knowledge required.

Q13: What's the actual security boundary between your CI pipeline and your Kubernetes cluster?

Expected answer: CI never has cluster credentials or write access — it only builds images and writes to Git. ArgoCD, running inside the cluster, is the only thing with write access to the cluster, and it pulls changes rather than having them pushed to it. This limits the blast radius if CI credentials were ever compromised.

Section 4 — Core Technical Depth (no specific project reference — tests fundamentals)

Q14: What's the difference between a Kubernetes Deployment and a StatefulSet, and why did you use a StatefulSet for MySQL?

Expected answer: StatefulSets give stable network identity and stable persistent storage per pod (via volumeClaimTemplates) — critical for a database where pod identity and its attached disk must survive restarts/rescheduling. Deployments are for stateless, interchangeable replicas.

Q15: What does selfHeal: true actually do in ArgoCD, and what's a scenario where you wouldn't want it enabled?

Expected answer: It automatically reverts any manual change to a cluster resource back to match Git on the next sync. You might disable it temporarily during live debugging where you need to make a manual change and observe its effect without it being auto-reverted mid-investigation.

Q16: Explain IRSA (IAM Roles for Service Accounts) in one or two sentences, as if to a non-Kubernetes person.

Expected answer: It lets a specific pod (via its Kubernetes ServiceAccount) securely assume a real AWS IAM role and call AWS APIs, without ever storing an AWS access key inside the cluster — the trust is established through the cluster's OIDC identity provider instead.

Q17: What's the actual difference between what Trivy, SonarQube, and Gitleaks each catch — why do you need all three?

Expected answer: Gitleaks scans for hardcoded secrets/credentials in code; SonarQube checks code quality/maintainability and some security anti-patterns in source code; Trivy scans for known CVEs in dependencies and container image layers. They cover three non-overlapping risk categories — leaked secrets, code quality, and vulnerable third-party components — so no single tool replaces the others.

Q18: If Prometheus shows a pod's memory climbing steadily until it OOMKills, walk me through your diagnosis process.

Expected answer: Check Grafana memory panel for the trend shape (leak vs. spike vs. legitimate load increase), check kubectl describe pod for OOMKilled events and exit codes, check application-level heap metrics (JVM heap for a Spring Boot app specifically, since that's your stack) via actuator/Micrometer metrics, correlate with recent deploys or traffic patterns, and only then decide between raising resource limits (band-aid) versus investigating a memory leak in code (real fix).