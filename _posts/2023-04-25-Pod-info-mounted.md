---
layout: post
title: Embracing the Kubernetes Downward API
subtitle: harness the power of Pod and container fields within containers
categories: kubernetes
tags: [kubernetes, kubernetes-pod, kubernetes-downward-api]
---

In the realm of Kubernetes, it is sometimes important to expose certain Pod and container fields to containers running within the Pod. 
These approaches, collectively known as the _downward API_, enable developers to harness the power of Pod and container fields within their containers.

The Kubernetes Downward API is an essential mechanism that facilitates containers in a Kubernetes cluster to obtain metadata regarding themselves or the environment in which they operate, which enables us to make more solid decisions. It enables a container to gather information about its configuration and additional metadata. This information is valuable for generating configuration files, implementing context-aware behavior, or conducting monitoring.

The Downward API can be employed through two methods:

**Environment Variables**: By utilizing the env or envFrom field in the container's specification, pod or container metadata can be injected into a container's environment variables. The metadata is exposed as environment variables and can be accessed by the applications operating within the container.

**Volumes**: The Downward API can also populate files within a volume. This is achieved using a DownwardAPIVolume, a Kubernetes volume type capable of exposing pod metadata as files in a directory. The volume can subsequently be mounted into the container's filesystem at a specified path.

Today I am going to make sure that you understand both and can use them with confidence in your deployments. But first I want to make clear that it saidly **does not support all the fields in the podspec**.
This is a detailed list of fields that can be used with the it and the methods they are available through (Environment variables, Volume files, or both):

1. metadata.name (Both)
   - The name of the pod.

2. metadata.namespace (Both)
   - The namespace the pod is running in.

3. metadata.labels (Volume files)
   - A set of key-value pairs (labels) attached to the pod.

4. metadata.annotations (Volume files)
   - A set of key-value pairs (annotations) providing additional non-identifying information about the pod.

5. status.podIP (Environment variables)
   - The IP address of the pod.

6. spec.nodeName (Environment variables)
   - The name of the node the pod is running on.

7. status.hostIP (Environment variables)
   -  The IP address of the node where the pod is running.

8. spec.serviceAccountName (Environment variables)
   - The name of the service account associated with the pod.

9. metadata.uid (Environment variables)
   - The unique identifier (UID) assigned to the pod by the Kubernetes system.

10. resources.requests.cpu (Environment variables)
   - The amount of CPU requested by the container.

11. resources.requests.memory (Environment variables)
   - The amount of memory requested by the container.

12. resources.limits.cpu (Environment variables)
   - The maximum amount of CPU allowed for the container.

13. resources.limits.memory (Environment variables)
   - The maximum amount of memory allowed for the container.


## Using the downwards API with volumes

>Note: the downwards api for volumes currently is pretty limited. The only supported values are: "metadata.annotations", "metadata.labels", "metadata.name", "metadata.namespace", "metadata.uid"

In order to expose the pods labels and annotations, I will create a Pod with a single container and project Pod-level fields into the running container as files. The Pod manifest provided below reveals a `downwardAPI` volume, which the container mounts at `/etc/downwardAPI`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: learning-about-downwardapi
  labels:
    zone: DE-Falkenstein
    cluster: test-meadow
  annotations:
    app: downwardAPI
    team: devops
spec:
  containers:
    - name: demo-container
      image: registry.k8s.io/busybox
      command: ["sh", "-c"]
      args:
      - while true; do
          if [[ -e /etc/downwardapi/labels ]]; then
            echo -en 'Labels:\n'; cat /etc/downwardapi/labels; fi;
          if [[ -e /etc/downwardapi/annotations ]]; then
            echo -en '\nAnnotations:\n'; cat /etc/downwardapi/annotations; fi;
          sleep 5;
        done;
      volumeMounts:
        - name: downwardapi
          mountPath: /etc/downwardapi
  volumes:
    - name: downwardapi
      downwardAPI:
        items:
          - path: "labels"
            fieldRef:
              fieldPath: metadata.labels
          - path: "annotations"
            fieldRef:
              fieldPath: metadata.annotations
          - path: "podName"
            fieldRef:
              fieldPath: metadata.name
          - path: "Namespace"
            fieldRef:
              fieldPath: metadata.namespace              
