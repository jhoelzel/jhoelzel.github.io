---
layout: post
title: Gaining Total Control of Your Kubernetes Nodes with  Custom Images
subtitle: Custom Images as the Backbone of Kubernetes Success
categories: kubernetes
tags: [ci, kubernetes-orchestration]
---

When managing Kubernetes clusters, ensuring that every node is secure, consistent, and optimized is crucial. We've all experienced situations where nodes behave unexpectedly due to configuration drift, outdated software, or poorly maintained base images. A powerful solution to these problems is using **custom images**—the OS-level equivalent of well-crafted container images. These images guarantee that every node you provision is identical, secure, and optimized for your workloads. In this article, we’ll dive deep into how they provide total control, enhance security, and streamline operations in Kubernetes clusters, particularly when used with RKE2.

### What Are Custom Images?

Custom images or sometimes called "golden images" are immutable, pre-configured system templates that include the operating system, necessary software, and specific configurations tailored to your environment. They serve as the foundation for every node in a Kubernetes cluster, whether it's a control plane, worker node, or specialized component like a load balancer.

They are similar to container images in that they strip away unnecessary components, leaving only what’s essential for your Kubernetes environment. These images can be versioned and reused, ensuring consistency, security, and performance across nodes and environments.

### Why could they be Essential for Kubernetes Deployments

#### 1. Consistency Across Your Cluster

One of the biggest challenges in Kubernetes environments is maintaining consistency across nodes, especially when scaling dynamically. Over time, configuration drift can cause nodes to behave unpredictably, leading to hard-to-diagnose issues.

Custom images eliminate this risk by ensuring that every node starts from an identical, pre-tested configuration. Whether you’re spinning up control plane nodes or worker nodes, each node will have the same OS version, configuration settings, and software stack. This uniformity makes troubleshooting easier and ensures that scaling operations are smooth and predictable.

The consistency provided by them also extends across environments, such as development, staging, and production. With the same image in each environment, you can be confident that any issues found in testing won’t reappear due to configuration differences in production.

#### 2. Security by Design

Public cloud images often come with extra packages, services, or outdated components that introduce security vulnerabilities. Additionally, these images are updated according to the cloud provider’s schedule, which means you might not always have the latest security patches applied when needed.

With custom images, **you control what’s included**. You can:
- Start with a minimal OS, stripping out unnecessary components to reduce the attack surface.
- **Apply security hardening** by disabling unnecessary services, enforcing secure SSH configurations, and locking down permissions.
- Ensure compliance with industry standards, such as **CIS benchmarks**, right from the start.

By maintaining and regularly updating your own images, you ensure that every node is fully patched and secure when it’s provisioned. This **immutable infrastructure** approach eliminates configuration drift and ensures that each node is in a known, secure state from the moment it joins your cluster.

#### 3. Speeding Up Node Provisioning

One of the biggest benefits of such images is the speed at which they allow you to provision new nodes. Traditional setups rely on cloud-init scripts or post-boot configuration steps, which can be slow and error-prone. If a script fails, you could end up with an incomplete node configuration, leading to instability.

With custom images, all the required configurations, binaries, and tools—such as **RKE2**, container runtimes, and monitoring agents—are baked into the image. This means nodes are ready to join your cluster as soon as they boot, without the need for lengthy initialization scripts. In dynamic environments where nodes are frequently scaled up or down, this **rapid provisioning** significantly reduces operational delays.

#### 4. Full Control Over Your Node Environment

Using cloud provider images often means you’re stuck with whatever the provider includes, such as pre-installed vendor-specific agents or unnecessary software that might not align with your Kubernetes environment. Worse, the cloud provider can update these images without notice, introducing unexpected changes to your infrastructure.

With custom images, **you’re in control**. You select the base OS, the installed software, and the kernel version. You can fine-tune system configurations and security policies to meet the specific needs of your Kubernetes workloads. For example:
- **Optimize kernel parameters** for performance in Kubernetes environments.
- **Remove unnecessary services** and packages to minimize the attack surface and improve node efficiency.
- **Customize security policies** to enforce organization-wide standards.

They also allow for **vendor independence**, ensuring that your nodes are configured identically across multiple cloud providers or on-premise environments. This flexibility is particularly important for organizations using a **multi-cloud** or **hybrid-cloud** strategy.

#### 5. Embedding Custom Scripts and Self-Written Programs

One of the biggest advantages of their is the ability to **embed your own tools, scripts, and self-written programs** directly into the image, far exceeding what public cloud images can offer.

