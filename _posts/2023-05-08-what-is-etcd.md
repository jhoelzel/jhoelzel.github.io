---
layout: post
title: Demystifying etcd
subtitle: And why it is the Backbone of Distributed Systems
categories: kubernetes
tags: [kubernetes, kubernetes-state, etcd]
---

By now, you are no strangers to the challenges of managing distributed systems. Enter etcd(`pronounced /ˈɛtsiːdiː/,`), the unsung hero that quietly toils behind the scenes to keep these systems in check. **etcd is an open source distributed key-value store tailor-made to hold and administer the crucial information that distributed systems**, such as Kubernetes, need to function smoothly.

At the heart of distributed systems lie multiple machines working in harmony to achieve a shared objective. Kubernetes utilizes etcd as its reliable data store, providing a consistent view of the system's status, including clusters, pods, and running application instances etc. If etcd were to pen an autobiography, it might be titled, **"From Linux Directory to Distributed Data Store: My Journey."** Because the name "etcd" originates from a naming convention within the Linux directory structure. In UNIX, all system configuration files for a single system are contained in a folder called "/etc," and the "d" in "etcd" stands for "distributed."

But *why etcd*, you ask? 

This versatile tool is:

- Fully replicated: Every node has access to the _entire data store_, ensuring data availability and redundancy.
- Highly available: With no single point of failure, etcd _can gracefully handle hardware failures_ and network partitions.
- Reliably consistent: Every data read returns the most recent write, avoiding conflicts or discrepancies.
- Fast: etcd boasts a benchmark of 10,000 writes per second.
- Secure: Automatic TLS, optional SSL client certificate authentication, and role-based access controls protect sensitive data.
- Simple: Applications can interact with etcd using standard HTTP/JSON tools.

One might say that etcd is like a high-performance engine and it works best with fast storage disks (SSDs are highly recommended).

## Built on the Raft consensus algorithm

Etcd ensures data store consistency across all nodes in a cluster, maintaining highly available and consistently replicated copies of the data store. Raft elects a leader node that manages replication for the followers, creating a harmonious and efficient system.

The leader accepts requests from clients and forwards them to follower nodes. **The leader verifies that a majority of follower nodes have stored each new request and only then applies the entry to its local state machine**. If this is not the case, the leader will retry until all followers have stored all log entries consistently. This helps the system using it by providing an equal query surface to all processes that seek the information, not matter which node they query. Furthermore if a follower node fails to receive a message from the leader within a specified time interval, an election is held to choose a new leader automatically. It declares itself a candidate, and the other followers vote for it or any other one based on their availability.

In the worst case scenario, where the majority of etcd nodes fail, the cluster will not accept any more writes to it and will maintain its state. The cluster can only recover from a majority failure once the majority of members become available and reach concensious. If they can not, then the operator must start disaster recovery to recover the cluster by themselves.

In the realm of Kubernetes, etcd plays a vital role as the primary key-value store for cluster state data. Kubernetes leverages etcd's "watch" function to monitor data and reconfigure itself when changes arise. This allows Kubernetes to maintain an ideal state and quickly respond to any discrepancies. _The "watch" function stores values representing the actual and ideal state of the cluster_ and can initiate a response when they diverge. This is the little magic, that lies in every kubernetes cluster and what makes them work in the first place.

## Etcd VS Redis

Now, let's address the elephant in the room: etcd vs. Redis. While both are open source tools, their primary functions differ significantly. Redis is an in-memory data store that excels in speed, but etcd trades off speed for greater reliability and guaranteed consistency, making it the go-to choice for storing and managing distributed system configuration data. Redis supports a wider variety of data types and structures than etcd and has much faster read/write performance. However, etcd has superior fault tolerance, stronger failover, and continuous data availability capabilities. _Most importantly, etcd persists all stored data to disk_, trading off speed for greater reliability and guaranteed consistency. 

## Etcd VS SQL

And what about etcd vs. MySQL and PostgreSQL? While all three are data storage solutions, etcd is a distributed key-value store specifically designed for distributed systems, while MySQL and PostgreSQL are traditional relational databases adept at handling structured data with relationships. Choosing the right data storage solution is like selecting the perfect pair of shoes: it depends on the occasion, or in this case, the specific requirements of your system.

