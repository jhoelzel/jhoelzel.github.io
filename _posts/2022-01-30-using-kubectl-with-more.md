---
layout: post
title: Using kubectl with more for easy scrolling
subtitle: Read longer output with ease!
categories: hints
tags: [kubectl, more, cli]
---

Sometimes kubectl will return output that is just to long to handle. With some integrated terminals cutting the output after a certain lenth (Iam looking at you Visual Studio Code), its nice to be able to scroll throught kubetl output much more easily.

For this usecase the more tool is easily used <https://wiki.ubuntuusers.de/more/>

```
# with more you can easily scroll through the kubectl output:
kubectl get deployment.apps/cluster-autoscaler -o yaml | more
```