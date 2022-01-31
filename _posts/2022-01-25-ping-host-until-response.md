---
layout: post
title: Ping a hust until he responds
subtitle: Nifty for terraform or other provisioning scripts
categories: hints
tags: [ping, cli]
---

When you are playing with terraform or any other provising software for a while, you will most likely run into an issue rather sooner than later, where you will have to wait for a host to reboot.
Sleeping during your execution steps is one thing, but what if there was a better way?

There actually is and it is diretly integrated into ping: <https://linux.die.net/man/8/ping>

## ping -c

The -c option of ping

``` 
-c count
Stop after sending count ECHO_REQUEST packets. With deadline option, ping waits for count ECHO_REPLY packets, until the timeout expires.
```

great that means we can use

``` console
$ ping <target> -c 1
```

and be done with it? Right?

Well its not so easy because ping, as any other looping sotware, can be used to flood your target.
Since some services will see this a Denial of Service attack, there is agreat chance that you will be banned from pinging the server again, because the network provider decided your flooding the server.

Lukily there is another flag that we can make use of that will help us here:

## ping -i

The -i option of ping stands for intervall and here is what it does:

``` 
-i interval
Wait interval seconds between sending each packet. The default is to wait for one second between each packet normally, or not to wait in flood mode. Only super-user may set interval to values less 0.2 seconds.
```

Awesome! that means we can use it in combination of -c and stop the flooding of the server and only ping our target every five seconds:

``` console
$ ping <target> -c 1 -i 5
```

Nice.

But are we done yet?

## ping -w

In theory yes, but what is if your target never does boot up or ping is disabled? Our script will simply wait for ever and never finish.
Therefore we need to do something about this and lukily ping has us covered again:

``` 
-w deadline
Specify a timeout, in seconds, before ping exits regardless of how many packets have been sent or received. In this case ping does not stop after count packet are sent, it waits either for deadline expire or until count probes are answered or for some error notification from network.
```

which turn our command into:

``` console
$ ping <target> -c 1 -i 5 -w 120
```

This means that our ping command will terminate itself after 120 seconds, if the host could not have been reached by then.

Et voila! Our ping command will run for the specified time in a specified intervall or until it will process one succesfull package.
Happy pinging!
