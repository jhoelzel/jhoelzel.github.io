---
layout: about
title: About me
---

<img src="https://avatars.githubusercontent.com/u/30208763?v=4" alt="Me" style="width: 100px;float: left;padding: 1rem;" />
<h2 style="margin:0;">Hi 👋, I'm Johannes</h2>

**I talk to humans and machines and specialize in golang, kubernetes as well as automating everything!**

As a consultant and developer, I provide services ranging from DevOps to application development and have a history of completing projects on time and within budget. My current focus is on Kubernetes, Golang, and Terraform but I have experience working in various environments.

Growing up with the internet, I uploaded my first websites to Geocities, served as tech-admin for numerous bulletin boards, and have been professionally coding since 2003. After attempting to build my first start-up in 2006, I learned that success would not come simply because I built it. As a result, I earned a bachelor's degree in economical psychology, which has greatly enhanced my team leadership and communication skills and allows me to view products from multiple perspectives.

What began as a small agency offering server hosting in 2007 has since evolved into a full-service consultancy and freelancing business, serving clients around the world. I started working when servers were primarily dedicated, experienced the virtual server revolution, briefly delved into serverless tech, and am now immersed in the Kubernetes ecosystem and its container technology. My background in psychology gives me unique insights into team leadership and communication, and I have a proven track record of leading and creating successful international teams.

My preferred cloud platforms are Azure and AWS, but I have also worked on other platforms, including bare-metal and everything in between. I enjoy tackling complex problems and have experience with multi-cloud deployments.

I am currently offering my services as a freelancer and would be happy to discuss potential opportunities involving Kubernetes. If you're looking for someone with seniority and experience in the field, please feel free to reach out. I have extensive experience with Amazon AWS, Microsoft Azure, Hetzner-cloud, DigitalOcean, Contabo and bare-metal Kubernetes, and I am confident in my abilities and can back them up with experience.

All of this makes me an ideal candidate if you are seeking a senior-level professional who is flexible, collaborative, and hands-on. If you would like to see what I have been up to, please check the projects section on LinkedIn.

## Tailored secure Kubernetes Solutions with Office 365 Integration
I specialize in crafting custom Kubernetes environments like RKE2, k3s, rke2 and (many) more seamlessly integrated with Office 365 for optimal performance and collaboration. By leveraging Azure AD, I provide centralized identity management that simplifies security and access control. My focus extends to ensuring financial compliance with regulations such as GDPR, SOC2, and FINRA, using Microsoft’s built-in tools for encryption, audit, and data protection. The result? Scalable, secure infrastructures that balance technical agility with compliance, tailored to your business needs.

I also specialize in architecting multi-cloud environments across multiple Hosters like AWS, Hetzner, Contabo, Ionos, DigitalOcean and more. By distributing production systems across multiple cloud providers-not just availability zones-security, failover, and uptime are dramatically increased. Through an access proxy, your team experiences a seamless interface, making multi-cluster management effortless and transparent. Access proxies eliminate the need for direct access to host systems, further enhancing security by controlling and monitoring connections centrally. This abstraction layer not only simplifies multi-cloud management but also generates comprehensive audit logs, offering insights into **every** interaction with your infrastructure. They are invaluable for compliance, providing a level of detailed auditability that’s often missed in traditional setups. By implementing access proxies, you maintain robust security and meet compliance requirements effortlessly, ensuring transparency and accountability across all systems.

## Kubernetes Experience

### Cluster Environments & Deployment Models

* Azure Kubernetes Service (AKS)
* Amazon Elastic Kubernetes Service (EKS)
* Bare-metal and on-premises clusters
* Edge-server deployments
* Hetzner Cloud, DigitalOcean, and Contabo
* Multi-availability-zone clusters
* Single-node Kubernetes clusters

### Kubernetes Distributions & Platforms

