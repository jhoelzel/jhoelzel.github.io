---
layout: post
title: Restarting a container without downtime
subtitle: or how to reload without taking down prod!
categories: kubectl
tags: [kubectl, docker, container]
---

In this post I will try to explain how to restart your running pods with kubectl.

## the old way

Before Kubernetes version 1.15, we mostly had only one choice to restart a deployment that is running and that with

``` Console
kubectl scale deployment mydeployment --replicas=0 -n mynamespace
``` 

followed by

``` Console
kubectl scale deployment mydeployment --replicas=1 -n mynamespace
``` 

and as you can probably deduct from the commands, this involved killing all pods involved, and rescaling them.
This frequently lead to downtimes of systems, even if just for seconds and is thus a suboptimal way of restarting your services.

## the new and improved solution

Luckily we do not need to do it this way anymore and can make use of the newly integrated way, of rolling out a deployment with:

``` Console
kubectl -n service rollout restart deployment <mydeployment>
``` 

The avid reader will notice right away that instead of scaling our deployment to 0, therefore killing all the replicas and then booting them back up, we are able to use ar rolling rollout, which will restart our pods one by one and therefore will keep our deployment alive. Therefore guaranteeing that at least one of our pods is still available.

Please not though, if you only have one replica activated, using the rolling restart will still only minimize the downtime of your deployment.
