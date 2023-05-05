---
layout: post
title: Kubernetes Headless Services
subtitle: harness the power of Pod and container fields within containers
categories: kubernetes
tags: [kubernetes, kubernetes-services, kubernetes-headless-services]
---

**Kubernetes services** are abstractions that define a logical set of pods and a policy to access them. Services enable loose coupling between pods and **provide a stable IP address**, DNS name, and load balancing for distributing network traffic.

**Headless Services do not provide a stable IP address** are a special type of Kubernetes service that allow for more direct control over pod-to-pod communication and service discovery, particularly useful for stateful applications and custom load balancing scenarios.

## What are Kubernetes Headless Services?

Simply said, a headless service is a Kubernetes **service without a cluster IP**, allowing pods to be directly reached via their individual IP addresses. This facilitates direct communication between pods and bypasses the default load balancing provided by Kubernetes.

Unlike ClusterIP services, which provide a single IP address that load balances traffic to the backend pods, headless services expose each pod's IP address directly. This is useful when pods need to be individually addressed or when custom load balancing is required, or you simply want the endpoints of your service listed without providing access.

The primary usecases for headless services are:

1. **Stateful applications**: Headless services are essential for stateful applications that require stable network identities and direct communication between instances, such as databases and message brokers.

2. **Service discovery**: Headless services enable custom service discovery mechanisms, allowing applications to discover and communicate with individual pod instances.

3. **Custom load balancing**: By exposing individual pod IPs, headless services allow for the implementation of custom load balancing strategies tailored to specific application requirements.

## Headless Services with a regular Deployment

First lets create a deployment for our Headless Service. They are primarily used with StatefulSets but in order to demostrae why we will start with a deployment.

This YAML file creates a deployment for a sample application with 3 replicas.

```YAML
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-app-container
        image: busybox:latest
        args:
        - /bin/sh
        - -c
        - while true; do { echo -e 'HTTP/1.1 200 OK\n\nHello from Busybox!'; } | nc -l -p 80; done
        ports:
        - containerPort: 80
```

Now let's create the corresponding headless service:

```YAML
apiVersion: v1
kind: Service
metadata:
  name: my-headless-service
spec:
  # Set clusterIP to None to create a headless service
  clusterIP: None
  # Selector to match the desired set of pods
  selector:
    app: my-app
  ports:
    - name: http
      protocol: TCP
      port: 80
      targetPort: 80
```

As you can see, there is no magic and the only real difference between a headless and a "normal" service is, that the headless service will not provision a clusterIP for you.

Apply the YAML configuration using `kubectl apply -f <filename.yaml>` to create the headless service.

```SH
$ kubectl apply -f ./example.yaml
deployment.apps/my-app-deployment created
service/my-headless-service created     
```

Verify the creation of the headless service using `kubectl get services` and ensure that the 'CLUSTER-IP' field is set to 'None'.

```SH
$ kubectl get service my-headless-service -o wide
NAME                  TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE   SELECTOR
my-headless-service   ClusterIP   None         <none>        80/TCP    90s   app=my-app
```

If you want to list all endpoints of your service you can simply do so with `kubectl get endpoints <myservice>` which will look something like this:

```SH
$ kubectl get endpoints my-headless-service
NAME                  ENDPOINTS                                            AGE
my-headless-service   10.42.0.124:80,10.42.0.125:80,10.42.0.126:80   4m7s
```

Or if you like some more specific information you can output them as Json like this:

