**Production support + automation + cloud + observability → DevOps/SRE transition**

#### interview introduction

> Hi, I’m Rohit. I am a Production-focused infrastructure engineer transitioning into DevOps/SRE, with hands-on experience in automation, observability, containers, Kubernetes, IaC and CI/CD.", primarily working across AWS and Azure environments.
In my current role at LTIMindtree, I support more than 150 production applications in a 24/7 environment. My responsibilities involve monitoring application health using Grafana and Prometheus, troubleshooting production incidents, managing Linux and cloud infrastructure, and working with application and vendor teams during critical P1 and P2 incidents.
Over time, I started focusing more on automation and infrastructure. I’ve used Python, Bash and Ansible to automate health checks, troubleshooting workflows, server configuration, patch management and access provisioning across AWS and Azure servers. This helped reduce manual effort and improve consistency in production operations.
Alongside my professional experience, I’ve been actively transitioning toward DevOps and SRE. I’ve worked hands-on with Docker, Kubernetes, Amazon EKS, Helm, Terraform, GitHub Actions, Jenkins, ArgoCD and DevSecOps tools such as Trivy, SonarQube and Gitleaks.
For example, in my DevBoard project, I containerized a three-tier application using Docker, deployed it on Kubernetes, implemented GitOps using ArgoCD, and added centralized logging using Loki and Promtail. In another project, I deployed a Spring Boot application on EKS using Helm and built a DevSecOps CI/CD pipeline with security and code-quality gates.
So, my background is a combination of production support, cloud infrastructure, automation and observability, and I’m now looking to move into a dedicated DevOps/SRE role where I can apply these skills to building and operating reliable, automated infrastructure and deployment platforms.


#### day-to-day job


> My current role is primarily focused on production application and infrastructure support, but a significant part of my day-to-day work overlaps with DevOps and SRE practices.

> I work in a 24/7 production environment supporting more than 150 applications running across AWS and Azure. My day generally starts with checking the health of production applications and infrastructure using Grafana and Prometheus dashboards. I look for abnormal CPU or memory utilization, application errors, availability issues, latency and other indicators that could potentially affect users.

> If I identify an issue, I start troubleshooting from the infrastructure and application layers. Depending on the problem, I check Linux processes, services, network connectivity, ports, logs, resource utilization and application health. I use commands such as systemctl, journalctl, ps, top, df, free, ss and curl, along with application and monitoring logs.

> For cloud-related issues, I investigate AWS or Azure resources, connectivity, security rules, instance health and configuration. For example, if an application cannot communicate with another service, I verify whether the service is running, whether the required port is listening, and whether firewall, security-group or network rules are allowing the required traffic.

> A major part of my role is incident management. I handle around 20–30 ServiceNow tickets daily and participate in P1/P2 incident resolution. During a critical incident, my approach is to first identify the impact and scope, then isolate the failing component, restore service as quickly as possible, and finally perform root-cause analysis to prevent recurrence.

> Automation is another important part of my work. Whenever I see a repetitive operational task, I try to automate it using Python, Bash or Ansible. For example, I have automated health checks, troubleshooting workflows, server configuration, OS patching and access provisioning across AWS and Azure servers.

> From a DevOps perspective, I would map my current responsibilities into several areas.

> **1. Linux & Infrastructure**

> I work with Linux servers for application troubleshooting, service management, process monitoring, filesystem and resource checks, networking and configuration troubleshooting.

> **2. Cloud**

> I work with AWS and Azure infrastructure, including server configuration, connectivity, access controls and troubleshooting cloud-hosted applications.

> **3. Monitoring & Observability**

> I use Grafana and Prometheus to monitor application and infrastructure health. I investigate metrics and alerts to identify performance and availability issues.

> **4. Incident Management / SRE**

> I handle production incidents, especially P1/P2 issues, where the focus is restoring service quickly, identifying root cause and preventing recurrence. This has given me practical experience with reliability and operational ownership.

> **5. Automation**

> I use Python, Bash and Ansible to automate repetitive operational activities such as health checks, configuration, patching and provisioning.

> **6. Networking**

> During troubleshooting, I regularly investigate connectivity issues by checking ports, services, firewall rules, CIDR access and communication between systems.

> **7. Configuration & Change Management**

> I manage application configuration, version upgrades, license renewals and infrastructure changes while following change-management and production deployment processes.

> **8. DevOps / CI/CD**

> Although CI/CD is not the primary responsibility of my current production support role, I have built hands-on CI/CD experience through my projects using GitHub Actions and Jenkins, including automated build, testing, security scanning and deployment workflows.

> **9. Containers & Kubernetes**

> I have gained hands-on experience outside my current production role with Docker, Kubernetes, EKS and Helm. I've containerized applications, created Kubernetes deployments and services, worked with EKS infrastructure and deployed applications using Helm.

> **10. GitOps**

> In my DevBoard project, I used ArgoCD to implement GitOps. Kubernetes manifests were maintained in Git as the source of truth, and ArgoCD handled synchronization and drift reconciliation.

> So overall, my current role gives me strong production operations, troubleshooting, cloud, monitoring and automation experience, while my projects have helped me build the containerization, Kubernetes, IaC, CI/CD, GitOps and DevSecOps skills required for a dedicated DevOps/SRE position.


#### "Walk me through your pipeline."  what happens when each stage fails.


#### Similarly, for EKS, expect questions around:

EKS architecture
Node groups
Pods
Services
Ingress
ALB
EBS CSI driver
IAM/IRSA
Security groups
VPC
Terraform
Helm
Kubernetes troubleshooting

#### You can talk about:

What do you do when the application is down? How do you determine whether it's the application, container, Kubernetes service, node, network, firewall, cloud infrastructure or dependency?

Your interview positioning should therefore be:

4 years production support
→ AWS/Azure
→ Linux
→ Monitoring/Observability
→ Incident/RCA
→ Automation
→ Docker
→ Kubernetes/EKS
→ Terraform
→ CI/CD
→ GitOps
→ DevSecOps
→ DevOps/SRE