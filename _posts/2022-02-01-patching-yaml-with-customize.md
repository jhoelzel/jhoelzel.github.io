---
layout: post
title: Patching YAML with kustomize
subtitle: while setting up the hetzner cloud manager
categories: kubernetes
tags: [kubernetes, cloud-manager, kustomize]
---

Often I see myself confronted with the problem of customizing a set of yaml files from a third party. Helm solves this problem with its Chart integration and allows rich modification of the underlying yaml through the definition of values.
While I like helm, sometimes I really need an easier way to set up something and really do not want to bother in setting up or customizing a helm chart.

But good news everyone, there is a tool that already does that for you. Its name is kustomize and I am sure that most of you have by now heard of it.

Kustomize defines itself:

```
Kustomize introduces a template-free way to customize application configuration that simplifies the use of off-the-shelf applications.
```

and it can be found at <https://kustomize.io/>. But the best thing is that it is actually integrated now into kubectl. With a simple ***kubectl apply -k <file>** you are ready to go right now.

Let's look at an actual example. This time we are modifying the default cloud controller manager deployment of the "hetzner cloud" which I regularly use for cheaper k3s sister-clusters.

First, let's check out what we would actually be installing: <https://github.com/hetznercloud/hcloud-cloud-controller-manager/releases/latest/download/ccm-networks.yaml>. I have copied its contents here for better observability, but please check out the link as its definition may change over time.

As you can see we will download the resource directly from Github and you will see it contains multiple definitions for our cluster: 

``` YAML
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: cloud-controller-manager
  namespace: kube-system
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: system:cloud-controller-manager
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
  - kind: ServiceAccount
    name: cloud-controller-manager
    namespace: kube-system
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hcloud-cloud-controller-manager
  namespace: kube-system
spec:
  replicas: 1
  revisionHistoryLimit: 2
  selector:
    matchLabels:
      app: hcloud-cloud-controller-manager
  template:
    metadata:
      labels:
        app: hcloud-cloud-controller-manager
      annotations:
        scheduler.alpha.kubernetes.io/critical-pod: ''
    spec:
      serviceAccountName: cloud-controller-manager
      dnsPolicy: Default
      tolerations:
        # this taint is set by all kubelets running `--cloud-provider=external`
        # so we should tolerate it to schedule the cloud controller manager
        - key: "node.cloudprovider.kubernetes.io/uninitialized"
          value: "true"
          effect: "NoSchedule"
        - key: "CriticalAddonsOnly"
          operator: "Exists"
        # cloud controller manages should be able to run on masters
        - key: "node-role.kubernetes.io/master"
          effect: NoSchedule
          operator: Exists
        - key: "node-role.kubernetes.io/control-plane"
          effect: NoSchedule
          operator: Exists
        - key: "node.kubernetes.io/not-ready"
          effect: "NoSchedule"
      hostNetwork: true
      containers:
        - image: hetznercloud/hcloud-cloud-controller-manager:v1.12.1
          name: hcloud-cloud-controller-manager
          command:
            - "/bin/hcloud-cloud-controller-manager"
            - "--cloud-provider=hcloud"
            - "--leader-elect=false"
            - "--allow-untagged-cloud"
            - "--allocate-node-cidrs=true"
            - "--cluster-cidr=10.244.0.0/16"
          resources:
            requests:
              cpu: 100m
              memory: 50Mi
          env:
            - name: NODE_NAME
              valueFrom:
                fieldRef:
                  fieldPath: spec.nodeName
            - name: HCLOUD_TOKEN
              valueFrom:
                secretKeyRef:
                  name: hcloud
                  key: token
            - name: HCLOUD_NETWORK
              valueFrom:
                secretKeyRef:
                  name: hcloud
                  key: network
```

Thats a lot of YAML right? But since we need all of it we are going to ignore the length. We are going to kustomize this deployment.

First we are going to create our base file, which will download the yaml definition directly.
Let's create a filename with the following content:

kustomization.yaml:

``` YAML
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
- https://github.com/hetznercloud/hcloud-cloud-controller-manager/releases/latest/download/ccm-networks.yaml

patchesStrategicMerge:
- patch.yaml
```

As you can see downloading the yaml is just a matter of adding it to a list inside of our kustomization.yaml. You can add as many sources as you like and also just integrate local files by using a folder based approach.
What is more interesting is the little annotation called ***patchesStrategicMerge*** in our kustomization.yaml. It is a list of file paths where each file should be resolved to a strategic merge patch and result in a final yaml file that will contain our patched definition.
That means it will merge the base definition with the patches you provide. Kustomize shines most when using overlays but this time I just want to show you how to run a quick way to patch the deployment for future uses.

When deploying a cloud controller manager, the chances are hight that you will have to adapt it to your cluster settings. And that's our ideal use case today. As we have seen in the kustomization.yaml, kustomize is looking up a file called patch.yaml, so let us create it and modify the default deployment of the ccm:

``` YAML
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hcloud-cloud-controller-manager
  namespace: kube-system
spec:
  template:
    spec:
      containers:
        - image: hetznercloud/hcloud-cloud-controller-manager:latest
          imagePullPolicy: Always
          name: hcloud-cloud-controller-manager
          command:
            - "/bin/hcloud-cloud-controller-manager"
            - "--cloud-provider=hcloud"
            - "--leader-elect=false"
            - "--allow-untagged-cloud"
            - "--allocate-node-cidrs=true"
            - "--cluster-cidr=10.42.0.0/16"
```

and thats it! All we have to do now is to run

``` Console
$ kubectl apply -k ./
```

and our modified cloud controller will be applied to our cluster. Fully modified and patched, just the way we like it.
As you can see this brings a lot of power to yaml kustomization and therefore I love it.

In the future I might create another post in which I will explain the use and advantage of overlays, but that is for next time.
For now, enjoy patching your deployments with kustmize.


## Sources

<https://kustomize.io/>

<https://kubectl.docs.kubernetes.io/faq/kustomize/>