For example:
- **Custom Go programs** can be pre-installed to automate node-specific tasks, like gathering enhanced metrics for autoscaling decisions or enforcing workload-specific policies.
- **Automation scripts** can be embedded to handle logging, monitoring, and security scanning without needing additional setup post-deployment.
- You can pre-install **custom agents** to monitor network activity or handle distributed logging, ensuring every node is configured consistently.

This flexibility allows you to create nodes that are fully prepared to handle the exact tasks and configurations required, eliminating the need for additional manual setup and providing a level of customization that public cloud images cannot match.

### Real-World Example: Financial App Deployment on Hetzner and DigitalOcean

In a real-world scenario, I worked with a financial application where custom images were implemented across infrastructure on Hetzner and DigitalOcean. The primary goal was to ensure fast, consistent node provisioning while maintaining strict security standards.

Here’s how they were implemented:
- **Compliance**: Every image was built to comply with **CIS benchmarks**, starting from a hardened base OS. This ensured that each node was secure from the start, with unnecessary services disabled and access tightly controlled.
- **Role-Specific Optimization**: Different images were created for control plane nodes, worker nodes, agents, and load balancers. Each image was optimized for its specific task, ensuring that nodes were tailored to their role in the cluster.
- **Go-Based Access Proxy**: The **Teleport Access Proxy** (written in Go) was baked into the images to provide secure, auditable access to each node. This ensured compliance with strict security and access control policies, with minimal post-deployment configuration.
- **Fast Provisioning**: Node provisioning times were reduced to minutes, even in a highly regulated environment. With everything pre-configured in the image, nodes could immediately join the cluster without needing time-consuming cloud-init scripts or additional configuration steps.

The remaining infrastructure—such as networking, firewalls, and VPCs—was managed through **Terraform**, allowing for a highly automated and consistent deployment process. This combination of custom images and Terraform greatly simplified the process of deploying compliant, secure, and scalable infrastructure.

### Avoiding Cloud Provider Limitations

Cloud provider images often come with **vendor-specific tools**, logging agents, and configurations that might not be relevant to your Kubernetes environment. Additionally, they are updated on the provider’s timeline, which can leave you with outdated packages or unanticipated changes.

You **eliminate these limitations**:
- You can build the image exactly as needed, without the bloat of vendor-specific agents or unnecessary services.
- You control when updates are applied, ensuring that your infrastructure remains secure and stable on your terms, not the provider’s.
- In **multi-cloud environments**, using custom images allows for consistent node configurations across different cloud platforms, simplifying operations and avoiding vendor lock-in.

### Automating Image Updates and Infrastructure with Terraform

Maintaining and updating images requires a streamlined process to ensure that they remain secure and up to date with the latest patches and software releases. Integrating your image-building pipeline with **CI/CD automation** and **Terraform** helps ensure that your infrastructure is always in sync with the latest configurations.

#### 1. CI/CD Integration for Automated Image Building
By integrating custom image builds into your CI/CD pipeline, you can automatically trigger new builds whenever there are security patches or software updates. A typical workflow looks like this:
- **Trigger a Build**: The CI/CD pipeline triggers an image build whenever a patch is released or a configuration change is required.
- **Automated Testing**: Once the image is built, automated tests are run to ensure it works as expected. This includes security scans, performance tests, and conformance checks.
- **Versioning and Rollout**: Each image is versioned, ensuring that changes are tracked and that you can easily roll back to a previous version if needed. Once tested, the new image is rolled out incrementally to ensure stability.

#### 2. Terraform for Infrastructure Provisioning
While custom images take care of node configuration, **Terraform** manages the surrounding infrastructure, such as networking, firewalls, and VPCs. Terraform automates the provisioning of the entire Kubernetes environment, ensuring that:
- Nodes are deployed with the correct network configurations.
- **Firewall rules** are enforced, ensuring that only necessary ports are open.
- The infrastructure is versioned and tracked, making it easy to replicate environments or roll back changes.

Using Terraform allows for a fully automated, **infrastructure-as-code** approach to managing Kubernetes deployments, from node configuration to networking and security.

### Edge Computing and Custom Images: Expanding Kubernetes Beyond the Cloud

As Kubernetes expands beyond traditional cloud environments into **edge computing**, custom images become even more valuable. Edge nodes are often resource-constrained and operate in environments with limited network connectivity. They can be optimized to handle these unique requirements by:
- Stripping down the OS and minimizing the resource footprint to improve performance on constrained hardware.
- Preloading essential components and enabling local caching to reduce reliance on network connectivity.


