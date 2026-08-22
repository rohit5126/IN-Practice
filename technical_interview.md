##### Here's a full interview question bank, organized the way a recruiter/hiring panel would actually structure a DevOps/SRE loop against your resume — starting with your work experience, then your two projects, then core technical depth checks. Expected answers are tailored to what you've actually claimed, so they're consistent with what you'd need to defend under follow-up.

#### Section 1 — Professional Experience (LTIMindtree)

**Q1: You mention maintaining 98% SLA compliance across 150+ applications — walk me through what SLA compliance actually meant day-to-day, and what happens when you're about to breach one.**

> Day-to-day, SLA compliance meant every ticket in ServiceNow had a response and resolution timer attached based on its severity — P1s had the tightest windows, P3/P4s more relaxed. I kept an eye on our dashboards showing time-remaining-to-breach across my queue, so I could prioritize based on what was closest to breaching, not just what came in first.

> When a ticket was getting close to breaching, I didn't just wait for it to happen — I'd escalate early. That meant looping in L3 or the application vendor if it was outside my scope, reassigning it if it needed a different team's ownership, or proactively communicating with the business stakeholder that resolution was taking longer than expected and giving them a realistic update, rather than letting the SLA silently breach with no warning.

> For the 2% that did breach, I didn't just let it go — those were the ones I'd go back and do a root cause on. Usually it came down to either a dependency on an external vendor's response time, a ticket that got misrouted initially and lost time before reaching the right owner, or a genuinely complex issue that needed more investigation than the SLA window allowed. I'd document those so the pattern could be flagged — if the same type of issue kept breaching, that was a signal we needed a process fix, not just individual effort.

**Q2: Tell me about the critical production printing outage you resolved — what was your actual troubleshooting process, not just the outcome.**

> When we had the critical printing outage, I started by checking Grafana and Prometheus dashboards for any obvious anomalies in metrics, then logged into the affected server directly to verify the actual state of things — checking whether the relevant service was even running.

> After going through the service logs, I found that the printing issue was happening because we weren't getting a response from the separate printing server that hosts that service. So I logged into that server and checked its status, and found the service had unexpectedly stopped — it turned out this happened after a recent OS patching cycle.

> Once I identified that, I restarted the service on the printing server, then restarted the dependent application service on the main server as well, which resolved the outage.

> After that, I did a proper RCA and informed both the patching team and the vendor about the issue. This actually turned out to be a recurring problem — it happened again over the next two patching windows, which occur every month on the third Saturday. I collaborated with the stakeholders and explained the actual root cause, walking them through why this kept happening after every patching cycle. Based on that, we came to the conclusion that the printing server should be removed from the regular patching activity going forward, since patching was directly disrupting the service each time. That decision eliminated the recurrence completely.

**Q3: You automated patch management across 80+ servers with Ansible — what did the playbook actually do, and how did you handle a server where the playbook failed partway through?**

> The playbook handled OS-level patch management across our AWS and Azure servers — applying patch updates on already-existing servers, with no configuration changes involved. So instead of manually logging into 80+ servers one by one to patch each of them, this let us push consistent, standardized patch updates across all of them from one playbook run.

> For servers where the playbook failed partway through, I didn't just rerun it against the whole fleet — I'd first check the Ansible output to see exactly which task failed and on which host, then use --limit to target just that specific server instead of re-running against everything. I'd log into that server directly to check what actually went wrong — sometimes it was a connectivity issue, sometimes a service that didn't come back up cleanly after the patch. Once I fixed the underlying issue, I'd rerun the playbook just for that host to bring it back in line with the rest of the fleet.

> We also didn't patch all 80+ servers at once — we ran it in batches across the refinery, so if something did go wrong, it only affected a subset of servers at a time rather than causing a wider another refiner while patching servers. for example we have multiple refinery across USA so each refinery has its own timezone and we make sure to update patches on each server based on the refinery time at midnight. so as an offshore team member its our responsibility to look after the patching as it falls in our shift timings.

**Q4: What's the difference between a P1 and a P2 incident in your world, and how did escalation actually work across a 24/7 global support model?**

