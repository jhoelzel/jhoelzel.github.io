---
layout: post
title: Resetting time in a docker container on Windows
subtitle: or how hibernation sometimes frustrates me.
categories: hints
tags: [wsl, time-problems]
---

Often after hibernation your docker containers will be out of sync, which will create problems with everything expiration based.
I experienced this problem, after my laptop came out of hibernation and I had just setup a fresh kubernetes cluster, which, because of its cert-based auth, showed that my certificate was set to the future and therefore I was not able to authenticate myself.

This quick fix will update the time in your local docker desktop container which should be using wsl and Windows.
It is using hwclock:

```
hwclock is a tool for accessing the Hardware Clock. You can display the current time, set the Hardware Clock to a specified time, set the Hardware Clock to the System Time, and set the System Time from the Hardware Clock.
```

You can find more information about hwclock here: <https://linux.die.net/man/8/hwclock>

In order to fix your container time, which inherited by the doker-desktop container simply run this in powershell:

``` powershell
$ wsl -d docker-desktop -e /sbin/hwclock -s
```

After that your system time should be updated and on time again.
