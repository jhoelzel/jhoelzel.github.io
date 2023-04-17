---
layout: post
title: Taming Traffic Spikes in Kubernetes 
subtitle: The Power of Resource Limits and Overprovisioning Workloads
categories: kubernetes
tags: [kubernetes, autoscaling, hpa, automation, overprovisioning]
---

## In autoscaling you have to be prepared! The Consequences of Ignoring or setting Limits

In Kubernetes, setting resource limits can help you manage your cluster's resources more effectively. However, whether you set resource limits or not, there will always be a chance that some users are left out due to resource constraints or workload spikes. To ensure the best user experience and maintain high availability, it is crucial to design buffered workloads and configure your autoscalers to compensate for fluctuations in demand.

In this article, we will explore the importance of overprovisioning workloads and how to set up autoscalers (mainly the [Horizontal Pod Autoscaler](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/)) in Kubernetes to optimize resource utilization and provide a seamless user experience, regardless of whether resource limits are set or not. Through this balanced approach, you can achieve a more robust and resilient application infrastructure that meets the needs of your users even in the face of fluctuating demand. It also asumes that you are using the [cluster-autoscaler](https://github.com/kubernetes/autoscaler) to dynamically scale your cluster nodes.

***TLDR***
> By implementing buffered workloads, you can create a cushion that allows your application to handle sudden increases in traffic or resource usage more gracefully. This approach helps prevent users from experiencing degraded performance or service outages during peak times. Combined with well-configured autoscalers, this strategy ensures that your cluster can dynamically adjust to changes in demand, scaling up or down as needed to accommodate user requests.


## The scenario

Consider an 8-core nodesize cluster where Service A is deployed with CPU requests and no limits, and the HPA is set up accordingly, using a target metric value like average CPU utilization.

Our workload consists of a deployment that we would like to autoscale, but also not overprovision our cluster. We start at one replica and would potentially provison up to 100 of them, if there is a lot of load on our systems.
Of course there are other parts omitted, because we should have a 12 factor application and there are usually also databases involved, but please bear with me for simplicity.

This is our workload: 

```YAML
# the deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: service-a
spec:
  replicas: 1
  selector:
    matchLabels:
      app: service-a
  template:
    metadata:
      labels:
        app: service-a
    spec:
      containers:
      - name: service-a-container
        image: <your-image>
        resources:
          requests:
            cpu: 200m
---
apiVersion: autoscaling/v2beta2
kind: HorizontalPodAutoscaler
metadata:
  name: service-a-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: service-a
  minReplicas: 1
  maxReplicas: 100
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50

```

To start with, we have a CPU request but no limit in place: CPU requests/limits are set to 200m / none.
200m is usually enough to encorporate the average usage of the our super huge startup that means round about 20 concurrent users so we figured this is a great way to start off.

### oh no, here comes a traffic spike

In our hypothetical situation, a sudden surge in requests overwhelms the service, causing it to consume all available CPU resources on the node.

Kubernetes' HPA operates as a control loop that runs intermittently, with an interval set by the **--horizontal-pod-autoscaler-sync-period** parameter to the kube-controller-manager (the default interval is 15 seconds and I bet you did not change it). The controller manager queries the resource utilization against the metrics specified in each HPA definition during each interval. It then selects the Pods based on the target resource's **.spec.selector** labels and obtains the metrics from either the resource metrics API (for per-Pod resource metrics) or the custom metrics API (for all other metrics). 

We start out in a green field and have just deployed our application and Marketing is beating the drums to send our message out to the world.
Therefore if there is a sharp increase in traffic, for instance your new and shiny app being shared on discord or reaching the front page of hackernews our scenarios might play out like this:

## Having no limits

The sudden massive influx of request hits the cluster and our pods start fighting for resources and critical services such as DNS, kube-proxy, and monitoring tools are competing for their share of CPU too. If your cluster is configured correctly, they will have priority over your workloads because they have QOS annotations, and therfore also steal some of the leftover resources your app could have consumed. We have provisioned our cluster with a little bit of headroom that we wanted to leave for unexpected surges like this, but as we have seen with out request limit, we figured that 6 more cores should be plenty for now. 

For per-Pod resource metrics like in our case, CPU, the controller calculates the utilization value as a **percentage of the equivalent resource request on the containers** in each Pod if a target utilization value is set. If a target raw value is set, the raw metric values are used directly. The controller takes the mean of the utilization or raw value (depending on the target type) across all targeted Pods in a deployment and produces a ratio used to scale the number of desired replicas.
Consequently, the HPA scales the service based on the actual CPU usage and the target utilization. 

Suppose the target CPU utilization is set at 50%, and the actual CPU usage is 8000m (8 cores), or all available cpu on the node, simply because as I mentioned a tremendous spike can happen in 15 seconds with social media involved. In this case, the HPA calculates the desired number of replicas using the formula: desiredReplicas = ceil[currentReplicas * (currentUtilization / targetUtilization)]. If we assume there's currently 1 pod running with 200m CPU request, the HPA calculates ceil[1 * (8000m / (1 * 200m * 50%))] = 80 and thus will try to scale to *80 replicas*. Now of course this is an over the top example, yet it is possible, especially with requests where the CPU usage is not linear with request usage. If you have not provisioned any buffer-nodes, kubernetes will now add 4 new servers/nodes for you. As our nodesize is set to 8 cores and we also need room for monitoring, logging, the kubelet and possibly more on each node.

Even if we do not go directly to the extreme and resource usage rises only to 4000m or 4 cores the will still mean ceil[1 * (4000m / (1 * 200m * 50%))] = 40 Pods! Or as we have 8 core nodes, 2 new Servers.

If there are not enough sufficient nodes available, the Cluster Autoscaler comes into play. It ensures that there are enough nodes to accommodate the desired number of replicas by adding or removing nodes from the cluster based on the current resource usage and the desired state defined by the HPA. But **provisioning new nodes takes time**! Even on the fastest servers adding a new node can take a couple of **minutes**.

In the meantime, the Kubernetes scheduler tries its very best to assign our workloads to nodes. It will do so based on **available resources**, constraints, and other factors. Therefore, our newly created pods are likely to be distributed across available nodes where they can still be squeezed, including freshly created ones. The scheduler will try to maintain a balanced distribution of pods. **But during the provisioning process there will be only so much room to grow**.

The situation will worsen if a node is filled with multiple pods without any limits. As you might have guessed, our pods *will* start fighting each other for the resources. All of them with no resource limits and equally targeted by our round-robin load balancers, they will also rise in cpu usage.

So who wins the fight? **Noone**. These are the same pods with the same QoS settings and everything. The kubelet will start to randomly kill your pods, on which there will be requests being processed until new nodes become available to balance the workload, if any... However, critical services such as DNS, kube-proxy, and monitoring tools are less likely to be affected, as they typically run as part of a higher QoS class unless your deem your application more important.

### Result for Endusers

As you might imagine, when your **requests are getting killed** on the cluster, your end-users will not be perceived with happiness. Running requests might also get OOM Killed and therfore **you can not even guarantee the state of your appplication**. A higher CPU-request for the deployment would not have done much either, as the sheduler will always try to shedule pods according to their resource requirements. Your cluster is still going crazy, most likely, when you finally get all the nodes you wanted provisioned, the users will already be gone for them. It sucks for everyone involved and people will remember your error page and potentially loosing data.

### Limits are not necessary but required in order to scale gradually

Lets consider a scenario with limits in place and change our deploymment accordingly:

``` YAML
apiVersion: apps/v1
kind: Deployment
metadata:
  name: service-a
spec:
  replicas: 1
  selector:
    matchLabels:
      app: service-a
  template:
    metadata:
      labels:
        app: service-a
    spec:
      containers:
      - name: service-a-container
        image: <your-image>
        resources:
          requests:
            cpu: 200m
          limits:
            cpu: 500m
```

In this scenario, each pod running the Service A container is limited to a maximum of 500 millicores of CPU usage. The resource requests remain the same, with each pod requesting 200 millicores of CPU.

When a sudden surge in requests overwhelms the service, the pods' CPU usage increases. However, **due to the resource limits, each pod's CPU usage cannot exceed 500 millicores**. This prevents the pods from consuming all the available CPU resources on the node, reducing the likelihood of resource contention issues.

The HPA will still monitor the actual CPU usage and still attempt to scale the number of replicas based on the target utilization. Suppose the target CPU utilization is set at 50%. If there's currently 1 pod running with a 200m CPU request, and the actual CPU usage is 500m (due to the resource limit), the HPA calculates ceil[1 * (500m / (1 * 200m * 50%))] = 5 and will try to scale to 5 replicas. Spawning 5 pods to accommodate the workload will happen quick and easy as most likely no new node will have to be provisioned yet. **That still will not be enough for your traffic spike, but it will do this again 15 seconds later**.

As the number of replicas increases, the total CPU usage observed by the HPA will be capped by the combined CPU limits of all running pods. **This reduces the likelihood of over-scaling based on CPU usage spikes and will not kill your requests**.
The Cluster Autoscaler still operates to ensure there are enough nodes to accommodate the desired number of replicas and you cann still scale as much as you would like, but the reduced CPU usage due to the limits will likely require fewer nodes to be provisioned.
The Kubernetes scheduler continues to assign workloads to nodes based on available resources, constraints, and other factors, but with resource limits in place, resource contention is *less likely* to occur.

In this scenario, the pods will *not* fight for resources as aggressively as they would without limits, and critical services such as DNS, kube-proxy, and monitoring tools are less likely to be affected.
**Setting resource limits helps maintain stability and predictability within the cluster**. It also encourages better resource management and more efficient scaling.

Again Kubernetes' HPA operates as a control loop that runs intermittently, with an interval set by the **--horizontal-pod-autoscaler-sync-period** parameter, therefore our calculation will be repeated every 15 seconds and scaling will be much more smooth. The cluster autoscaler is also able to be configured to automatically provision nodes ones your resources are being exhausted at a limit you define and it will have more time to add new nodes.


## Result for Endusers

When a container in Kubernetes consumes all of its allocated CPU limit, the system will take action to prevent it from using more CPU resources than the specified limit. The Linux kernel employs a technique called **CPU throttling**, which restricts the container's CPU usage to adhere to the imposed limit. **The most immediate consequence of a container using all of its CPU limit is a reduction in performance**. As the container's CPU usage is throttled, it may not be able to process requests as quickly as it could without the limit. This can result in increased latency and slower response times for users interacting with the containerized application. Which will not make your users happy, but **keep the state of your application safer** as the OOM-Reaper is much less likely to do a drive-by.

## the only way to win: Embracing Buffer Nodes and Resource Monitoring for Predictable Scaling in Kubernetes

In Kubernetes, managing cluster resources effectively and ensuring high-quality service delivery is vital for a successful application deployment. As you have seen before, **simply setting limits does not guarantee high avalability**. One way to achieve this balance is by maintaining buffer nodes and performing regular checks on resource limits to plan scaling events accordingly. By incorporating these strategies, you can ensure that your cluster has the capacity to handle unexpected traffic spikes while maintaining resource limits to provide a consistent user experience.

### Overprovisioning: A Safety Net for Your Cluster
Buffer nodes (also called overprovisioning) serve as a safety cushion for your cluster, providing extra capacity to accommodate sudden increases in workload. By maintaining **at least one buffer node in your cluster**, you can ensure that there's always a node available to handle traffic spikes more smoothly, allowing your application to scale linearly without sacrificing quality.

Having buffer nodes not only helps to keep your system responsive during traffic surges but also provides a more predictable environment for autoscaling. When combined with well-configured autoscalers, buffer nodes help prevent overloading existing nodes and facilitate smoother scaling, reducing the risk of resource contention and performance degradation.

This is actually also another point for setting up resource limits: You will be much happier setting the number of buffer nodes you need, because they are relative to your resource requirements.
If you want to be able to double the workload that you usually have quickly, they will give you at least some metric to start with.
Increasing the replicas for your deployment and just letting them idle, could also work the same way, but this approach is much more dynamic, especially when you have more than one workload in your cluster, or your system consists of multiple microservices with different concerns.

**Advantages of overprovisioning**:

* Absorb sudden spikes in resource demands: Buffer nodes act as extra capacity in your cluster, enabling it to handle sudden increases in resource usage, such as a surge in incoming requests. This helps to prevent resource starvation for existing workloads and ensures that there is room to accommodate new or expanded workloads.

* Reduced provisioning delays: Having buffer nodes readily available means that when additional capacity is needed, the cluster can quickly allocate resources without waiting for new nodes to be provisioned. This can result in faster scaling response times, maintaining consistent application performance during periods of increased demand.

* Increased reliability and stability: Buffer nodes provide a safety margin to protect your cluster from unexpected resource shortages, which could lead to degraded performance or even downtime. By maintaining a buffer of available nodes, you can mitigate the risk of such incidents and maintain a more stable environment for your workloads.

### Resource Monitoring: The Key to Proactive Scaling
Regular checks on resource limits play a critical role in planning scaling events effectively. By closely monitoring resource usage across your cluster, you can identify trends and patterns that may indicate the need for adjustments in resource allocation or autoscaling configuration.

Proactive resource monitoring enables you to anticipate and prepare for scaling events, ensuring that your application can continue to deliver consistent performance even as demand fluctuates. By keeping a close eye on resource limits and usage, you can make data-driven decisions about when and how to scale, helping to maintain optimal resource utilization and minimize costs.

**Advantages of regular limit measuring**:

* Optimized resource allocation: Regularly measuring and evaluating resource limits helps to ensure that your workloads have the appropriate amount of resources to perform optimally. This can result in more efficient use of cluster resources and prevent both over-allocation and under-allocation of resources.

* Improved application performance: By monitoring and adjusting resource limits as needed, you can ensure that your applications have the necessary resources to maintain high performance levels. This can lead to better user experience, reduced latency, and faster response times.

* Early detection of potential issues: Regularly measuring resource limits can help you identify potential problems, such as resource contention or unanticipated spikes in resource usage. By detecting these issues early, you can take proactive measures to address them, such as fine-tuning resource allocations or scaling your workloads, before they significantly impact your cluster's performance.

### Result for Endusers

By incorporating buffer nodes and regular resource monitoring into your Kubernetes strategy, you can create a more resilient and predictable environment for your application. These practices, combined with sensible resource limits, ensure that your cluster can respond effectively to changing demand while maintaining a high level of service quality. Now, there is no guarantee that there still wont be a spike or DDOS that will impact your service quality, but this will be an excellent start for your services to achie true HA.

Together, buffer nodes and resource monitoring empower you to plan scaling events more effectively, helping you to strike the right balance between resource allocation and application performance. As a result, you can deliver a seamless user experience, even when faced with unexpected traffic spikes or fluctuations in demand.


## Best Practices for Managing Kubernetes CPU Limits
The key to success lies in **deploying a service without limits initially**, and **then swiftly analyzing its usage with tools like Prometheus and Grafana**. With this information in hand, configure the service's requests, limits, and HPA settings accordingly. You actually ned to define a **scaling strategy that fits your needs** and its ill-advised to simply slap requests and limits onto the deployment and call it a day.

### Monitoring and Fine-tuning
Ensure the proper functioning of your service by setting up alerts for when certain boundaries are reached and **regularly reviewing logs, metrics, and scaling activities**. Fine-tune the configuration over time and consider implementing soak/load tests to ensure your services meet desired Service Level Agreements (SLAs) and performance parameters.

### Impact on Cost Optimization
**Properly managing CPU limits can lead to significant cost savings and optimization benefits.** By preventing resource overallocation or underutilization, organizations can make the most of their available resources and avoid paying for unnecessary computing power. A prime example for this would be managing your cost-centers with [Kubecost](https://www.kubecost.com/). Kubecost assists in monitoring and managing costs and capacities within Kubernetes environments. By integrating with the existing infrastructure, it enables teams to track, manage, and minimize expenditures effectively.

### Managing Resource Constraints Beyond CPU Limits
In addition to CPU limits, it is crucial to consider other resource constraints, such as memory limits. Implementing appropriate limits for memory and other resources can further improve application performance, cost optimization, and overall stability of your Kubernetes environment. Please also do not forget about your Storage. I have seen it many times that a pod requires a VolumeClaim many times of that it actually requires.

### Managing CPU Limits in Different Kubernetes Environments
Addressing the management of CPU limits in various environments, such as on-premises, public cloud, or hybrid cloud, is essential for a comprehensive understanding. Each environment may require unique approaches to managing resources, depending on factors like infrastructure, networking, and available tooling. For instance, your Dev-Cluster could be hosted on-premise or a cheaper hoster where resource requests and limits are not costing you an arm and a leg and later be transfered to your high-available managed cloud.

### Role of Kubernetes Operators in Resource Management
Kubernetes Operators can help manage and maintain application-specific custom resources, including the management of CPU limits. By automating complex operational tasks, Operators can streamline the process of managing resources in Kubernetes, ensuring applications run smoothly and efficiently. By applying a little bit of coding, you may achieve even better results than the HPA because you can go into application specific details. You could also use an operator to dynamically scale the resource limits and request based on time of day or any other custom metric you desire.

### Impact on Application Performance and User Experience
Improperly managed CPU limits can negatively affect application performance and end-user experience. Latency, service disruptions, and unpredictable response times may result from inadequate resource allocation. Implementing appropriate CPU limits ensures consistent performance, enabling a positive user experience. The best microservice setup in the world will not help you at all if your networking is clogged or your if your node is simply out of ports to allocate. This is specifically true for VOIP applications where an call can have multiple ports assigned.

## The Ongoing Quest for the Optimal Kubernetes Configuration
The journey towards the ideal Kubernetes configuration is an **ongoing process**. By being cautious of the pitfalls associated with a no-limits approach, you can avoid potential issues. Instead, approach each service with care, crafting a finely tuned balance between limits, resources, and scalability. This method enables you to resist the allure of the no-limits philosophy and achieve a well-performing, consistent, and scalable infrastructure.
Or differently said... there is no such thing as a free lunch, finetune your systems and keep your logging sharp.


### Bonus: Where do the metrics come from

For per-Pod custom metrics, the controller operates similarly to per-Pod resource metrics, but it works with raw values instead of utilization values. For object metrics and external metrics, a single metric is fetched, compared to the target value, and used to produce a ratio.

A common use case for HPA is to configure it to fetch metrics from aggregated APIs like metrics.k8s.io, custom.metrics.k8s.io, or external.metrics.k8s.io. The metrics.k8s.io API is typically provided by an add-on called [***Metrics Server***](https://github.com/kubernetes-sigs/metrics-server), which must be launched separately for some distributions.

HPA's controller accesses corresponding workload resources that support scaling, such as Deployments and StatefulSets. These resources have a subresource named scale, which allows dynamic setting of the number of replicas and examination of their current states.

### Bonus: watching the HPA react

As an another illustration, let's say the present metric value is 200m, and the target value is 100m. In this case, the replica count would be multiplied by two, as 200.0 / 100.0 equals 2.0. Therefore the it makes sure that two pods are launched. Conversely, if the current value is 50m, the replica count would be reduced by half, since 50.0 / 100.0 equals 0.5. The control plane refrains from any scaling actions if the computed ratio is close enough to 1.0 (within a globally adjustable tolerance, set to 0.1 by default). Therefore only one pod is running, unless of course you set a different replica limit.

In real life this might look like this:

``` BASH 
kubectl get hpa <workload> --watch

NAME       REFERENCE                     TARGET      MINPODS   MAXPODS   REPLICAS   AGE
workload   Deployment/workload/scale    200 / 100    1         10        2          3m
```

In addition to resource requests and limits, you can use Kubernetes features like Pod Priority and Preemption to ensure that critical workloads have the necessary resources even during resource contention. Pod Priority and Preemption allow Kubernetes to evict lower-priority pods first to make room for higher-priority ones when resources are scarce, however, there are ultimately limited resources available on a node.

### Bonus: About Cluster Autoscaler

The Cluster Autoscaler is a sophisticated tool designed to automatically modify the size of a Kubernetes cluster based on certain conditions. It primarily addresses two scenarios: first, when pods in the cluster fail to run due to a lack of available resources; and second, when nodes within the cluster are underutilized over a prolonged duration and the pods they host can be efficiently reassigned to other existing nodes.

By dynamically adapting the cluster size, the Cluster Autoscaler ensures optimal resource allocation, minimizing the occurrence of resource-related issues while maximizing the overall efficiency of the cluster. This provides a robust and efficient solution for managing Kubernetes clusters in various environments, ultimately delivering a more streamlined and professional experience for users.

You can find out if it works on your cloud out of the box right in the [kubernetes github org for autoscalers](https://github.com/kubernetes/autoscaler/tree/master/cluster-autoscaler)

### Sources
* [Horizontal Pod Autoscaler](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/)
* [HPA Walkthrough](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/)
* [Cluster Autoscaler](https://github.com/kubernetes/autoscaler/tree/master/cluster-autoscaler)
* [More Kubernetes Autoscaler](https://github.com/kubernetes/autoscaler)
* [How to Fix OOMKilled Kubernetes Error (Exit Code 137)](https://komodor.com/learn/how-to-fix-oomkilled-exit-code-137/)
* [Kubecost](https://www.kubecost.com/)
* [Kubecost Docs](https://docs.kubecost.com/)
* [Metrics-Server](https://github.com/kubernetes-sigs/metrics-server)