- Ensuring that nodes can function autonomously when disconnected from the control plane.

Specialized images designed for edge environments allow Kubernetes to be deployed in remote locations with minimal infrastructure while still ensuring consistency and security.

### Long-Term Strategy: Evolving Your Infrastructure 

Custom images are not just about solving today’s problems—they’re about future-proofing your Kubernetes infrastructure. As workloads evolve, compliance standards become stricter, and new technologies like **AI/ML** or **edge computing** take hold, they provide a flexible foundation that can adapt to new requirements without significant reengineering.

#### 1. Modular and Adaptable
Images can be designed to be modular, allowing you to build images optimized for specific workloads. For example, you might have:
- A lightweight image for resource-constrained environments (e.g., edge nodes).
- A high-performance image optimized for AI/ML workloads with pre-installed libraries like TensorFlow or PyTorch.

#### 2. Collaboration Between DevOps and Development Teams
Custom images help ensure that development, staging, and production environments are consistent. By embedding standard tools, libraries, and runtime environments directly into the images, you reduce the likelihood of “works on my machine” or "works in staging" issues. This accelerates collaboration between DevOps and development teams, enabling faster debugging and fewer production issues.

### 3. They life in GIT
These images can easiy withstand auditing procedures by using proper GitOps practises. Environments are not only clearly built with a version history but with the use of github actions, building your image becomes an easy workflow like this:

```YAML
name: Build All DigitalOcean Snapshots

on:
  workflow_dispatch:

jobs:
  build_digitalocean:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v2

      - name: Set up Packer
        uses: hashicorp/setup-packer@v1

      - name: Build DigitalOcean snapshots
        run: make build-digitalocean
        env:
          DO_TOKEN: ${{ secrets.DO_TOKEN }}
          DISCORD_WEBHOOK_URL: ${{ secrets.DISCORD_WEBHOOK_URL }}

```
## Deep Dive: Automating Custom Kubernetes Images with Packer

When managing Kubernetes clusters in any environment, using custom images ensures consistency, security, and speed during node provisioning. **HashiCorp Packer** simplifies the creation of these images, automating their build process so they can be pre-configured, secure, and Kubernetes-ready.

In this chapter, we’ll walk through using Packer to build a custom **K3s** image for **DigitalOcean**, ensuring the image is optimized, hardened, and lean. This approach minimizes the need for manual node configuration, making your infrastructure more efficient and reliable.
This is of course not an complete example, yet I think it will get the point across.

#### Packer Template for DigitalOcean

This template provisions a **K3s-ready** image on DigitalOcean using Ubuntu, with firewall settings, SSH hardening, and clean-up steps to ensure the image is secure and lightweight. Importantly, it includes a reset of **cloud-init** to ensure fresh configuration on new instances and of course if you like, even initialize the image itself with a cloud-config.

```hcl
source "digitalocean" "ubuntu-k3s" {
  image       = "ubuntu-20-04-x64"
  region      = "nyc3"
  size        = var.instance_size
  ssh_username = "root"
}

build {
  name    = "ubuntu-k3s-build"
  sources = ["source.digitalocean.ubuntu-k3s"]

  provisioner "shell" {
    inline = [
      # Update the system and install necessary tools
      "apt-get update -y",
      "apt-get upgrade -y",
      "apt-get install -y curl ufw",

      # Set up firewall rules for security
      "ufw allow OpenSSH",       # SSH access
      "ufw allow 6443/tcp",      # K3s API port
      "ufw allow 8472/udp",      # Flannel networking for K3s
      "ufw allow 10250/tcp",     # Kubelet communication
      "ufw enable",              # Enable firewall

      # Disable password-based SSH for added security
      "sed -i 's/^#PasswordAuthentication yes/PasswordAuthentication no/' /etc/ssh/sshd_config",
      "systemctl restart sshd",

      # Install K3s (lightweight Kubernetes)
      "curl -sfL https://get.k3s.io | sh -",

    ]
  }

  # Clean up to minimize image size
  provisioner "shell" {
    inline = [
      "apt-get clean",                       # Clean apt cache
      "rm -rf /var/lib/apt/lists/*",         # Remove apt lists
      "rm -rf /tmp/*",                       # Clean /tmp
      "rm -rf /var/tmp/*"                    # Clean /var/tmp
    ]
  }

  # Reset cloud-init so the image is fresh for future instances
  provisioner "shell" {
    inline = [
      # Stop cloud-init to reset its state
      "systemctl stop cloud-init",

      # Clean up cloud-init logs and state to ensure new instance gets fresh initialization
      "rm -rf /var/lib/cloud/",
      "rm -rf /var/log/cloud-init.log /var/log/cloud-init-output.log",

      # Ensure cloud-init runs on new instances
      "touch /etc/cloud/cloud-init.disabled",   # Temporarily disable cloud-init
      "rm /etc/cloud/cloud-init.disabled"       # Re-enable for the next boot
    ]
  }
}
```

