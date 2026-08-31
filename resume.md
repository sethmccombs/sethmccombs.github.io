---
layout: page
title: Resume
subtitle: Work Experience
---

### Greybeam ([greybeam.ai](greybeam.ai))
**Founding Infrastructure Engineer**  
**October 2025 – August 2026**

- Led infrastructure operations across AWS, including EKS, RDS, VPC networking, load balancing, and S3.
- Expanded Greybeam's platform into GCP, building and operating infrastructure using GKE, Cloud SQL, and Cloud Storage.
- Introduced and standardized infrastructure tooling across several areas:
  - Automation/Deployment: [ArgoCD](https://argoproj.github.io/cd/), [Terragrunt Scale](https://terragrunt.com/terragrunt-scale)
  - Observability: [kube-prometheus](https://github.com/prometheus-operator/kube-prometheus), [Datadog](https://www.datadoghq.com)
  - Security & Policy: [conftest](https://www.conftest.dev), [Kusari Inspector](https://www.kusari.dev/inspector/), GitHub Dependabot, [Cert-Manager](https://cert-manager.io)
- Partnered with Greybeam's co-founders to establish infrastructure and security practices supporting the company's SOC 2 certification. 
- Planned and executed zero-downtime upgrades across critical infrastructure services and tooling, including:
  - Istio Ambient Mesh and Gateway components
  - RDS version upgrades
  - Terraform and OpenTofu versions, and AWS Terraform providers 
- Implemented [Karpenter](https://karpenter.sh)/Spot nodes in AWS to save 30%+ on Cloud Costs per customer
- Sole on-call engineer for all after hours issues/alerts

---

### **AcuityMD** ([acuitymd.com](https://acuitymd.com))
**Senior SWE 2 / Infra Tech Lead**  
**November 2022 - October 2025** (promoted to Senior 2 in February 2024)

- Led infrastructure engineering and platform strategy at AcuityMD.
- Drove the migration toward a GKE-based platform, consolidating workloads previously distributed across GCP container and managed services including Cloud Run, Cloud Functions, and App Engine.
- Led the migration from GCP Cloud Build to GitHub Actions for CI and established [ArgoCD]([ArgoCD](https://argoproj.github.io/cd/))-based GitOps workflows for application deployments.
- Introduced automation tooling to improve developer workflows, including [Atlantis]([Atlantis](https://www.runatlantis.io)) for Terraform automation and [Tilt]([Tilt](https://tilt.dev)) for local Kubernetes development.
- Built an internal CLI that enabled application engineers to independently troubleshoot Kubernetes microservices and cloud-hosted services.
- Improved platform networking, security, and reliability through the adoption of Istio Gateway and GCP Gateway technologies.
- Established infrastructure and application observability using ([prometheus](https://prometheus.io)) and [Datadog](datadog.com)
- Helped define and implement SLI/SLO-based reliability metrics for platform and component availability.
- Participated in the infrastructure on-call rotation and served as escalation point for CICD and platform issues

---

### **Crunchyroll** ([crunchyroll.com](https://www.crunchyroll.com))
**Senior DevOps Engineer / Tech Lead**  
**April 2022 – November 2022**

- 1 of 3 tech leads supporting application engineers on worldwide team
- Helped lead EKS based platform "reconception"
    - Istio rollout
    - Migration from VM based services to full containerization
- Built/Maintained Cloud Infra with Terraform and Cloudformation
- Supported CICD systems - Jenkins, Spinnaker and CircleCI
- Helped architect the "future of Crunchyroll" using GCP services


---

### **Workday** ([workday.com](https://www.workday.com))
**Software Development Engineer III – Public Cloud SRE**  
**November 2020 – April 2022**

- Member of Public Cloud Platform team, responsible for
  - Operation and maintenance of self-managed Kubernetes platform hosted in Amazon Web Services
  - Support for deployment infrastructure for weekly platform and core application patches
- Participated in rewrite of all internal documentation for new environment builds
- Designed new support model for cross-team engagements
- CI/CD automation and Kubernetes platform infrastructure
- Introduced and drove cross-team initiatives modeled on Kubernetes SIGs
- Co-led secure image pipeline project for FedRAMP compliance 
- Bi-weekly on-call rotation for issues in automated deployments/patching Kubernetes environments


---

### **Sysdig** ([sysdig.com](https://sysdig.com))
**Infrastructure Engineer**  
_Nov 2019 – Nov 2020_

* Managed AWS and IBM Cloud environments
* Lead Jenkins admin; standardized CI workflows
* Improved developer autonomy through self-service tooling and documentation

---

### **Triller** (Short Form Video App)
**Senior Site Reliability Engineer**  
_Mar 2019 – Nov 2019_

* Full infrastructure ownership, AWS & Kubernetes-based
* Transitioned to Infrastructure as Code
* Increased visibility and reliability with OSS observability tools
* Standardized infra/tools using widely adopted OSS solutions

---

### **Veracode / SourceClear** ([veracode.com](https://www.veracode.com))
**DevOps Engineer**  
_May 2018 – Feb 2019_

* Managed GitLab-based CI/CD systems
* Led Kubernetes adoption initiatives
* Oversaw SourceClear’s Kubernetes clusters and post-acquisition integrations

**IT Ops Engineer**  
_Nov 2017 – May 2018_

* Led MDM implementation and G-Suite/2FA management
* Automated onboarding and managed global VPN infrastructure  