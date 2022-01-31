---
layout: post
title: Using curl with a fake host header
subtitle: Query resources under different hostnames
categories: hints
tags: [kubectl, more, cli]
---

Sometimes kubectl will return output that is just to long to handle. With some integrated terminals cutting the output after a certain lenth (Iam looking at you Visual Studio Code), its nice to be able to scroll throught kubetl output much more easily.

For this usecase the more tool is easily used <https://wiki.ubuntuusers.de/more/>

``` console
$ curl --resolve yourdomain.com:443:127.0.0.1 https://yourdomain.com/
```
