---
layout: post
title: Update the timezone in a running alpine container
subtitle: because time is of the essence!
categories: helm
tags: [alpine, docker, container]
---

If you are like me you are often facing the problem that the dev container you are currently using is using a timezone different to your own.
Certificate creation is amongst the best examples for a frustrating job das will fail if you are not prepared. So lets update or TZ

Note: This only for works for  NON uclibc installs!

## download the alpine tz package

First we need to install "tzdata" from the alpine package repository:

``` Console
$ sudo apk add tzdata
```

or if you are already running as root (which you should not)

``` Console
$ apk add tzdata
```


## list all available timezones

``` Console
$ ls /usr/share/zoneinfo
------------------------------------------------------------------
total 336
drwxr-xr-x   19 root     root          4096 Jul  5 18:44 .
drwxr-xr-x    1 root     root          4096 Jul  5 18:44 ..
drwxr-xr-x    2 root     root          4096 Jul  5 18:44 Africa
drwxr-xr-x    6 root     root         20480 Jul  5 18:44 America
drwxr-xr-x    2 root     root          4096 Jul  5 18:44 Antarctica
drwxr-xr-x    2 root     root          4096 Jul  5 18:44 Arctic
drwxr-xr-x    2 root     root         12288 Jul  5 18:44 Asia
drwxr-xr-x    2 root     root          4096 Jul  5 18:44 Atlantic
drwxr-xr-x    2 root     root          4096 Jul  5 18:44 Australia
drwxr-xr-x    2 root     root          4096 Jul  5 18:44 Brazil
-rw-r--r--    1 root     root          2094 Mar 17 14:47 CET
-rw-r--r--    1 root     root          2310 Mar 17 14:47 CST6CDT
drwxr-xr-x    2 root     root          4096 Jul  5 18:44 Canada
drwxr-xr-x    2 root     root          4096 Jul  5 18:44 Chile
-rw-r--r--    2 root     root          2416 Mar 17 14:47 Cuba
-rw-r--r--    1 root     root          1908 Mar 17 14:47 EET
-rw-r--r--    1 root     root           114 Mar 17 14:47 EST
-rw-r--r--    1 root     root          2310 Mar 17 14:47 EST5EDT
-rw-r--r--    2 root     root          1955 Mar 17 14:47 Egypt
-rw-r--r--    2 root     root          3492 Mar 17 14:47 Eire
drwxr-xr-x    2 root     root          4096 Jul  5 18:44 Etc
drwxr-xr-x    2 root     root          4096 Jul  5 18:44 Europe
-rw-r--r--    1 root     root           116 Mar 17 14:47 Factory
-rw-r--r--    7 root     root          3648 Mar 17 14:47 GB
-rw-r--r--    7 root     root          3648 Mar 17 14:47 GB-Eire
-rw-r--r--   10 root     root           114 Mar 17 14:47 GMT
-rw-r--r--   10 root     root           114 Mar 17 14:47 GMT+0
-rw-r--r--   10 root     root           114 Mar 17 14:47 GMT-0
-rw-r--r--   10 root     root           114 Mar 17 14:47 GMT0
-rw-r--r--   10 root     root           114 Mar 17 14:47 Greenwich
-rw-r--r--    1 root     root           115 Mar 17 14:47 HST
-rw-r--r--    2 root     root          1203 Mar 17 14:47 Hongkong
-rw-r--r--    2 root     root          1162 Mar 17 14:47 Iceland
drwxr-xr-x    2 root     root          4096 Jul  5 18:44 Indian
-rw-r--r--    2 root     root          2582 Mar 17 14:47 Iran
-rw-r--r--    3 root     root          2388 Mar 17 14:47 Israel
-rw-r--r--    2 root     root           482 Mar 17 14:47 Jamaica
-rw-r--r--    2 root     root           309 Mar 17 14:47 Japan
-rw-r--r--    2 root     root           316 Mar 17 14:47 Kwajalein
-rw-r--r--    2 root     root           625 Mar 17 14:47 Libya
-rw-r--r--    1 root     root          2094 Mar 17 14:47 MET
-rw-r--r--    1 root     root           114 Mar 17 14:47 MST
-rw-r--r--    1 root     root          2310 Mar 17 14:47 MST7MDT
drwxr-xr-x    2 root     root          4096 Jul  5 18:44 Mexico
-rw-r--r--    4 root     root          2437 Mar 17 14:47 NZ
-rw-r--r--    2 root     root          2068 Mar 17 14:47 NZ-CHAT
-rw-r--r--    4 root     root          2444 Mar 17 14:47 Navajo
-rw-r--r--    5 root     root           561 Mar 17 14:47 PRC
-rw-r--r--    1 root     root          2310 Mar 17 14:47 PST8PDT
drwxr-xr-x    2 root     root          4096 Jul  5 18:44 Pacific
-rw-r--r--    2 root     root          2654 Mar 17 14:47 Poland
-rw-r--r--    2 root     root          3497 Mar 17 14:47 Portugal
-rw-r--r--    2 root     root           761 Mar 17 14:47 ROC
-rw-r--r--    2 root     root           617 Mar 17 14:47 ROK
-rw-r--r--    2 root     root           383 Mar 17 14:47 Singapore
-rw-r--r--    3 root     root          1947 Mar 17 14:47 Turkey
-rw-r--r--    8 root     root           114 Mar 17 14:47 UCT
drwxr-xr-x    2 root     root          4096 Jul  5 18:44 US
-rw-r--r--    8 root     root           114 Mar 17 14:47 UTC
-rw-r--r--    8 root     root           114 Mar 17 14:47 Universal
-rw-r--r--    2 root     root          1535 Mar 17 14:47 W-SU
-rw-r--r--    1 root     root          1905 Mar 17 14:47 WET
-rw-r--r--    8 root     root           114 Mar 17 14:47 Zulu
-r--r--r--    1 root     root          4463 Mar 17 14:47 iso3166.tab
-rw-r--r--    3 root     root          3536 Mar 17 14:47 posixrules
drwxr-xr-x   18 root     root          4096 Jul  5 18:44 right
-r--r--r--    1 root     root         19419 Mar 17 14:47 zone.tab
-r--r--r--    1 root     root         17593 Mar 17 14:47 zone1970.tab    
```

as you can see there are many to choose from but mine will be Europe/Berlin

## copy timezone into localtime:

Lets copy the selected timezone into localtime

``` Console
$ sudo cp /usr/share/zoneinfo/Europe/Berlin /etc/localtime
```

or as root

``` Console
$ cp /usr/share/zoneinfo/Europe/Berlin /etc/localtime
```

and finally we need to set our timezone locally:

``` Console
$ sudo echo "Europe/Berlin" >  /etc/timezone
```

or as root:

``` Console
$ apk echo "Europe/Berlin" >  /etc/timezone
```

## check your results and remove the package

In order to check if everything worked, check the current date with date:

``` Console
$ workspace date
Tue Jul  5 18:55:50 CEST 2022   
```

and finally remove tz from your system

``` Console
$ sudo apk del tzdata
```

or as root:

``` Console
$ apk add tzdata
```