> In our environment, a P1 was a critical, business-impacting incident — something like a full outage or a major application being completely down, which needed an immediate response and had visibility up to leadership. A P2 was a high-impact but not fully-down issue — like degraded performance or a partial service disruption — still urgent, but with slightly more breathing room than a P1.

> For escalation across our 24/7 global support model, since we operated in shifts covering different regions, handoffs between shifts were critical. If an incident wasn't resolved by the end of a shift, we made sure to document exactly where we were in the investigation — what had been checked, what the current findings were, and what the next steps should be — so the next region picking it up wasn't starting from scratch. For P1s especially, we'd stay engaged or join a bridge call with the incoming shift to walk them through it directly rather than just leaving notes.

> If an issue was beyond what I could resolve on my own, I'd escalate it directly to the application vendor, while still keeping ownership of communicating status updates to the stakeholders until it was fully resolved.


#### Section 2 — DevBoard Project

**Q5: Why ArgoCD specifically for this project, instead of just running kubectl apply in a CI pipeline?**

> CI never touches the cluster directly, only Git. ArgoCD is the only thing with cluster write access — smaller blast radius, full audit trail via commit history, and automatic drift correction (selfHeal).

**Q6: You mention Loki and Promtail for centralized logging — why not just use kubectl logs or CloudWatch?**

> kubectl logs only works per-pod, and once a pod restarts or gets rescheduled, those logs are gone — there's no persistence or history across pod lifecycles. It also doesn't let you search across multiple services at once, which becomes a real problem once you have more than a handful of containers running.

> That's why I set up Loki and Promtail instead — Promtail runs as an agent that ships logs from every container to Loki, and Loki indexes them by labels rather than doing full-text indexing, which keeps it lightweight compared to something like Elasticsearch. The bigger benefit was that it plugs directly into the same Grafana dashboards I was already using for Prometheus metrics, so I could correlate logs and metrics in one place instead of switching between tools.

> As for CloudWatch — it works, but it ties you to AWS specifically, and I wanted the logging setup to be consistent regardless of which cloud the workload was running on, since I was already dealing with both AWS and Azure environments at work. Loki gave me that same centralized, searchable logging without being locked into a single cloud provider's tooling.

**Q7: Your CI/CD pipeline went from "manual multi-step" to "single push to deploy" — what are the actual stages in between the push and the deploy?**

> Once code is pushed, the pipeline first runs the build stage — compiling the application and running tests to make sure nothing's broken before going any further. If that passes, it moves on to containerizing the app into a Docker image and pushing that image to the registry.

> After the image is pushed, the pipeline updates the Kubernetes manifest — specifically the image tag — and commits that change back into the Git repo, since ArgoCD is watching that repo as its source of truth. ArgoCD then picks up that change automatically on its next sync cycle and rolls it out to the cluster.

> So the actual gate in the middle is the build and test stage — if that fails, the pipeline stops right there and nothing gets pushed to the registry or deployed. Only a passing build actually makes it through to the image push and the automatic ArgoCD sync.

#### Section 3 — BankApp AI Project (this is where they'll go deepest)

Q8: Walk me through what happens, end-to-end, when a user logs in and hits "transfer" — including where state is stored and why.

> When a user logs in, the request first hits Envoy Gateway, which routes it based on the HTTPRoute to the bankapp Service, and from there to one of the actual application pods. Spring Security handles the authentication — it checks the submitted credentials against the account stored in MySQL, verifying the password against the BCrypt hash.

> Once authenticated, the session isn't kept in that pod's memory — it's persisted to Redis instead. That matters because I'm running multiple replicas of the app behind the Service, and if a user's next request gets routed to a different pod by the load balancer, that pod needs to be able to read the same session back out of Redis and know the user is still authenticated. If sessions were only in local pod memory, the user would essentially get logged out every time their request landed on a different pod.

> When the user hits "transfer," the app doesn't trust any cached balance from login time or from the session — it re-reads the account balance fresh from MySQL inside a transactional method, checks and updates it there, and only then writes the new transaction record. This matters because if you relied on a cached or stale balance, you could end up with race conditions — for example, two transfers happening close together could both read the same outdated balance and both get approved, even if there isn't actually enough money for both. Reading fresh inside a transaction ensures the balance check and the update happen against the true, current state of the database.