In a nutshell, etcd is a steadfast sentinel keeping distributed systems, like Kubernetes, in harmony. Its design prioritizes fully replicated, highly available, reliable, fast, secure, and simple data storage. With its superior fault tolerance and data persistence capabilities, etcd is the ideal candidate for storing and managing distributed system configuration information, standing steadfast in its duty to maintain the state of distributed systems. 

Traditional databases like MySQL and PostgreSQL prioritize consistency and durability, but their fault tolerance capabilities depend on the specific setup and configuration. While etcd is ideal for managing critical information in distributed systems, MySQL and PostgreSQL are versatile relational databases that handle structured data with relationships. 

> NOTE: Of course you can also store cluster state in a relational database, (K3S)[k3s.io] will happily do that for you, but once you actually get it up and running your MySQL setup will literally be overtaken by state queries. I have tried on many occasions to make it work, and an effective system WILL require absolute finetuning from your end.

## Etcd's Ecosystem and Extensibility

etcd's importance extends beyond Kubernetes, as it can also be utilized to coordinate critical system and metadata across clusters of various distributed applications. Its simplicity and consistency make it an attractive choice for any distributed system that requires a reliable data store.

Its API enables seamless integration with a myriad of applications, and developers can easily interact with it using popular programming languages like Go, Python, Java, and more. Moreover, etcd's extensible architecture allows it to act as a foundation for building custom distributed systems tailored to specific use cases and requirements.

> NOTE: While you definitly can, its not recommended to modify the state of a kubernetes cluster directly, whitout using the API.

The etcd community continues to grow and evolve, contributing to the development and enhancement of it. A plethora of third-party tools, libraries, and frameworks have emerged to simplify the deployment, management, and monitoring of etcd clusters. Some notable tools in the etcd ecosystem include:

*[etcdctl](https://github.com/etcd-io/etcd/tree/main/etcdctl)*: A command-line client for managing etcd.
*[etcd-operator](https://github.com/openshift/cluster-etcd-operator)*: A Kubernetes operator that automates etcd cluster management tasks.
*[etcdadm](https://github.com/kubernetes-sigs/etcdadm)*: a command-line tool for operating an etcd cluster. It makes it easy to create a new cluster, add a member to, or remove a member from an existing cluster. Its user experience is inspired by kubeadm.
*[etcd-backup-operator](https://github.com/giantswarm/etcd-backup-operator)* takes backups of ETCD instances on both the control plane and tenant clusters.
*[ETCD Manager](https://github.com/gtamas/etcdmanager)* Provide an efficient, modern GUI for desktop

A more extensive list of tools and libraries can be found in the (etcd docs itself)[https://etcd.io/docs/v3.5/integrations/]

## Things to consider when Deploying and Managing etcd

As custodians of complex distributed systems, senior engineers understand the importance of deploying and managing etcd with care. Adhering to best practices is essential to ensure the health and performance of an etcd cluster. Some key recommendations include:

**Hardware considerations**: Use SSDs for storage, as etcd's performance relies heavily on disk speed. Provide adequate memory and CPU resources to accommodate the cluster's workloads.
**Configuration**: Tune etcd's configuration parameters to optimize performance for your specific use case, considering factors like network latency and data size.
**Monitoring and alerting**: Implement monitoring solutions, such as Prometheus and Grafana, to track etcd performance metrics and set up alerts for potential issues.
**Backup and recovery**: Regularly create and test backups of your etcd data to ensure swift recovery in the event of data loss or corruption.
**Security**: Implement role-based access controls, TLS, and SSL client certificate authentication to protect sensitive configuration data and restrict access to authorized personnel only.

## In Conclusion

etcd is a robust and reliable distributed key-value store that quietly underpins distributed systems like Kubernetes. Its fully replicated, highly available, reliable, fast, secure, and simple design ensures that it remains a steadfast guardian of critical information in distributed systems. With its unique advantages over Redis, MySQL, and PostgreSQL in the realm of distributed system configuration management, etcd has earned its place as an essential tool for senior engineers.

As the etcd ecosystem continues to flourish, and best practices for its deployment and management are widely adopted, the future looks promising for this unsung hero. With etcd, distributed systems can thrive and achieve their full potential, all while maintaining harmony and consistency in the face of ever-growing complexity.

### Sources
- (Etcd docs)[https://etcd.io/docs/v3.5/]
- (Developer Guide)[https://etcd.io/docs/v3.5/dev-guide/]
- (Operation Guide)[https://etcd.io/docs/v3.5/op-guide/]