```SH
$ kubectl get endpoints my-headless-service --output=json
{
    "apiVersion": "v1",
    "kind": "Endpoints",
    "metadata": {
        "annotations": {
            "endpoints.kubernetes.io/last-change-trigger-time": "2023-05-02T08:58:35Z"
        },
        "creationTimestamp": "2023-05-02T08:58:20Z",
        "labels": {
            "service.kubernetes.io/headless": ""
        },
        "name": "my-headless-service",
        "namespace": "default",
        "resourceVersion": "169091",
        "uid": "c431ad71-40e8-4140-987f-654e596e957d"
    },
    "subsets": [
        {
            "addresses": [
                {
                    "ip": "10.42.0.137",
                    "nodeName": "ultron",
                    "targetRef": {
                        "kind": "Pod",
                        "name": "my-app-deployment-6694968687-c6rv9",
                        "namespace": "default",
                        "uid": "32a435fb-7938-4a6d-a4c0-ad0439a9dabe"
                    }
                },
                {
                    "ip": "10.42.0.138",
                    "nodeName": "ultron",
                    "targetRef": {
                        "kind": "Pod",
                        "name": "my-app-deployment-6694968687-9djtn",
                        "namespace": "default",
                        "uid": "e45f0d0e-292e-408c-99a1-ecdc7ccaa3cc"
                    }
                },
                {
                    "ip": "10.42.0.139",
                    "nodeName": "ultron",
                    "targetRef": {
                        "kind": "Pod",
                        "name": "my-app-deployment-6694968687-ln985",
                        "namespace": "default",
                        "uid": "fd77afaf-8b5f-4994-9e12-c9ba780ce44f"
                    }
                }
            ],
            "ports": [
                {
                    "name": "http",
                    "port": 80,
                    "protocol": "TCP"
                }
            ]
        }
    ]
}
```

### Discovering pods with DNS

```SH
$ kubectl run -i --tty dnsutils --image=tutum/dnsutils --restart=Never -- sh
If you don't see a command prompt, try pressing enter.
# 
dig my-headless-service.default.svc.cluster.local A

; <<>> DiG 9.9.5-3ubuntu0.2-Ubuntu <<>> my-headless-service.default.svc.cluster.local A
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 20692
;; flags: qr aa rd; QUERY: 1, ANSWER: 3, AUTHORITY: 0, ADDITIONAL: 1
;; WARNING: recursion requested but not available

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;my-headless-service.default.svc.cluster.local. IN A

;; ANSWER SECTION:
my-headless-service.default.svc.cluster.local. 5 IN A 10.42.0.137
my-headless-service.default.svc.cluster.local. 5 IN A 10.42.0.139
my-headless-service.default.svc.cluster.local. 5 IN A 10.42.0.138

;; Query time: 0 msec
;; SERVER: 10.43.0.10#53(10.43.0.10)
;; WHEN: Tue May 02 08:55:22 U
```

### Using SRV records for service discovery 

SRV records are a type of DNS record that provides additional information beyond just the IP address of a service. They are particularly useful for service discovery within a cluster, allowing applications to obtain information about available pod instances and their respective ports.

An SRV record is a DNS record with the format: `_service._protocol.<service_name>.<namespace>.svc.cluster.local`. It contains information about the service's hostname, port, protocol, and priority. Applications can perform DNS lookups using the SRV record to discover the available pod instances and their corresponding ports, allowing them to establish connections and communicate with individual pods. 

This is especially useful for stateful applications that require direct pod-to-pod communication, as well as for custom load balancing scenarios where the built-in Kubernetes load balancing is not sufficient.

In order to check this out we first need to create a busybox container in the same namespace. Next, we can perform an dig command to query the SRV records for the headless service:

```SH
$ kubectl run -i --tty dnsutils --image=tutum/dnsutils --restart=Never -- sh
If you don't see a command prompt, try pressing enter.
# dig _http._tcp.my-headless-service.default.svc.cluster.local SRV

; <<>> DiG 9.9.5-3ubuntu0.2-Ubuntu <<>> _http._tcp.my-headless-service.default.svc.cluster.local SRV
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 61360
;; flags: qr aa rd; QUERY: 1, ANSWER: 3, AUTHORITY: 0, ADDITIONAL: 4
;; WARNING: recursion requested but not available

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;_http._tcp.my-headless-service.default.svc.cluster.local. IN SRV

;; ANSWER SECTION:
_http._tcp.my-headless-service.default.svc.cluster.local. 5 IN SRV 0 33 80 10-42-0-138.my-headless-service.default.svc.cluster.local.
_http._tcp.my-headless-service.default.svc.cluster.local. 5 IN SRV 0 33 80 10-42-0-137.my-headless-service.default.svc.cluster.local.
_http._tcp.my-headless-service.default.svc.cluster.local. 5 IN SRV 0 33 80 10-42-0-139.my-headless-service.default.svc.cluster.local.

;; ADDITIONAL SECTION:
10-42-0-137.my-headless-service.default.svc.cluster.local. 5 IN A 10.42.0.137
10-42-0-139.my-headless-service.default.svc.cluster.local. 5 IN A 10.42.0.139
10-42-0-138.my-headless-service.default.svc.cluster.local. 5 IN A 10.42.0.138

;; Query time: 0 msec
;; SERVER: 10.43.0.10#53(10.43.0.10)
;; WHEN: Tue May 02 08:58:58 UTC 2023
;; MSG SIZE  rcvd: 703
```


