---
layout: post
title: Getting your kubeconfig from k3s
subtitle: K3s Cluster Access
categories: hints
tags: [kubectl, k3s, config]
---

## the k3s.yaml 

K3s will store its kubeconfig in **/etc/rancher/k3s/k3s.yaml** and is used to configure access to the Kubernetes cluster. Depending on how you are going to use your cluster, you are going to need it to configure your local installed version of kubectl.

It should look something like this:

``` YAML
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: <redacted>
    server: https://127.0.0.1:6443
  name: default
contexts:
- context:
    cluster: default
    user: default
  name: default
current-context: default
kind: Config
preferences: {}
users:
- name: default
  user:
    client-certificate-data: <redacted>
    client-key-data: <redacted>=
```

The keen observer will notice that, apart from my redactions, the IP-Address of the cluster is set to 127.0.0.1 and therefore you will not be able to connect to your control-plane from your localhost.
Therefore we need to update its IP.

**Please note that you should never change the file in the folder itself, because your cluster will be using it communicate**

Thus I make it a habbit to copy the file to a local folder with scp:

``` console
$ scp -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null -i ${var.private_key} ${self.user}@${self.ipv4_address}:/etc/rancher/k3s/k3s.yaml ./kubeconfig.yaml
```

and to then change the IP myself locally:

``` console
$ sed -i -e 's/127.0.0.1/${self.ipv4_address}/g' ./kubeconfig.yaml
```

After that your kubeconfig is ready to go. ( A thing to note is that now is the point to change the cluster or context names locally to easier identify them later on)
The easiest way to use it with kubectl is to export the kubeconfigpath:

``` console
$ export KUBECONFIG=/etc/rancher/k3s/k3s.yaml
```

At this point you are able to query your cluster normally.

Another way to setup the config with kubectl is to pass the config directly:

``` console
$ kubectl --kubeconfig /etc/rancher/k3s/k3s.yaml get pods --all-namespaces
```

which will work just as fine.
