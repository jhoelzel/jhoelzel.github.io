---
layout: post
title: Using curl with a fake host header
subtitle: Query resources under different hostnames
categories: hints
tags: [curl, cli]
---

In order to work with fake resources on your localhost you sometimes have to fake your hostname.
With curl you can simply do that by providing your own domain resolution into the request:

``` console
$ curl --resolve yourdomain.com:443:127.0.0.1 https://yourdomain.com/
```

Alternativly you can do the same thing on port 80 without https:

``` console
$ curl --resolve yourdomain.com:80:127.0.0.1 http://yourdomain.com/
```