**Q9: Why does your CI pipeline block a PR instead of just scanning after merge?**

> My CI pipeline blocks the PR instead of just scanning after merge because I wanted to catch issues before they ever entered the main branch — that's the whole idea of shifting security left. If a vulnerability, a bad code smell, or a leaked secret is only caught after merge, it's already sitting in the Git history and potentially already built into an image by that point — you're cleaning up after the fact instead of preventing it.

> By running Checkstyle, Hadolint, SonarQube, Trivy, and Gitleaks as gates on the PR itself, none of that ever gets a chance to reach main in the first place. It also makes the feedback loop faster for whoever's making the change — they see the failure right on their PR and can fix it immediately, instead of finding out later that something they already merged has to be reverted or patched.

**Q10: You automated MySQL credential rotation with Lambda and Secrets Manager — what would break if the rotation happened while the app was actively serving traffic, and how did you account for that?**

> If rotation happened while the app was actively serving traffic, the main risk is that the app's existing database connections in the pool are still using the old password — so once the rotation completes and the old password is invalidated, any new connection attempts using the stale credential would start failing until the app picks up the new value.

> To account for that, I relied on the secret being synced into the cluster on a refresh interval, so the running pods would eventually pick up the updated credential without a manual restart. I also made sure the connection pool had retry/backoff logic, so a temporary auth failure right after rotation wouldn't just crash the app — it would retry and succeed once the new credential was in place.

> Honestly, this is an area I'd want to harden further in a real production setup. AWS Secrets Manager's rotation actually supports a staged approach with AWSCURRENT and AWSPENDING versions specifically to avoid a hard cutover, and I'd want to make sure my rotation Lambda and the app's credential refresh were properly aligned with that staging so there's zero window where the app is using an invalidated password. I'd also lean toward scheduling rotation during low-traffic windows as an extra safety margin rather than relying purely on retry logic to smooth over the gap.


**Q11: Your load balancer's IP kept changing — explain why, and why "restart the service" isn't a real fix.**

Expected answer: Default AWS NLBs don't reserve static IPs — a new one gets provisioned whenever the LB's underlying network interfaces change. Restarting doesn't address the root cause; the actual fix was installing the AWS Load Balancer Controller and binding pre-allocated Elastic IPs to the NLB so the IP is permanently reserved, not dynamically assigned.

**Q12: Why Terraform to bootstrap ArgoCD instead of just installing ArgoCD manually once?**

> I used Terraform to bootstrap ArgoCD because relying on a manual one-time install means that step is something you have to remember to do correctly every single time the cluster gets rebuilt — and in my case, since I was iterating on this project and occasionally tearing down and recreating the cluster, a manual step like that is exactly the kind of thing that gets forgotten or done inconsistently.

> By codifying it in Terraform, running terraform apply alone takes me from a fresh cluster to a fully self-reconciling ArgoCD setup with the root app already applied — no tribal knowledge required, and no risk of skipping a step or doing it slightly differently the second time around. It also keeps the whole bootstrap process version-controlled and repeatable, which fits the same principle as the rest of the platform — nothing should depend on someone remembering a manual command.

**Q13: What's the actual security boundary between your CI pipeline and your Kubernetes cluster?**

> The actual boundary is that my CI pipeline never has direct access to the cluster at all — it doesn't hold any cluster credentials, and it can't run kubectl apply or push changes into the cluster directly. All CI does is build and scan the image, push it to the registry, and then commit the updated image tag back into Git.

> ArgoCD is the only thing that actually has write access to the cluster, and it works by pulling changes from Git rather than having anything pushed to it. So even if my CI pipeline's credentials were ever compromised, the blast radius is limited — an attacker couldn't use CI access to directly modify anything running in the cluster, since CI simply doesn't have that permission in the first place. That separation between "CI can only write to Git" and "only ArgoCD can write to the cluster" is really the core of the security boundary.


 #### Section 4 — Core Technical Depth (no specific project reference — tests fundamentals)

**Q14: What's the difference between a Kubernetes Deployment and a StatefulSet, and why did you use a StatefulSet for MySQL?**

> A Deployment is meant for stateless workloads, where all the pods are interchangeable — if one pod dies, Kubernetes just spins up a replacement, and it doesn't matter which specific pod handles a request since they're all identical and don't hold any unique state.

