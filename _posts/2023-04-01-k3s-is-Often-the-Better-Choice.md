# Comparing k3s with Kubernetes: How k3s is Often the Better Choice

As a startup, you may be considering the use of container orchestration for your company. Container orchestration allows you to manage and deploy containerized applications easily and efficiently. There are several options available, but two of the most popular choices are **k3s** and **Kubernetes**. In this post, we will compare these two options and discuss why k3s may often be the better choice for smaller setups.

**TLDR**
> Kubernetes and k3s are both container orchestration platforms that can help businesses manage their infrastructure. Kubernetes is more feature-rich and well-established, but it requires more resources and has a more complex installation and update process. K3s is lightweight and easy-to-use, with a smaller resource footprint and simpler installation and update process. K3s is particularly well-suited for starters and smaller teams with limited resources, while Kubernetes may be better for larger organizations with more complex needs and on-call staff.

## Introduction

Before diving into the comparison, let's first define k3s and Kubernetes and discuss the role of the Cloud Native Computing Foundation (CNCF) in their development and maintenance.

**Kubernetes**, also known as K8s, is an open-source container orchestration system originally developed by Google. It allows you to deploy and manage containerized applications at scale. Kubernetes is maintained by the Cloud Native Computing Foundation (CNCF), a member-driven organization dedicated to advancing the development of cloud native technologies. You can find more information about Kubernetes on its [official website](https://kubernetes.io/).

**k3s** is a lightweight Kubernetes distribution developed by Rancher Labs. It was designed to be easy to install, run, and maintain on any infrastructure, including bare-metal servers, virtual machines, and the cloud. In September 2020, k3s became a Cloud Native Computing Foundation (CNCF) project, which means it must pass the same software conformance tests that other CNCF-certified distributions for Kubernetes must pass. This ensures that configurations built for Kubernetes will work with k3s. You can find more information about k3s on its [official website](https://k3s.io/).


# Managed Kubernetes Clusters in the Cloud are also distributions

Managed Kubernetes in the cloud such as Amazon Web Services (AWS), Google Cloud Platform (GCP), and Microsoft Azure. These services take care of the operational aspects of deploying and managing a Kubernetes cluster, such as provisioning and scaling resources, monitoring and logging, and security and compliance. This allows organizations to focus on developing and deploying their applications, rather than worrying about the underlying infrastructure.

Nevertheless, managed Kubernetes in the cloud is not the same technology as vanilla Kubernetes, but rather an abstraction on top of it. The service providers use the same Kubernetes codebase and APIs that are used for deploying and managing a cluster on-premises or in a self-hosted environment, but fine tuned to their respective infrastructures. This means that applications developed for a managed Kubernetes in the cloud environment can be easily ported to a different environment, such as a self-hosted cluster or a different cloud provider. There are also managed k3s services offered by cloud providers, such as Rancher's k3s cloud, which provide a managed and simplified solution for deploying and running k3s in the cloud. 

Just as k3s is a distribution you can install yourself, managed clouds have their own adaptation of the original codebase. One of the key differences in k3s however is, that its distribution is completely open source and therefore can be forked and modified at will.

## Key differences and similarities between k3s and Kubernetes

Now that we have a basic understanding of k3s and Kubernetes, let's take a closer look at the key differences and similarities between these two options.

### Architecture and components

Both k3s and Kubernetes follow a similar architecture, consisting of a master node and worker nodes. The master node is responsible for maintaining the overall state of the cluster and scheduling tasks to the worker nodes. The worker nodes are responsible for running the containerized applications.

Kubernetes has a more complex architecture than k3s, with additional components such as etcd (a distributed key-value store) and the Kubernetes API server. k3s includes all of the necessary components by default, making it easier to install and manage.

K3s is a lightweight and simplified distribution of Kubernetes that comes with several components pre-integrated. Some of these components include:

- **etcd**: K3s includes a lightweight and embedded etcd instance, which is used for storing configuration data and maintaining the state of the cluster. Some alternative [storage backends for etcd in k3s](https://docs.k3s.io/installation/datastore) are:
    - MySQL
    - PostgreSQL
    - Embedded SQLite
    - Embedded etcd for High Availability

- **Container Runtime**: K3s comes with containerd, a widely adopted and lightweight container runtime, pre-installed. This eliminates the need to install and configure a separate container runtime, such as Docker, in Kubernetes.

- **Ingress Controller**: K3s includes Traefik, a popular ingress controller, pre-installed. This eliminates the need to install and configure an ingress controller, such as NGINX or Istio, in Kubernetes.

- **Service Load Balancer**: K3s includes a built-in service load balancer that can automatically route traffic to services within the cluster. In Kubernetes, a separate load balancer, such as HAProxy or MetalLB, must be installed and configured.

- **Cluster add-ons**: K3s comes with several add-ons pre-installed, such as metrics-server, coredns and local-path-provisioner. These add-ons are not included in a vanilla Kubernetes installation and have to be installed and configured separately.

By including these components pre-installed, K3s aims to simplify the deployment and management of Kubernetes clusters, making it more accessible for small and edge deployments. However, it's important to note that the pre-integrated components in K3s may not have the same level of flexibility and customization as the separate components in Kubernetes.


### Performance and scalability

Both k3s and Kubernetes are designed to be scalable and performant. However, k3s is optimized to run on low-resource environments, such as edge devices and IoT. It has a smaller resource footprint and requires fewer resources to run, making it an ideal choice for small and medium businesses with limited resources.

The following are the minimum CPU and memory [requirements for nodes in a high-availability K3s server](https://docs.k3s.io/installation/requirements):
| Deployment Size | Nodes | VCPUS | RAM |
| --- | --- | --- |
| Small            | Up to 10 | 2 | 4 GB |
| Medium           | Up to 100 | 4 | 8 GB |
| Large            | Up to 250 | 8 | 16 GB |
| X-Large          | Up to 500 | 16 | 32 GB |
| XX-Large         | 500+ | 32 | 64 GB |


### Security features

Both k3s and Kubernetes offer a range of security features, including role-based access control (RBAC), secure communication between components, and support for TLS. k3s includes these security features by default, making it easier to set up and secure a cluster. Additional features can be added through the use of third-party tools or plugins.


## k3s for smaller teams

Now that we've looked at the key differences and similarities between k3s and Kubernetes, let's take a closer look at why k3s may be the better choice for startups.

### Flexibility and ease of use

One of the main benefits of k3s is its flexibility and ease of use. k3s can be deployed on any infrastructure, including bare-metal servers, virtual machines, and the cloud. It is also optimized to run on ARM-based devices, making it ideal for edge computing scenarios.

In addition, k3s includes a single-binary deployment option, making it easier for system administrators to install and manage. The update process is also simpler, as it can be done with a single binary. This is especially beneficial for system administrators who are new to Kubernetes and may not be familiar with the more complex installation and update process of Kubernetes.

### Out-of-the-box single-node deployment

Another advantage of k3s is its out-of-the-box single-node deployment option. This allows you to quickly and easily set up a production-ready Kubernetes cluster with minimal setup and configuration. This is especially useful for teams that may not have the resources or expertise to set up and manage a multi-node Kubernetes cluster. The process of simply running the binary and having your cluster start up is simple and effective and can easily be used in ci pipelines and more. This is one of the mayor strong points of k3s, as you even can deploy a k3s cluster inside of a k3s easily.

### Integration and activation of features by default

k3s also includes a range of features that are integrated and activated by default. This includes support for load balancing, ingress controllers, and more. This makes it easier to set up and manage a cluster, as you don't have to manually install and configure these features. The flexibility exists to activate and deactivate features at will and of course also to use only third party tools for important parts like networking.

### Auto-update capability

Another advantage of k3s is its auto-update capability. This allows you to easily update your cluster to the latest version without the need to manually update each component. This is especially useful for smaller setups that may not have the resources or expertise to manage updates manually.

## Comparison of k3s and Kubernetes in practice

So far, we've looked at the key differences and similarities between k3s and Kubernetes, as well as the benefits of k3s. But how do these options compare in practice?

Here is a summary of the key differences between k3s and Kubernetes:

| Feature   | K3s... | Kubernetes...  |
| --- | --- | --- |
| Lightweight | Yes, easy to install and half the memory compared to Kubernetes  | No, Kubernetes can require more resources 
| Binary size | Less than 100 MB | multiple binaries in total larger
| Resource requirements | Low | Medium |
| Packaged as | Single binary | Multiple binaries
| Great for | Edge, IoT, CI, Development, ARM, Embedding K8s, Situations where a PhD in K8s clusterology is infeasible, but also Large-scale production environments | Large-scale production environments, running on various cloud providers, and multiple platforms
| Storage backend | sqlite3 as the default storage mechanism, etcd3, MySQL, Postgres are also available | etcd by default, other storage options available
| Complexity | Wrapped in simple launcher that handles a lot of the complexity of TLS and options | Can be more complex to set up and manage 
| Secure by default | Yes, with reasonable defaults for lightweight environments | Yes, but additional configuration may be required 
| Additional features | local storage provider, service load balancer, Helm controller, Traefik ingress controller AND third-party tools or plugins | Additional features can be added through the use of third-party tools or plugins
| External dependencies | Minimized (just a modern kernel and cgroup mounts needed) | Can require additional dependencies depending on the set up and configuration.
| Scalability | Good for small to medium scale deployments | Highly scalable and can handle large clusters 
| Customizability | Limited customization options | High degree of customizability through Kubernetes API and third-party tools
| Support | Community-driven support | Wide range of support options including official Kubernetes support, vendor support, and community support
| Maintenance | Minimal maintenance required | More maintenance required to keep the cluster healthy and running smoothly
| Cloud-agnostic | Yes | Yes
| Dual-Stack support (IPv4/Ipv6) | Yes | Yes
| Service discovery and load balancing | Yes | Yes
| Self-healing | Yes | Yes
| GitOps support | Yes | Yes
| CI/CD integration | Yes | Yes
| Automated Deployments | Yes | Yes
| Open-source | Yes | Yes
| SSO-Integration | Yes | Yes
| CLI and API for management | kubectl | kubectl
| Kubernetes ecosystem compatibility | Compatible with a great subset of Kubernetes ecosystem | Compatible with full Kubernetes ecosystem
| Vendor lock-in | None | None |
| Licensing | Apache 2.0 open-source license | Apache 2.0 open-source license |
| Production readiness | ready | ready |



## Community support and resources

Both k3s and Kubernetes have a wealth of documentation and tutorials available online. However, Kubernetes has a larger and more active community compared to k3s. BUT both projects are backed by the Cloud Native Computing Foundation (CNCF), a member-driven organization dedicated to advancing the development of cloud native technologies. The CNCF hosts a range of events, meetups, and other community activities, as well as a large number of resources and documentation. As k3s and kubernetes are feature compatible, almost all tutorials apply in the same way for k3s as they do for kubernetes.

### Considerations for long-term stability and maintenance in small businesses

Small businesses may want to consider the long-term stability and maintenance of k3s and Kubernetes when deciding which option is right for them. Rancher labs has officially been acquired by [Suse](https://www.suse.com/de-de/products/rancher-kubernetes-engine/) in 2020 and therefore continuation of this project should be considered safe.

## Conclusion

In conclusion, k3s and Kubernetes are both powerful container orchestration platforms that can helpbusinesses improve the efficiency and reliability of their infrastructure. Kubernetes is a more feature-rich and well-established platform with a larger and more active community. It is generally better suited for larger organizations with more complex needs and on-demand platform engineers, because it does not come with included batteries and therefore introduces all of the complexities associated with it.

On the other hand, k3s is a lightweight and easy-to-use platform that is particularly well-suited for smaller teams with limited resources. It has a smaller resource footprint and requires fewer resources to run, making it an ideal choice. The single-binary deployment, makes k3s especially appealing for linux veterans, as the systemd based setup of the binary, which simply restarts on failure, also provides a simple entry into a deep dive of kubernetes. It can run on a variety of platforms, including ARM and x86 processors, and in a variety of environments, including on-premises, in the cloud, and at the edge. k3s is designed to reduce the complexity of deployment, operation, and maintenance of Kubernetes clusters.

If you enjoyed this article and would like to know more how k3s could be deployed in your company, make sure to reach out to me through LinkedIn.



## Sources

- [k3s documentation](https://k3s.io/docs/)
- [Kubernetes documentation](https://kubernetes.io/docs/)
- [Rancher Labs](https://rancher.com/)
- [Packet](https://www.packet.com/)
- [Cloud Native Computing Foundation (CNCF)](https://www.cncf.io/)
- [Suse](https://www.suse.com/de-de/products/rancher-kubernetes-engine/)
- [storage backends for etcd in k3s](https://docs.k3s.io/installation/datastore)
- [requirements for nodes in a high-availability K3s server](https://docs.k3s.io/installation/requirements)