### Let's dive into the details

1. **Source Configuration**:
   - The source uses **Ubuntu 20.04** on DigitalOcean, with flexibility to select the instance size via the `var.instance_size` variable.
   - The image is built in the **nyc3** region, and `root` is used as the SSH user during the image build process.

2. **Provisioning**:
   - **System Updates and Tools**: Updates the OS and installs essential tools like **curl** and **UFW** (firewall).
   - **Firewall Setup**: Configures UFW to allow only required ports:
     - **SSH (OpenSSH)**: For secure SSH access.
     - **K3s API (6443/tcp)**: To allow K3s communication.
     - **Flannel (8472/udp)**: For Kubernetes networking.
     - **Kubelet (10250/tcp)**: Ensures Kubelet communication.
   - **SSH Hardening**: Disables password-based SSH access to enforce key-based authentication, adding another layer of security.
   - **K3s Installation**: Downloads and installs **K3s**, the lightweight Kubernetes distribution, making the node ready for your cluster.
   - **Verification**: Confirms that K3s is installed and running by checking the node status.

3. **Image Cleanup**:
   - After provisioning, the template runs a cleanup process to remove unnecessary files, package lists, and temporary data. This step is essential for keeping the image lightweight and secure by reducing the potential attack surface and improving node performance.

4. **Resetting Cloud-Init**:
   - **Cloud-init** is reset to ensure that when a new instance is created from this image, it runs a fresh cloud-init cycle, pulling the correct instance-specific data like networking and metadata configurations.
   - Logs and cloud-init states are wiped clean, guaranteeing that each new instance starts without residual configuration from the original build process.

### Why This Template Works for Kubernetes

This Packer template is designed to make **DigitalOcean** Kubernetes nodes more efficient and secure. It ensures that each node is:
- **Pre-configured and consistent**: With all necessary software and security settings baked into the image, every node will behave the same way, reducing potential configuration drift.
- **Hardened for security**: The template applies some security best practices, including firewall rules and SSH hardening, minimizing potential attack vectors. You can see how we can easily go even much deeper if we wanted to.
- **Optimized for Kubernetes**: K3s is pre-installed, and the node is verified as ready to join the cluster, reducing the need for post-boot configurations.

By automating the creation of custom Kubernetes images using Packer, you gain control over how your nodes are built, ensuring they are **secure**, **consistent**, and **ready to scale**. This template demonstrates how to efficiently build K3s-ready nodes that are hardened, optimized, and require minimal manual setup. By integrating Packer into your Kubernetes workflow, you can basically do what you want and also save time, increase operational security, and improve the overall efficiency of your infrastructure.

#### What now?

With this setup you can initialize the kubernetes node by simply providing the k3s config through cloud-init in terraform. Or through the provider API. Or even through kubernetes operators ;). 

### TLDR: Custom Images as the Backbone of Kubernetes Success

In today’s fast-paced cloud-native environments, where scalability, security, and flexibility are critical, **custom images** are the backbone of successful Kubernetes deployments. By building and maintaining them tailored to your workloads, you ensure that your nodes are consistently configured, secure, and ready to meet the demands of production environments—whether in the cloud, at the edge, or on-prem.

From **fast provisioning** and **automated updates** to **resource optimization** and **enhanced resilience**, they are a powerful tool for any organization leveraging Kubernetes. When combined with automation tools like **Terraform** and integrated into a broader **DevOps** strategy and offer the control and flexibility needed to manage modern infrastructure at scale.

By adopting golden images, you are not only solving the challenges of today but also preparing your infrastructure for the future—whether that involves scaling across multiple clouds, expanding to edge environments, or adopting new workloads. They give you the foundation to build a Kubernetes environment that is predictable, secure, and adaptable to the ever-changing demands of modern cloud-native applications.
