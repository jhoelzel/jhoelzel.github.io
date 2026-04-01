---
layout: post
title: go_wait_for_k8s
subtitle: Ensuring Kubernetes Resource Readiness with InitContainers
categories: kubernetes
tags: [automation, cli, kubernetes-probes]
---

In Kubernetes, ensuring that dependent services are ready before your application starts can be a critical task. For instance, your application might rely on a PostgreSQL database, and you need to make sure the database is fully initialized and ready to accept connections before the app itself starts. Handling this properly often requires custom scripts or tools to manage readiness checks, which can get complex and error-prone.

This is where `go_wait_for_k8s` comes in. Designed to run as an `InitContainer` within your Kubernetes pods, it ensures that critical dependencies are ready before your main application container starts. This approach guarantees that your application only launches when its dependencies are fully operational, making your deployments more robust and reliable.

### The Problem: Ensuring Dependencies are Ready

When deploying applications that depend on other services-like a web application that requires a database or a cache-it’s crucial to ensure those services are ready before your application attempts to interact with them. If not handled properly, your application might fail to start or exhibit erratic behavior due to unavailable resources.

For example:

- **Database Dependencies**: Your application might fail to start or encounter connection issues if the database is not ready.
- **Service Dependencies**: Microservices might require other services to be available and ready before they can function correctly.

Traditionally, this has been managed with custom scripts or by building readiness checks directly into the application. However, these methods can be cumbersome and are often prone to failure.

### How `go_wait_for_k8s` Solves This Problem

`go_wait_for_k8s` is specifically designed to run as an `InitContainer` in your Kubernetes pod, ensuring that the necessary dependencies are ready before your main application container starts. It interacts with the Kubernetes API to check the readiness of specific resources like pods, deployments, or services.

#### Why Use an InitContainer?

Using an `InitContainer` ensures that `go_wait_for_k8s` runs before any other containers in the pod. The `InitContainer` blocks the startup of the main application container until the specified conditions are met, guaranteeing that your application only starts when its dependencies are available.

### Example Use Case: Waiting for a PostgreSQL Database

Let’s consider a common scenario where your application relies on a PostgreSQL database. You want to ensure that the PostgreSQL deployment is fully rolled out and the pod is ready before your application starts. Here’s how you can use `go_wait_for_k8s` to handle this scenario.

#### Step 1: Define the `InitContainer` in Your Pod Spec

In your application’s Kubernetes deployment, you would define an `InitContainer` that runs `go_wait_for_k8s` to wait for the PostgreSQL deployment to be ready.

Here’s an example deployment YAML:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
  namespace: app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-app
        image: my-app-image:latest
        ports:
        - containerPort: 8080
      initContainers:
      - name: wait-for-postgres
        image: ghcr.io/jhoelzel/go_wait_for_k8s:latest
        args:
        - "--namespace=app"
        - "--resource=deployment"
        - "--name=postgres"
        - "--condition=available"
```

In this YAML:

- The `InitContainer` named `wait-for-postgres` uses the `go_wait_for_k8s` image.
- It waits for the PostgreSQL deployment in the `app` namespace to reach the `available` condition, meaning all pods in the deployment are ready.
- Only after this check passes will the main application container (`my-app`) start.

#### Step 2: Deploy to Kubernetes

You can deploy this configuration to your Kubernetes cluster using `kubectl`:

```bash
kubectl apply -f my-app-deployment.yaml
```

With this setup, the `InitContainer` ensures that your application does not start until the PostgreSQL deployment is fully ready, preventing issues related to premature starts.

### Using `go_wait_for_k8s` for Other Dependencies

While the PostgreSQL example is common, `go_wait_for_k8s` can be used to wait for various types of resources. Here are a few examples:

- **Waiting for a Redis Pod**: Ensure that a Redis pod is ready before starting your caching service.

  ```yaml
  initContainers:
  - name: wait-for-redis
    image: ghcr.io/jhoelzel/go_wait_for_k8s:latest
    args:
    - "--namespace=cache"
    - "--resource=pod"
    - "--name=redis-pod"
    - "--condition=ready"
  ```

- **Waiting for an API Service**: Ensure that a critical API service is available before your microservice starts.

  ```yaml
  initContainers:
  - name: wait-for-api
    image: ghcr.io/jhoelzel/go_wait_for_k8s:latest
    args:
    - "--namespace=services"
    - "--resource=deployment"
    - "--name=api-service"
    - "--condition=available"
  ```

### Security and RBAC Considerations

Running `go_wait_for_k8s` as an `InitContainer` also integrates well with Kubernetes' RBAC (Role-Based Access Control) system. By assigning a service account with minimal permissions to the `InitContainer`, you can ensure that it only has access to the resources it needs to query.

Here’s an example of how you might configure RBAC for `go_wait_for_k8s`:

#### Create a Role with Minimal Permissions

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: app
  name: go-wait-for-k8s-role
rules:
- apiGroups: [""]
  resources: ["pods", "deployments"]
  verbs: ["get", "list", "watch"]
```

#### Bind the Role to a Service Account

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: go-wait-for-k8s-binding
  namespace: app
subjects:
- kind: ServiceAccount
  name: go-wait-for-k8s-sa
  namespace: app
roleRef:
  kind: Role
  name: go-wait-for-k8s-role
  apiGroup: rbac.authorization.k8s.io
```

Assign this service account to the `InitContainer`:

```yaml
spec:
  initContainers:
  - name: wait-for-postgres
    image: ghcr.io/jhoelzel/go_wait_for_k8s:latest
    args:
    - "--namespace=app"
    - "--resource=deployment"
    - "--name=postgres"
    - "--condition=available"
    serviceAccountName: go-wait-for-k8s-sa
```

This setup ensures that `go_wait_for_k8s` can only access the necessary Kubernetes resources, adhering to the principle of least privilege.

### TLDR: a simple init container to wati for your resources

`go_wait_for_k8s` is a simple yet powerful tool designed to enhance the reliability of your Kubernetes deployments. By running as an `InitContainer`, it ensures that critical dependencies are ready before your application starts, reducing the risk of failed starts and other issues related to unavailable resources.

Whether you’re managing databases, services, or any other type of Kubernetes resource, `go_wait_for_k8s` provides a streamlined way to handle readiness checks natively within your Kubernetes environment. With minimal setup, you can make your applications more resilient and your deployments more predictable.

For more information, to see the source code, or to contribute, visit the [GitHub repository](https://github.com/jhoelzel/go_wait_for_k8s). As always, feedback and contributions are welcome. Let’s keep making Kubernetes a more robust platform, one InitContainer at a time.