```

## reviewing the container definition

The main components are briefly described below:

* image: The container image is sourced from the registry.k8s.io repository, with the specific image being "busybox", a lightweight and versatile Linux distribution. In this case it will help use outputting our stored information

* args: This is a list of arguments that will be passed to the shell command. In this case, it contains a single argument that is a multi-line shell script. The shell script does the following:
    a. It runs an infinite loop using the "while true" construct.
    b. Within the loop, it checks if the file "/etc/downwardapi/labels" exists by using the "-e" flag in the conditional expression. If it exists, it prints a newline character ('\n') twice and then the contents of the file using "cat".
    c. Similarly, it checks if the file "/etc/downwardapi/annotations" exists, and if so, prints a newline character ('\n') twice and the contents of the file using "cat".
    d. The loop then pauses for 5 seconds using the "sleep 5" command before reiterating through the process.

Therefore, we define a Pod called "learning-about-downwardapi" that runs a BusyBox image and executes a shell script, which continuously checks for the existence of two files (/etc/downwardapi/labels and /etc/downwardapi/annotations), and prints their contents if they exist, before pausing for 5 seconds and repeating the process.
We can observe that the Pod features a downwardsAPI Volume, and the container kindly mounts the volume at /etc/downwardsapi as files, so the pod can access the information.
The first element represents that the value of the Pod's metadata.labels field should be saved in a file called 'labels'. 
The second element proposes that the value of the Pod's annotations field should be stored in a file named 'annotations'.

## applying and testing

Now that we have prepared our pod definition, lets apply it with:

```sh 
kubectl apply -f ./downwardsapitest.yaml 
```
And since I have not provided any namespace, it will be in the default namespace and we can retrieve the logs with:

```sh 
kubectl logs learning-about-downwardapi
```

and our output looks something like this:

```sh
Labels:
cluster="test-meadow"
zone="DE-Falkenstein"
Annotations:
app="downwardAPI"
kubectl.kubernetes.io/last-applied-configuration="{...}"
kubernetes.io/config.seen="2023-04-25T07:29:39.669534852Z"
kubernetes.io/config.source="api"
```

As you can see, all of the labels and annotations have succesfully been mounted to the pod and we can access them through the container file system.
This is particularly  usefull, when your code makes decisions based on these parameters.

You also can simply exec the running pod itself to verify the files existance:

```sh 
kubectl exec -it learning-about-downwardapi -- sh
```

where _cat_ will give you the same results:

```sh 
/# cat /etc/downwardapi/labels
cluster="test-meadow"
zone="DE-Falkenstein"/ # 
```

What is more interesting though is if the actual contents of the folder */etc/downardsapi*! Let me show you by runing *ls*:

```sh 
/# cd /etc/downwardapi/
/etc/downwardapi # ls -la
total 4
drwxrwxrwt    3 root     root           120 Apr 25 07:37 .
drwxr-xr-x    1 root     root          4096 Apr 25 07:37 ..
drwxr-xr-x    2 root     root            80 Apr 25 07:37 ..2023_04_25_07_37_05.3522933447
lrwxrwxrwx    1 root     root            32 Apr 25 07:37 ..data -> ..2023_04_25_07_37_05.3522933447
lrwxrwxrwx    1 root     root            18 Apr 25 07:37 annotations -> ..data/annotations
lrwxrwxrwx    1 root     root            13 Apr 25 07:37 labels -> ..data/labels
```

In the generated output, it is evident that both the labels and annotations files are located within a temporary subdirectory. In our specific example, the subdirectory is denoted as "..2023_04_25_07_37_05.3522933447". Within the "/etc/downwardapi" directory, "..data" functions as a symbolic link that connects to the aforementioned temporary subdirectory. Additionally, within the same "/etc/downwardapi" directory, both the labels and annotations serve as symbolic links, so they can be updated!

Utilizing symbolic links facilitates dynamic and atomic updates of the metadata. This is achieved through writing updates to a new temporary directory, followed by an atomic update of the "..data" symlink employing the "rename(2)" system call. If you are used to deploying without kubernetes, you may remember this approach from countless release-deploy scripts out there.

Lets patch our pod with a new annotation to see how that works.

```sh 
kubectl patch pod learning-about-downwardapi -p '{"metadata":{"annotations":{"updatestatus":"in-progress"}}}'
pod/learning-about-downwardapi patched  
```

and we exec our running pod again with
```sh 
kubectl exec -it learning-about-downwardapi -- sh
```

and check out our mounted folder:

```sh 
cd /etc/downwardapi/
/etc/downwardapi # ls -la
total 4
drwxrwxrwt    3 root     root           120 Apr 25 07:47 .
drwxr-xr-x    1 root     root          4096 Apr 25 07:37 ..
drwxr-xr-x    2 root     root            80 Apr 25 07:47 ..2023_04_25_07_47_52.1041351476
lrwxrwxrwx    1 root     root            32 Apr 25 07:47 ..data -> ..2023_04_25_07_47_52.1041351476
lrwxrwxrwx    1 root     root            18 Apr 25 07:37 annotations -> ..data/annotations
lrwxrwxrwx    1 root     root            13 Apr 25 07:37 labels -> ..data/labels
```

As you can see the date has changed, so lets review our data:

```sh 
/etc/downwardapi # cat annotations 
app="downwardAPI"
kubectl.kubernetes.io/last-applied-configuration="{...}"
kubernetes.io/config.seen="2023-04-25T07:37:05.107154061Z"
kubernetes.io/config.source="api"
team="devops"
updatestatus="in-progress"
```

And there you have it, this way you can mount critical information directly into your pods and of course it does not stop at labels and annotations. You can mount the entire podspec this way.

## Why is this important

Pods are considered immutable once they are created. If you need to change the environment variables for a running pod, you must create a new pod with the updated environment variables.
However **Labels and annotations are indeed exceptions to the immutability of pods.**

Using this technique and code that uses for instance a filesystemwatcher, your program will be able to adapt to changes in annotations and labels which are currently widely used with operators or controllers.
**There is a huge opportunity to have your pod react in real time to changes in the outside world, without actually stopping the container and ensuring downtime!**

# Using the Downwards API with Environment variables

Environment variables are not as flexible as the mounted volumes, as I have demostrated in the last chapter, but they are still usefull for the information they contain.
**You cannot directly patch environment variables in a running pod.** If you need to change the environment variables for a running pod, you must create a new one with the updated environment variables and have to terminate the old one before.
In the case of a Deployment, StatefulSet, or DaemonSet, you can update the environment variables in the respective template spec, which will trigger a rolling update to replace the existing pods with new ones that have the updated environment variables.
They do however offer invaluable information as you can see in the example below:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: learning-about-downwardapi
  labels:
    zone: DE-Falkenstein
    cluster: test-meadow
  annotations:
    app: downwardAPI
    team: devops
spec:
  containers:
    - name: demo-container
      image: registry.k8s.io/busybox
      command: ["sh", "-c"]
      args:
      - while true; do
          echo -en 'Pod Name:\n'; echo $POD_NAME;
          echo -en '\nNamespace:\n'; echo $POD_NAMESPACE;
          echo -en '\nPod IP:\n'; echo $POD_IP;
          echo -en '\nNode Name:\n'; echo $NODE_NAME;
          echo -en '\nHost IP:\n'; echo $HOST_IP;
          echo -en '\nService Account Name:\n'; echo $SERVICE_ACCOUNT_NAME;
          echo -en '\nUID:\n'; echo $POD_UID;
          echo -en '\nCPU Request:\n'; echo $CPU_REQUEST;
          echo -en '\nCPU Limit:\n'; echo $CPU_LIMIT;
          echo -en '\nMemory Request:\n'; echo $MEMORY_REQUEST;
          echo -en '\nMemory Limit:\n'; echo $MEMORY_LIMIT;
          sleep 5;
        done;
      env:
        - name: POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        - name: POD_NAMESPACE
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
        - name: POD_IP
          valueFrom:
            fieldRef:
              fieldPath: status.podIP
        - name: NODE_NAME
          valueFrom:
            fieldRef:
              fieldPath: spec.nodeName
        - name: HOST_IP
          valueFrom:
            fieldRef:
              fieldPath: status.hostIP
        - name: SERVICE_ACCOUNT_NAME
          valueFrom:
            fieldRef:
              fieldPath: spec.serviceAccountName
        - name: POD_UID
          valueFrom:
            fieldRef:
              fieldPath: metadata.uid
        - name: CPU_REQUEST
          valueFrom:
            resourceFieldRef:
              containerName: demo-container
              resource: requests.cpu
        - name: CPU_LIMIT
          valueFrom:
            resourceFieldRef:
              containerName: demo-container
              resource: limits.cpu
        - name: MEMORY_REQUEST
          valueFrom:
            resourceFieldRef:
              containerName: demo-container
              resource: requests.memory
        - name: MEMORY_LIMIT
          valueFrom:
            resourceFieldRef:
              containerName: demo-container
              resource: limits.memory
      resources:
        requests:
          cpu: 100m
          memory: 100Mi
        limits:
          cpu: 200m
          memory: 200Mi


```
# mount a specific annotation as environment variable
/E: 04.10.23