> A StatefulSet is different because it gives each pod a stable identity and stable storage. Each pod gets a predictable name and its own persistent volume through volumeClaimTemplates, and even if the pod restarts or gets rescheduled, it comes back with the same identity and reattaches to the same storage instead of getting a fresh, empty volume.

> That's exactly why I used a StatefulSet for MySQL — a database can't just be treated like an interchangeable, stateless pod. The data has to persist across restarts, and MySQL specifically needs its storage to stay tied to the same instance rather than risking it losing its volume or getting mixed up with another pod's storage if it gets rescheduled.

**Q15: What does selfHeal: true actually do in ArgoCD, and what's a scenario where you wouldn't want it enabled?**

> selfHeal: true means ArgoCD continuously watches the live state of the cluster against what's defined in Git, and if it detects any drift — like someone manually running kubectl edit or kubectl scale directly against a resource ArgoCD manages — it automatically reverts that change back to match Git on the next sync cycle, without anyone having to intervene.

> A scenario where I wouldn't want it enabled is during live debugging or troubleshooting an issue directly in the cluster — for example, if I needed to manually scale down a deployment, tweak an environment variable, or temporarily patch a resource to observe how the system behaves under that change. With selfHeal on, ArgoCD would just revert my manual change back to what's in Git before I even get a chance to properly observe the effect, which makes it hard to do that kind of live investigation. In that case, I'd temporarily disable auto-sync or selfHeal on that specific application, make my manual change, observe the behavior, and then re-enable it once I was done.

**Q16: Explain IRSA (IAM Roles for Service Accounts) in one or two sentences, as if to a non-Kubernetes person.**

> IRSA lets a specific pod in my Kubernetes cluster securely act as a particular AWS identity and call AWS APIs — like reading a secret from Secrets Manager or creating a load balancer — without ever having to store a long-lived AWS access key inside the cluster.

> Instead, the trust is set up through the cluster's OIDC identity provider, so AWS can verify that the request is really coming from that specific pod's service account before letting it assume the role.

**Q17: What's the actual difference between what Trivy, SonarQube, and Gitleaks each catch — why do you need all three?**

> Gitleaks scans for hardcoded secrets and credentials that might have accidentally been committed into the code — things like API keys, passwords, or tokens sitting in plain text in a file.

> SonarQube focuses on code quality — things like maintainability issues, code smells, complexity, and some security anti-patterns in how the code itself is written, independent of any external dependencies.

> Trivy is different again — it scans for known CVEs, both in the project's dependencies and in the actual container image layers, so it catches vulnerabilities coming from third-party libraries or the base image itself, not issues in the code I wrote.

> I need all three because they cover completely different risk areas — a clean SonarQube scan doesn't mean there isn't a leaked secret sitting in a config file, and a clean Gitleaks scan doesn't mean one of my dependencies doesn't have a known critical vulnerability. Each tool is really only looking at its own specific slice of the problem, so skipping any one of them leaves a real gap.

**Q18: If Prometheus shows a pod's memory climbing steadily until it OOMKills, walk me through your diagnosis process.**

> First, I'd look at the memory graph in Grafana to understand the actual pattern — whether it's a steady, continuous climb that never comes back down (which points to a leak), a sudden spike tied to a specific event, or just a gradual increase that lines up with genuinely higher traffic or load.

> Then I'd run kubectl describe pod on the affected pod to confirm it was actually OOMKilled and check the exit code and event history, just to rule out anything else going on at the same time, like a node-level resource issue.

> Since this is a Spring Boot app, I'd also check the JVM heap metrics through actuator/Micrometer to see if the heap itself is what's climbing, or if it's something outside the JVM heap, like native memory or too many open connections being held. I'd correlate the timing of the climb with anything recent — a new deployment, a config change, or a traffic pattern shift — to narrow down if this started after a specific change.

> Based on all that, I'd decide between two paths — if it really is a genuine leak in the application code, that needs an actual code fix, not just more memory. But if it's legitimate increased load that the current resource limits just can't handle, then raising the memory limits would be the right call. I only raise limits as a real fix when the usage pattern actually supports that being the right call, not as a way to avoid investigating further.
