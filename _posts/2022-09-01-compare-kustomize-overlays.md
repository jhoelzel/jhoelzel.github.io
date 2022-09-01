---
layout: post
title: Compare kustomize overlays
subtitle: and see what really changed
categories: cli
tags: [cli, linux, kustomize]
---

More often than I like, I have to compare different kustomize overlays in order to check how my overlays differ.
Luckily we can simply do this with diff:

``` Bash
$ diff \
  <(kubectl kustomize $OVERLAYS/staging) \
  <(kubectl kustomize $OVERLAYS/production) |\
```

# diff with kustomize an example

For demonstration purposes im using my standard simpleapp deployment, which you can find here: <https://github.com/jhoelzel/simpleapp/tree/master/kube-manifests/kustomize>
It uses two overlays, one for dev and one for prod. The main difference between them is that one uses a Nodeport and the other one uses a load balancer for ingress:

``` Console
$ diff \
  <(kubectl kustomize /var/workspace/tmp/simpleapp/kube-manifests/kustomize/overlays/dev) \
  <(kubectl kustomize /var/workspace/tmp/simpleapp/kube-manifests/kustomize/overlays/prod) |\
```

here we can see the difference between both overlays:

``` Diff
--- /proc/self/fd/11
+++ /proc/self/fd/13
@@ -4,16 +4,16 @@
   labels:
     app: simpleapp
     version: 0.0.1
-  name: simpleapp-nodeport-service
-  namespace: simpleapp-dev
+  name: simpleapp-loadbalancer-service
+  namespace: simpleapp-prod
 spec:
   ports:
   - port: 80
-    targetPort: 80
+    targetPort: 8080
   selector:
     app: simpleapp
     version: 0.0.1
-  type: NodePort
+  type: LoadBalancer
 ---
 apiVersion: apps/v1
 kind: Deployment
@@ -21,7 +21,7 @@
   labels:
     app: simpleapp
   name: simpleapp
-  namespace: simpleapp-dev
+  namespace: simpleapp-prod
 spec:
   replicas: 1
```

Thats it, you can easily see the changes between both environments and act accordingly.

# scrolling the output with more

Once we have many differences, simply outputting to console might be overwhelming, therefore we can use the integrated more to easily scroll the differences:

``` Console
$ diff \
  <(kubectl kustomize /var/workspace/tmp/simpleapp/kube-manifests/kustomize/overlays/dev) \
  <(kubectl kustomize /var/workspace/tmp/simpleapp/kube-manifests/kustomize/overlays/prod) |\
  more
```

# logging the differences to file

The easiest way to compare the changes might be logging the output to file. As we are using standard commands you can simply to that with:

``` Console
$ diff \
  <(kubectl kustomize /var/workspace/tmp/simpleapp/kube-manifests/kustomize/overlays/dev) \
  <(kubectl kustomize /var/workspace/tmp/simpleapp/kube-manifests/kustomize/overlays/prod) |\
  > your_file.log
```

There you have it, comparing differences the almost easy way. The most simple solution for you might me to use your editor for comparison.
In this case you can easily log all files separately using kustomize with:

``` Console
$ diff kubectl kustomize /var/workspace/tmp/simpleapp/kube-manifests/kustomize/overlays/dev > dev.log
$ diff kubectl kustomize /var/workspace/tmp/simpleapp/kube-manifests/kustomize/overlays/prod > prod.log
```

And then use your favorite editor to compare files.