> NOTE: Headless Services for deployments automatically create DNS records for each pod, following the pattern `<podip>.<service_name>.<namespace>.svc.cluster.local`.


Now using this information, lets try it out by spinning up a busybox and running wget:

```SH
$ kubectl run -i --tty busybox --image=busybox --restart=Never -- sh 
If you don't see a command prompt, try pressing enter.
/ # wget -O- http://10-42-0-138.my-headless-service.default.svc.cluster.local:80
Connecting to 10-42-0-138.my-headless-service.default.svc.cluster.local:80 (10.42.0.138:80)
writing to stdout
Hello from Busybox!
-                    100% |*****************************************************************************************************************************************************************************************************************************|    20  0:00:00 ETA
written to stdout
```

As you can see the dns name of the pod is using the *pod ip as dns hostnamme* and thus it is not going to be stable across new iterations of the pod or even scaling the deployment up or down.
Let us find out how the same scenario will play out with a StatefullSet:

## Using a SatefullSet

Now that we have seen what happens to a deployment, lets check what happens if we change it to a StatefullSet.

In order to that we have to change our yaml like this:

```YAML
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: my-app-statefulset
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  serviceName: my-headless-service
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-app-container
        image: busybox:latest
        args:
        - /bin/sh
        - -c
        - while true; do { echo -e 'HTTP/1.1 200 OK\n\nHello from Busybox!'; } | nc -l -p 80; done
        ports:
        - containerPort: 80
        readinessProbe:
          tcpSocket:
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 10
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 1Gi
---
apiVersion: v1
kind: Service
metadata:
  name: my-headless-service
spec:
  # Set clusterIP to None to create a headless service
  clusterIP: None
  # Selector to match the desired set of pods
  selector:
    app: my-app
  ports:
    - name: http
      protocol: TCP
      port: 80
      targetPort: 80

```

As you may see, I have changed Deployment to StatefulSet and added a serviceName field in the spec section that points to the headless service we have defined. We have also added a volumeClaimTemplates field to the spec section, which defines a persistent volume claim template that can be used by the StatefulSet's pods to store data.

The StatefulSet maintains a unique identity for each of its pods and ensures that they are created in a predictable order. It is useful when you need to manage stateful applications, like databases, where each instance requires a unique identity and persistent storage.

```SH
kubectl run -i --tty dnsutils --image=tutum/dnsutils --restart=Never -- sh

If you don't see a command prompt, try pressing enter.
dig my-headless-service.default.svc.cluster.local A

; <<>> DiG 9.9.5-3ubuntu0.2-Ubuntu <<>> my-headless-service.default.svc.cluster.local A
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 18912
;; flags: qr aa rd; QUERY: 1, ANSWER: 3, AUTHORITY: 0, ADDITIONAL: 1
;; WARNING: recursion requested but not available

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;my-headless-service.default.svc.cluster.local. IN A

;; ANSWER SECTION:
my-headless-service.default.svc.cluster.local. 5 IN A 10.42.0.148
my-headless-service.default.svc.cluster.local. 5 IN A 10.42.0.143
my-headless-service.default.svc.cluster.local. 5 IN A 10.42.0.146

;; Query time: 0 msec
;; SERVER: 10.43.0.10#53(10.43.0.10)
;; WHEN: Tue May 02 15:00:11 UTC 2023
;; MSG SIZE  rcvd: 257
```

So far the Answer section looks pretty much the same as for the deployment, but watch what happens if we quere the SRV record:

