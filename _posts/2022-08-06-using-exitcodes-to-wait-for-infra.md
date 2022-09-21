---
layout: post
title: Using exitcodes to wait for infra to boot
subtitle: or how to wait until a host ist pingable
categories: cli
tags: [cli, linux]
---

I am working with a lot infrastructure and often need to execute commands after on of my servers comes online or in other cases one script that manipulates another server, needs to wait for it to reboot. 

This is especially the case when you are "terraforming" or "ansibleling" bare metal clusters and would like the your backup cluster to provision itself completely.

The easiest way to achieve this, is to make use of the return code of the ping command.

## What are return codes

Return codes or also called error codes are codes that are used to determine the nature of an error that happened in the execution of a program and why it occurred. Basically you can use them to figure out what went wrong, but also that nothing went wrong at all.

You may find a list of common return codes for nix based systems here <https://slg.ddnss.de/list-of-common-exit-codes-for-gnu-linux/>, but in general, you can think of them as programs returning integers that tell you "in short" what went wrong and you can write your own by simply using the **return** Keyword in your code as well.


## Return codes as variables

You can retrieve them, after code has been executed. In this example after grep has run:

``` Bash
grep -q -w "mystring" "${FILE}"
# this will save the return code of our program into the grepreturncode variable:
grepreturncode=$?
```

Now this seems trivial, but with the codes saved as variables you can easily compare them and take precise actions upon the execution of other programs:

``` Bash
if test $grepreturncode -eq 0
then
	echo "mystring found in $FILE file."
    # take action
else
	echo "mystring not found in $FILE file."
    # take action
fi
``` 

While this is awesome, we can even go one level deeper and write checks as to wether our return code **changes**.

## Rudimentarily monitoring changes to return codes

Lets tweak our code above and wait **UNTIL** mystring found in $FILE.
We can do that with a simple while loop:

``` BASH
#!/bin/bash
# we dont want our first loop to succeed directly, so we declare the variable
grepreturncode=1
while [ $grepreturncode -neq 0 ]; do
    # lets execute our check
    grep -q -w "mystring" "${FILE}"
    # and save it into our variable
    grepreturncode=$?
done
```

I would like to point out, while this works, it is a hacky method to achieve our results. This will run until our string is found and as often as it can. Meaning it will try again almost directly after our grep command has been run. If you expect your status to change minutes later, you might want to introduce a sleep statement in between:

``` BASH
#!/bin/bash
# we dont want our first loop to succeed directly, so we declare the variable
grepreturncode=1
while [ $grepreturncode -neq 0 ]; do
    # lets execute our check
    grep -q -w "mystring" "${FILE}"
    # and save it into our variable
    grepreturncode=$?
    # lets wait for a minute before we check again
    sleep 1m
done
```

Much better, but still hacky. Remember, if our string is added after 1 minute and 1 second, our check is still going to wait for two minutes.
But it is a cheap and easy way to wait for changes.

## Waiting for a host to become online

There are multiple tools to check if a host is online but one of the simplest and know ones, is ping.
By definition ping checks if a host is available:

``` Text
ping is a computer network administration software utility used to test the reachability of a host on an Internet Protocol (IP) network.
```
<https://en.wikipedia.org/wiki/Ping_(networking_utility)>

Now, while I need to point out, that reachability does not equal operability, at least we can be certain, that the machine is most likely able to receive connections.

So let us adapt our sample to use ping instead of grep:

``` BASH
#!/bin/bash
# we dont want our first loop to succeed directly, so we declare the variable
pingreturncode=1
while [ $pingreturncode -neq 0 ]; do
    # lets execute our check
    ping "${HOST}"
    # and save it into our variable
    pingreturncode=$?
    # lets wait for a minute before we check again
    sleep 1m
done
# do something
```

And there we go. If our host is reachable (the return code of the ping command does equal 0 for success), it will execute the code.
Otherwise it will loop until the end of time in one minute steps ;)
