---
layout: post
title:  Building Resilience with kube-probesim
subtitle: A Tool to Simulate Kubernetes Probes
categories: kubernetes
tags: [kubernetes-problems, cli]
---
### Building Resilience with kube-probesim: A Tool to Simulate Kubernetes Probes

As the creator of `kube-probesim`, I wanted to solve a specific problem: simulating how Kubernetes applications handle liveness and readiness probe failures. This tool was born out of a need to quickly replicate real-world failure conditions like random probe failures, latency spikes, and failing external dependencies in a controlled manner.

For many Kubernetes developers and operators, building resilient systems requires more than just passing happy path tests. You need to understand how your application behaves under stress or failure. Kubernetes' liveness and readiness probes are crucial in this respect, but testing them without good tools can be tricky. That’s where `kube-probesim` comes in. Let me explain how you can leverage it and how easy it is to deploy, thanks to its availability in the GitHub Container Registry.

---

#### Why kube-probesim?

When I first started using Kubernetes in production, one of the challenges was ensuring applications were resilient enough to withstand failures—especially during scale-ups, unexpected traffic spikes, or external service downtimes. Probes (liveness and readiness) play a crucial role in ensuring the stability of the system, but testing these probes in realistic scenarios is difficult. `kube-probesim` simplifies this.

Here’s what it does:

- **Probe Failures**: Simulate random or time-triggered liveness/readiness probe failures.
- **Latency Simulation**: Introduce configurable network latencies.
- **External Dependency Failure**: Simulate downstream service failures.
- **Network Partitioning**: Test behavior under simulated network issues.

The goal is to empower developers and operators to test probe handling under controlled, configurable failure conditions without introducing a lot of complexity.

---

#### Deploying kube-probesim from GitHub Container Registry

Since I’m hosting `kube-probesim` in the GitHub Container Registry, deploying it into any Kubernetes cluster is a breeze. No need to build the image yourself—just pull the container directly from the registry.

Here's how you can do it in your own environment:

1. **Add kube-probesim Deployment**

   Create a Kubernetes deployment YAML with the GitHub-hosted container image:

   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: kube-probesim
   spec:
     replicas: 1
     selector:
       matchLabels:
         app: kube-probesim
     template:
       metadata:
         labels:
           app: kube-probesim
       spec:
         containers:
         - name: kube-probesim
           image: ghcr.io/jhoelzel/kube-probesim:latest
           ports:
           - containerPort: 8080
           env:
           - name: FAILURE_RATE
             value: "20"
           - name: LATENCY
             value: "50"
           - name: LIVENESS_FAIL_AFTER_TIME
             value: "60"
           - name: READINESS_FAIL_AFTER_TIME
             value: "120"
   ```

   This YAML uses the official `kube-probesim` container from GitHub’s Container Registry, configuring it with a 20% failure rate and 50ms of added latency.

2. **Deploy it**

   Apply the YAML to your Kubernetes cluster:

   ```bash
   kubectl apply -f kube-probesim-deployment.yaml
   ```

   Kubernetes will pull the image directly from GitHub and start the `kube-probesim` pod with the parameters you’ve set.

---

#### Real-World Scenarios

Here’s how I’ve used `kube-probesim` in the wild:

##### 1. **Random Failures**
   To simulate a 10% failure rate across both probes, with a 100ms response delay:

   ```bash
   FAILURE_RATE=10 LATENCY=100 ./kube-probesim
   ```

   This allows me to test how Kubernetes reschedules pods when failures start happening unpredictably, helping ensure the system can handle such events without downtime.

##### 2. **Failing External Dependencies**
   Using the `/dependency` endpoint with `FAIL_DEPENDENCY=true` helped me test scenarios where downstream services were unavailable, helping me tweak timeouts and retries for critical service dependencies.

   ```bash
   FAIL_DEPENDENCY=true ./kube-probesim
   ```

---

#### Observability and Monitoring

If you're running `kube-probesim` in a production-like environment, don’t forget to integrate your monitoring and logging stack (e.g., Prometheus or Grafana). Observing probe failures over time will help you understand how Kubernetes reacts and will give you insights into system recovery times and automatic pod rescheduling.

#### TLDR: A tool to simulate probe failure

The beauty of `kube-probesim` lies in its flexibility and how easy it is to deploy thanks to its GitHub Container Registry image. Whether you need to test random failures, network issues, or external service dependencies, `kube-probesim` lets you simulate these failures in a safe, controlled environment. It’s a simple but powerful way to make your applications more resilient before they go live in production.

### Next Steps:
- **Get the image from the [GitHub Container Registry](https://github.com/jhoelzel/kube-probesim/pkgs/container/kube-probesim).**
- **Deploy `kube-probesim` in your staging environment, and start testing!**
  
Remember, production is unforgiving. `kube-probesim` gives you the edge by helping you identify and fix failure points *before* they cause issues for real users.