One of my readers pointed out, that it is indeed possible to mount a specific annotation as an environment variable into a pod by usin the fieldPath ``metadata.annotations[yourAnnotation]`` which will be plenty usefull for directly mounting an annotation.
```
      env:
        - name: SOME_ANNOTATION
          valueFrom:
            fieldRef:
              fieldPath: metadata.annotations[k8s.io/some-annotation]
```
**However** please be aware that, opposed to a volumemount, ENV variables will not be updated during runtime if you use it this way. So while it is indeed possible, please use it with care.

# combining it all for all the information

Finally lets combine the environment variables with the volumemounts and check all the information together. This does not change what we have learned, but gives us a complete overview of information we can read in the pod:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: learning-about-downwardapi
  labels:
    zone: DE-Falkenstein
    cluster: test-meadow
  annotations:
    app: downwardAPI
    team: devops
spec:
  containers:
    - name: demo-container
      image: registry.k8s.io/busybox
      command: ["sh", "-c"]
      args:
      - while true; do
          echo -en 'Pod Name:\n'; echo $POD_NAME;
          echo -en '\nNamespace:\n'; echo $POD_NAMESPACE;
          echo -en '\nPod IP:\n'; echo $POD_IP;
          echo -en '\nNode Name:\n'; echo $NODE_NAME;
          echo -en '\nHost IP:\n'; echo $HOST_IP;
          echo -en '\nService Account Name:\n'; echo $SERVICE_ACCOUNT_NAME;
          echo -en '\nUID:\n'; echo $POD_UID;
          echo -en '\nCPU Request:\n'; echo $CPU_REQUEST;
          echo -en '\nCPU Limit:\n'; echo $CPU_LIMIT;
          echo -en '\nMemory Request:\n'; echo $MEMORY_REQUEST;
          echo -en '\nMemory Limit:\n'; echo $MEMORY_LIMIT;
          if [[ -e /etc/downwardapi/labels ]]; then
            echo -en '\nLabels:\n'; cat /etc/downwardapi/labels; fi;
          if [[ -e /etc/downwardapi/annotations ]]; then
            echo -en '\nAnnotations:\n'; cat /etc/downwardapi/annotations; fi;
          sleep 5;
        done;
      env:
        - name: POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        - name: POD_NAMESPACE
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
        - name: POD_IP
          valueFrom:
            fieldRef:
              fieldPath: status.podIP
        - name: NODE_NAME
          valueFrom:
            fieldRef:
              fieldPath: spec.nodeName
        - name: HOST_IP
          valueFrom:
            fieldRef:
              fieldPath: status.hostIP
        - name: SERVICE_ACCOUNT_NAME
          valueFrom:
            fieldRef:
              fieldPath: spec.serviceAccountName
        - name: POD_UID
          valueFrom:
            fieldRef:
              fieldPath: metadata.uid
        - name: CPU_REQUEST
          valueFrom:
            resourceFieldRef:
              containerName: demo-container
              resource: requests.cpu
        - name: CPU_LIMIT
          valueFrom:
            resourceFieldRef:
              containerName: demo-container
              resource: limits.cpu
        - name: MEMORY_REQUEST
          valueFrom:
            resourceFieldRef:
              containerName: demo-container
              resource: requests.memory
        - name: MEMORY_LIMIT
          valueFrom:
            resourceFieldRef:
              containerName: demo-container
              resource: limits.memory
      volumeMounts:
        - name: downwardapi
          mountPath: /etc/downwardapi
      resources:
        requests:
          cpu: 100m
          memory: 100Mi
        limits:
          cpu: 200m
          memory: 200Mi
  volumes:
    - name: downwardapi
      downwardAPI:
        items:
          - path: "labels"
            fieldRef:
              fieldPath: metadata.labels
          - path: "annotations"
            fieldRef:
              fieldPath: metadata.annotations
          - path: "podName"
            fieldRef:
              fieldPath: metadata.name
          - path: "Namespace"
            fieldRef:
                fieldPath: metadata.namespace