* Managed Kubernetes: AKS and EKS
* RKE2
* K3s
* Rancher Kubernetes Engine (RKE)
* Rancher Desktop
* Harvester
* Upstream/vanilla Kubernetes

### Custom Operators, Controllers & Automation

* Developed Kubernetes admission controllers for policy enforcement and resource validation.
* Built controllers to discover and annotate existing Kubernetes services.
* Automated application deployment and lifecycle management.
* Created operators for provisioning and managing MySQL databases.
* Implemented schedule-aware deployment scaling based on time and day.
* Developed full application lifecycle automation, including application workloads, RabbitMQ, PostgreSQL, and persistent storage, their backup and restore as well as their cross cluster movement.



### Selected Kubernetes Accomplishments

#### Platform Engineering & Bare-Metal Infrastructure

* Deployed production-grade Kubernetes clusters on bare-metal, on-premises hardware.
* Implemented CSI-compliant storage integrations for bare-metal clusters.
* Built IPv4, IPv6, and dual-stack clusters using both Calico and Cilium.
* Deployed heterogeneous clusters on Hetzner with ARM-based control-plane nodes and mixed ARM64/AMD64 worker nodes.
* Executed cluster-to-cluster migrations with minimal service disruption.

#### Security, Identity & Compliance

* Implemented zero-trust cluster access using Cloudflare Tunnel and Cloudflare Access.
* Enabled security compliance through Teleport access controls and strict audit logging.
* Designed isolated, effectively offline clusters that remained securely accessible through an access proxy, including proxied frontend applications.
* Designed fine-grained Kubernetes RBAC and role-assignment policies for a financial institution.
* Enabled OIDC-based Single Sign-On for Kubernetes clusters and hosted applications.
* Implemented mutual TLS communication using Envoy.

#### GitOps, Automation & Configuration Management

* Managed cluster infrastructure, lifecycle operations, and updates entirely through GitHub Actions.
* Implemented GitOps workflows using GitHub, GitLab, Bitbucket, Argo CD, and CI/CD pipelines.
* Migrated Kubernetes resource management from Helm to Kustomize.
* Automated DNS record and TLS certificate management using ExternalDNS and cert-manager.

#### Networking, Ingress & Service Connectivity

* Integrated Calico with BGP peers to support advanced routing and network connectivity.
* Managed ingress and load balancing for bare-metal clusters using HAProxy VMs and Traefik.
* Configured networking across IPv4-only, IPv6-only, and dual-stack environments.
* Implemented secure service-to-service communication through Envoy and mutual TLS.

#### Storage, Backup & Cost Management

* Managed persistent storage and cluster backups using Longhorn.
* Implemented Kubernetes cost monitoring and optimization using Kubecost.

#### Observability & Operations

* Implemented end-to-end monitoring, logging, and distributed tracing using the Grafana LGTM stack: Loki, Grafana, Tempo, and Mimir.
* Established operational visibility across clusters, infrastructure, and applications.

#### Specialized Workloads

* Enabled and operated AI workloads on Kubernetes.
* Deployed and managed VoIP applications on Kubernetes.
* Working with virtual machines in k8s


## Github Excerpts

