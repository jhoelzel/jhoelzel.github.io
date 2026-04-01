---
layout: post
title: Streamlining Helm Chart Management with Argo Helm Versioner
subtitle: Automating Helm version checks in Argo CD environments
categories: devops
tags: [argo, helm, kubernetes, ci/cd]
---
For anyone managing Kubernetes environments with Argo CD, keeping Helm charts up-to-date is a routine but essential task. Argo Helm Versioner is a straightforward tool that helps automate this process, ensuring your deployments stay current without adding extra overhead.

### Why Helm Chart Versioning Can Be a Headache

If you've ever found yourself manually checking which versions of Helm charts are running in your environment, you know how tedious and error-prone it can be. Argo Helm Versioner takes this off your plate by automatically scanning your project directories, identifying Argo CD applications that use Helm charts, and comparing the deployed versions with the latest available versions in the Helm repositories.

### Streamlining Your DevOps Workflow with Argo Helm Versioner

Argo Helm Versioner integrates seamlessly into your existing workflows. Whether you're running it as part of your CI/CD pipeline, performing regular checks, or just doing a one-off audit, this tool simplifies the process:

- **Deep Directory Scanning**: No more manual hunting. The tool finds every relevant YAML file, even if it’s buried deep in your directory structure.
- **Accurate Version Comparison**: By leveraging semantic versioning, Argo Helm Versioner ensures that even minor differences between versions are detected.
- **Clear Output**: The tool presents results in an easy-to-read table, showing you at a glance which applications need an update.

### Real-World Example: Using Argo Helm Versioner with Popular Helm Charts

Let’s explore how this might look with some widely-used Helm charts. Suppose you’re managing a Kubernetes environment that includes several popular services:

- **Nginx Ingress Controller**: A crucial component for managing external access to your services.
- **Prometheus**: The go-to solution for monitoring and alerting.
- **Grafana**: Paired with Prometheus for powerful data visualization.
- **Redis**: Often used as a caching layer or message broker.
- **ElasticSearch**: Commonly deployed for search and analytics.

Here's what the output might look like after running Argo Helm Versioner:

```
Application             FilePath                                    Current Version   Latest Version   Status
nginx-ingress           /apps/nginx-ingress/argo-app.yaml           4.0.6             4.3.0            Update available
prometheus              /apps/prometheus/argo-app.yaml              14.8.0            15.4.0           Update available
grafana                 /apps/grafana/argo-app.yaml                 6.17.4            7.2.0            Update available
redis                   /apps/redis/argo-app.yaml                   15.3.1            15.4.0           Up-to-date
elasticsearch           /apps/elasticsearch/argo-app.yaml           7.10.1            7.12.1           Update available
```

In this scenario:

- The `nginx-ingress`, `prometheus`, `grafana`, and `elasticsearch` services all have newer versions available, prompting you to review and potentially update these applications.
- The `redis` service is up-to-date, giving you peace of mind that no action is needed there.

This output gives you a clear, concise overview of the status of your Helm charts, enabling you to quickly address any outdated services.

### Customizing and Extending Argo Helm Versioner

While Argo Helm Versioner is designed to be simple, it’s also flexible. You can easily integrate it with other tools in your pipeline or extend its functionality to suit your specific needs. Whether you want to automate updates, trigger notifications, or just keep a clean report of your Helm chart versions, this tool has you covered.

### Practical Use Cases: A Handy Tool for Daily Operations

Imagine you're managing a microservices architecture with several teams. Argo Helm Versioner can help ensure that all your services are using the most current and secure versions of Helm charts. It's not about solving world problems-it's about making your life a bit easier by automating a routine task that, left unchecked, could lead to issues down the road.

### Conclusion: A Simple Tool for a Simple Task

Argo Helm Versioner is exactly what it sounds like: a tool to help you keep your Helm chart versions in check. It’s not trying to be more than it is. It’s a practical, no-frills solution for those who want to automate version checks and avoid the headache of manual updates. If you’re using Helm and Argo CD, it’s worth a look. For more information and to get started, check out the [Argo Helm Versioner GitHub repository](https://github.com/jhoelzel/argo-helm-versioner/tree/main). Contributions and feedback are welcome as the tool continues to evolve based on real-world needs.