```SH
# dig _http._tcp.my-headless-service.default.svc.cluster.local SRV

; <<>> DiG 9.9.5-3ubuntu0.2-Ubuntu <<>> _http._tcp.my-headless-service.default.svc.cluster.local SRV
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 12868
;; flags: qr aa rd; QUERY: 1, ANSWER: 3, AUTHORITY: 0, ADDITIONAL: 4
;; WARNING: recursion requested but not available

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;_http._tcp.my-headless-service.default.svc.cluster.local. IN SRV

;; ANSWER SECTION:
_http._tcp.my-headless-service.default.svc.cluster.local. 5 IN SRV 0 33 80 my-app-statefulset-0.my-headless-service.default.svc.cluster.local.
_http._tcp.my-headless-service.default.svc.cluster.local. 5 IN SRV 0 33 80 my-app-statefulset-1.my-headless-service.default.svc.cluster.local.
_http._tcp.my-headless-service.default.svc.cluster.local. 5 IN SRV 0 33 80 my-app-statefulset-2.my-headless-service.default.svc.cluster.local.

;; ADDITIONAL SECTION:
my-app-statefulset-1.my-headless-service.default.svc.cluster.local. 5 IN A 10.42.0.146
my-app-statefulset-0.my-headless-service.default.svc.cluster.local. 5 IN A 10.42.0.143
my-app-statefulset-2.my-headless-service.default.svc.cluster.local. 5 IN A 10.42.0.148

;; Query time: 0 msec
;; SERVER: 10.43.0.10#53(10.43.0.10)
;; WHEN: Tue May 02 15:01:52 UTC 2023
;; MSG SIZE  rcvd: 757
```


> NOTE: Headless services automatically create DNS records for each pod, following the pattern `<hostname>.<service_name>.<namespace>.svc.cluster.local`.

### Why headless services work best with StatefullSets

When a StatefulSet is created, Kubernetes assigns a unique identity to each of its pods, in the form of an ordinal index. The SRV records returned by the dig command reflect this by including the ordinal index in the hostname of each pod. For example, my-app-statefulset-0 refers to the first pod in the StatefulSet.

They are predominantly used with StatefulSets because they enable DNS-based service discovery for each individual pod, using the unique hostname assigned by Kubernetes in a stable manner. This allows applications running inside the pods to easily discover and connect to other pods in the StatefulSet, using their unique identities. This **stable network identity** is crucial for stateful applications that require a consistent identity for data replication, leader election, or other distributed systems tasks.


While it is possible to use a headless service with a Deployment, some of the advantages of using a headless service with a StatefulSet do not apply to a Deployment. Here's why:

1. Stable network identity: In a StatefulSet, each pod gets a unique and stable hostname based on the StatefulSet's name and an index (e.g., my-app-statefulset-0, my-app-statefulset-1). Deployments, on the other hand, create pods with random hashes in their names (e.g., my-app-deployment-6f8d6b94c7). Thus, pods created by a Deployment do not have stable hostnames, which makes it difficult to maintain a consistent network identity for stateful applications that rely on it for data replication, leader election, or other tasks. Also as we have seen, the DNS Hostname of a deployed pod will be its IP.

2. Direct access to individual pods: While headless services can still provide direct access to individual pods in a Deployment, the lack of a stable network identity makes it harder to manage communication between specific instances or replicas. Without stable hostnames, clients need to rely on other mechanisms, such as labels or annotations, to identify and communicate with specific instances. This adds complexity to the system and might not be suitable for certain stateful applications. Of course you can query the SRV entry in many iterations, but you need to make sure that your application takes care of it.

3. Ordered and controlled scaling: StatefulSets provide ordered and controlled scaling, ensuring that pods are created and terminated in a specific order (based on their index). This is particularly important for stateful applications where the order of scaling operations can affect data consistency or availability. Deployments, however, scale pods without any specific order, which could lead to issues in stateful applications.

4. Persistent storage: StatefulSets can leverage Stateful Volumes, which ensures that each pod gets its own unique and persistent storage. When a pod is rescheduled, *it retains the same storage*, allowing for seamless data recovery. Deployments do not have this native capability, making it challenging to manage stateful applications that require persistent storage.

Therefore, while headless services can be used with both StatefulSets and Deployments, the advantages of using a headless service with a StatefulSet do not apply to Deployments because of the lack of stable network identity, unordered scaling, and absence of native persistent storage support.

### Finishing up

Kubernetes headless services provide a powerful way to manage direct pod-to-pod communication, service discovery, and custom load balancing. They are particularly useful for stateful applications and situations where the default load balancing provided by Kubernetes is insufficient and are an important tool in a developer's arsenal.