1. **Argo Helm Versioner**
   - Utility designed to help manage and maintain Argo CD Applications deployed via Helm charts.
   - [Repository](https://github.com/jhoelzel/argo-helm-versioner)
2. **kube-probesimr**
   - ProbeSim is a lightweight Go application designed to simulate various failure scenarios for Kubernetes liveness and readiness probes.
   - [Repository](https://github.com/jhoelzel/kube-probesim)
3. **go-wait-for-k8s**
   - A utility program written in Go that monitors the readiness of Kubernetes resources like Pods, Jobs, Deployments, StatefulSets, DaemonSets, and ReplicaSets.
   - [Repository](https://github.com/jhoelzel/go_wait_for_k8s)

4. **SpInvalidFileNameFinder**
   - A command-line tool written in Go that helps you find and optionally rename files and folders with invalid names for SharePoint.
   - [Repository](https://github.com/jhoelzel/SpInvalidFileNameFinder)

5. **Consoleman**
   - A command-line utility that acts like Postman but runs in the console. You can use it to send HTTP requests to APIs and inspect the responses.
   - [Repository](https://github.com/jhoelzel/consoleman)

6. **Simpleapp**
   - A simple app that defines a basic Kubernetes app used in trainings, containing a simple MVC structure for packages, a Mux subrouter integration, kube manifests, and an easy-to-learn structure.
   - [Repository](https://github.com/jhoelzel/simpleapp)

7. **Auto Updating base images**
   - This image is based on `mcr.microsoft.com/azure-cli` and integrates `mongodb-tools` in order to easily backup databases in a production AKS.
   - [Repository](https://github.com/jhoelzel/docker-azure_cli-mongodb_tools)

8. **Cronor**
   - A Kubernetes cron job image with one task: change a deployment depending on whether it's day or night. Showcasing how easily the Kubernetes API can be implemented directly into your code in multiple ways.
   - [Repository](https://github.com/jhoelzel/cronor)

9. **Ingress and Egress with the same IP on Azure and Terraform**
   - [Repository](https://github.com/jhoelzel/aks_ingress_egress_same_ip)

10. **DevContainers**
   - A collection of Dockerfiles for various development environments.
   - [Repository](https://github.com/jhoelzel/devcontainer)

### 📩 Latest Blog Posts
<!-- BLOG-POST-LIST:START -->
- `2024-09-05` [All roads will lead you to Azure](https://www.hoelzel.it/compliance/2024/09/05/All-roads-lead-to-azure-eventually.html)
- `2024-09-05` [Gaining Total Control of Your Kubernetes Nodes with Custom Images](https://www.hoelzel.it/kubernetes/2024/09/05/kubernetes-custom-images.html)
- `2024-09-02` [Building Resilience with kube-probesim](https://www.hoelzel.it/kubernetes/2024/09/02/kube-probesim.html)
- `2024-09-01` [go_wait_for_k8s](https://www.hoelzel.it/kubernetes/2024/09/01/go-wait-for-k8s.html)
- `2024-09-01` [Kuberntes Access Proxies](https://www.hoelzel.it/kubernetes/2024/09/01/k8s-access-proxy.html)
- `2024-08-31` [Streamlining Helm Chart Management with Argo Helm Versioner](https://www.hoelzel.it/devops/2024/08/31/argo-helm-versioner.html)
- `2023-05-08` [Demystifying etcd](https://www.hoelzel.it/kubernetes/2023/05/08/what-is-etcd.html)
- `2023-05-04` [Fixing a Kubernetes Namespace Stuck in Terminating State](https://www.hoelzel.it/kubernetes/2023/05/04/fix-stuck-namespaces.html)
- `2023-05-01` [Kubernetes Headless Services](https://www.hoelzel.it/kubernetes/2023/05/01/Headless-Services.html)
- `2023-04-25` [Embracing the Kubernetes Downward API](https://www.hoelzel.it/kubernetes/2023/04/25/Pod-info-mounted.html)

<!-- BLOG-POST-LIST:END -->

## Certifications

- **Certified Kubernetes Security Specialist (CKS) Complete Course**
  - LevelUp360° DevOps \| GCP \| Terraform \| Kubernetes \| Ansible on Udemy
  - _Kubernetes_ _Cybersecurity_

- **Azure Kubernetes Service with Azure DevOps and Terraform**
  - Kalyan Reddy Daida on Udemy
  - _Kubernetes_ _Azure_

- **AWS Fargate & ECS - Masterclass \| Microservices, Docker, CFN**
  - Kalyan Reddy Daida on Udemy
  - _AWS_

- **AWS Certified Cloud Practitioner - Complete NEW Course 2021**
  - Neal Davis on Udemy
  - _AWS_

- **AWS EKS Kubernetes-Masterclass \| DevOps, Microservices**
  - Kalyan Reddy Daida on Udemy
  - _Kubernetes_ _AWS_

- **Certified Kubernetes Administrator (CKA) with Practice Tests**
  - Mumshad Mannambeth, KodeKloud Training on Udemy
  - _Kubernetes_

- **Mastering Go Programming**
  - Packt Publishing on Udemy
  - _Golang_

## Courses

- **Configuring and Managing Kubernetes Networking, Services, and Ingress** by Anthony Nocentino
  - _Kubernetes_

- **Configuring and Managing Kubernetes Security** by Anthony Nocentino
  - _Kubernetes_

- **Creating Custom Resources in Kubernetes** by Zachary Bennett
  - _Kubernetes_

- **Deploying and Managing Azure Kubernetes Service (AKS) Clusters** By Ben Weissman, and Anthony Nocentino
  - _Kubernetes_

- **Learn terraform by setting up Highly available wordpress**
  - _Terraform_

- **Managing Advanced Kubernetes Logging and Tracing** by Piotr Gaczkowski
  - _Kuberenetes_

- **Managing Apps on Kubernetes with Istio** by Elton Stoneman
  - _Kubernetes_

- **Monitoring and Scaling Applications in Kubernetes** By Tapan Ghatalia
  - _Kubernetes_

- **gRPC [Golang] Master Class: Build Modern API & Microservices** by Stephane Maarek
  - _Golang_

- **Psychological assessment systems that measure occupational competencies, personality and interests**
  - [www.therocinstitute.com](http://www.therocinstitute.com)

- **Scrum Master**

- **Design Thinking**

## general experience
- Fullstack (Frontend, Backend, APIs, Daemons, Operators, Infrastructure-Orchestration)
- Favorites: Golang, JavaScript, PHP, C#
- Cloud: AWS, Azure, Hetzner, OVH, Contabo, Bare-Metal
- Databases: MySQL , MSSQL, PostgreSQL, TimeScaleDB, Redis, MongoDB
- DevOps: Github actions, Ansible, Puppet, Terraform, Argo CD, Rancher
- Server: Kubernetes, k3s, K3OS, Harvester, Docker (Swarm), Bare-Metal , Serverless
- Location Networking, VPN (Wireguard, Software, MS-VPN)
- Jira / Redmine / Trello

### Languages
**Deutsch**
- Native or bilingual proficiency

**Englisch**
- Native or bilingual proficiency

**Spanisch**
- Limited working proficiency

### You can reach me on LinkedIn

[![Linkedin](https://img.shields.io/badge/linkedin%20-%230077B5.svg?&style=for-the-badge&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/johannes-h%C3%B6lzel)

### Some pictures so its not so boring here ;)

![Kubernetes](https://img.shields.io/badge/kubernetes%20-%23326ce5.svg?&style=for-the-badge&logo=kubernetes&logoColor=white)![Docker](https://img.shields.io/badge/docker%20-%230db7ed.svg?&style=for-the-badge&logo=docker&logoColor=white)![AWS](https://img.shields.io/badge/AWS-%23FF9900.svg?style=for-the-adge&logo=amazon-aws&logoColor=white)![Azure](https://img.shields.io/badge/azure-%230072C6.svg?style=for-the-badge&logo=azure-devops&logoColor=white)![Cloudflare](https://img.shields.io/badge/Cloudflare-F38020?style=for-the-badge&logo=Cloudflare&logoColor=white)![Alpine Linux](https://img.shields.io/adge/Alpine_Linux-%230D597F.svg?style=for-the-badge&logo=alpine-linux&logoColor=white)![Terraform](https://img.shields.io/badge/terraform%20-%235835CC.svg?&style=for-the-badge&logo=terraform&logoColor=white)![MySQL](https://img.shields.io/badge/mysql-%2300f.svg?&style=for-the-adge&logo=mysql&logoColor=white)![Postgres](https://img.shields.io/badge/postgres-%23316192.svg?&style=for-the-badge&logo=postgresql&logoColor=white)![Ansible](https://img.shields.io/badge/ansible%20-%231A1918.svg?&style=for-the-badge&logo=ansible&logoColor=white)![Rancher](https://img.shields.io/badge/ancher%20-%230075A8.svg?&style=for-the-badge&logo=rancher&logoColor=white)![Go](https://img.shields.io/badge/go-%2300ADD8.svg?style=for-the-badge&logo=go&logoColor=white)![PHP](https://img.shields.io/badge/php-%23777BB4.svg?style=for-the-badge&logo=php&logoColor=white)![JavaScript](https://mg.shields.io/badge/javascript-%23323330.svg?style=for-the-badge&logo=javascript&logoColor=%23F7DF1E)![React](https://img.shields.io/badge/react-%2320232a.svg?style=for-the-badge&logo=react&logoColor=%2361DAFB)![Shell Script](https://img.shields.io/badge/shell_script-%23121011.svg?style=for-the-adge&logo=gnu-bash&logoColor=white)![LaTeX](https://img.shields.io/badge/latex-%23008080.svg?style=for-the-badge&logo=latex&logoColor=white)![Markdown](https://img.shields.io/badge/markdown-%23000000.svg?style=for-the-badge&logo=markdown&logoColor=white)![HTML5](https://img.shields.io/badge/html5-23E34F26.svg?style=for-the-badge&logo=html5&logoColor=white)![Bootstrap](https://img.shields.io/badge/bootstrap-%23563D7C.svg?style=for-the-badge&logo=bootstrap&logoColor=white)![CSS3](https://img.shields.io/badge/css3-%231572B6.svg?style=for-the-badge&logo=css3&logoColor=white)![Less](https://mg.shields.io/badge/less-2B4C80?style=for-the-badge&logo=less&logoColor=white)![Chart.js](https://img.shields.io/badge/chart.js-F5788D.svg?style=for-the-badge&logo=chart.js&logoColor=white)![Webpack](https://img.shields.io/badge/webpack-%238DD6F9.svg?style=for-the-badge&logo=webpack&logoColor=white)![jQuery](https://img.shields.io/badge/jquery-%230769AD.svg?style=for-the-badge&logo=jquery&logoColor=white)![Adobe Photoshop](https://img.shields.io/badge/adobephotoshop-%2331A8FF.svg?style=for-the-badge&logo=adobephotoshop&logoColor=white)![Visual Studio](https://img.shields.io/badge/isual%20Studio-5C2D91.svg?style=for-the-badge&logo=visual-studio&logoColor=white)![Visual Studio Code](https://img.shields.io/badge/Visual%20Studio%20Code-0078d7.svg?style=for-the-badge&logo=visual-studio-code&logoColor=white)![Git](https://img.shields.io/badge/git-%23F05033.svg?style=for-the-adge&logo=git&logoColor=white)![GitHub](https://img.shields.io/badge/github-%23121011.svg?style=for-the-badge&logo=github&logoColor=white)![GitLab](https://img.shields.io/badge/gitlab-%23181717.svg?style=for-the-badge&logo=gitlab&logoColor=white)![Github Actions](https://img.shields.io/badge/ithub%20actions%20-%232671E5.svg?&style=for-the-badge&logo=github%20actions&logoColor=white)![WordPress](https://img.shields.io/badge/WordPress-%23117AC9.svg?style=for-the-badge&logo=WordPress&logoColor=white)![Openwrt](https://img.shields.io/badge/OpenWrt-00B5E2?style=for-the-adge&logo=OpenWrt&logoColor=white)![Windows](https://img.shields.io/badge/Windows-0078D6?style=for-the-badge&logo=windows&logoColor=white)![Raspberry-Pi](https://img.shields.io/badge/-Raspberry%20Pi-C51A4A?style=for-the-badge&logo=Raspberry-Pi)