```
And let's check out the example output:

```SH
kubectl logs learning-about-downwardapi 
Pod Name:
learning-about-downwardapi

Namespace:
default

Pod IP:
10.42.0.112

Node Name:
ultron

Host IP:
172.17.66.235

Service Account Name:
default

UID:
afca6d22-a1df-4787-a947-bccf6b1015f8

CPU Request:
1

CPU Limit:
1

Memory Request:
104857600

Memory Limit:
209715200

Labels:
cluster="test-meadow"
zone="DE-Falkenstein"
Annotations:
app="downwardAPI"
kubectl.kubernetes.io/last-applied-configuration="{...}"
kubernetes.io/config.seen="2023-04-25T11:31:49.978886812Z"
kubernetes.io/config.source="api"
team="devops"
```

There you have it, extended information of the pod, directly mounted into it. The Kubernetes Downward API provides a powerful and flexible way to expose pod metadata and container resources to the containers running within a pod. By making use of environment variables and volume mounts, developers can access essential information about the pod, such as its name, namespace, labels, and annotations, as well as the allocated resources and limits for the container.

Exposing this information within containers has several practical applications. For instance, it enables applications to adjust their behavior according to the available resources or the environment they are running in. Additionally, it can facilitate the development of monitoring and logging tools that can access and report the pod's metadata, making it easier to track and manage applications deployed in a Kubernetes cluster.

Furthermore, it helps create self-aware applications that can dynamically adapt to their environment, improving overall system resilience and responsiveness. This feature can be particularly useful when dealing with auto-scaling, rolling updates, and other scenarios where applications must gracefully handle changes in their environment.

The Kubernetes Downward API can be an invaluable tool for developers and operators working with containerized applications in a cluster environment. By providing easy access to critical pod metadata and container resources, it allows for more robust, flexible, and adaptive applications that can better handle the ever-changing landscape of modern distributed systems.
