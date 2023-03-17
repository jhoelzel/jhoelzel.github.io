---
layout: post
title: Only allowing cloudflare servers ips for your server
subtitle: and to secure your server from other threats.
categories: hints
tags: 
    - cloudflare
    - IPV4
    - IPV6
---

The chances are high that if you are using cloudflare to proxy your servers, that you will not have much traffic on them that is not originating from cloudflare.
In order to secure your server even further, you could block any external IP Adress that is not owned by cloudflare, or your own VPN.

This proves to be an effective way to block DOS that can be caught by cloudflare, which lets face it, will be better at it as if we would be able to ourselfs.

## Find all of cloudflare IP-Ranges

Lukily, cloudflare has got your covered! Please check out <https://www.cloudflare.com/ips> and you will find a nice sorted list of the IP Ranges that cloudflare uses.
This brings us the advantage to use all of them in our server configuration, but cloudflare even goes one step further and provides them for you as txt files for easy querying.

IPV4: <https://www.cloudflare.com/ips-v4>
IPV6: <https://www.cloudflare.com/ips-v6>

## Add them to your iptables

Now all we need to to is add them to our IP tables:

``` console
$ for i in `curl https://www.cloudflare.com/ips-v4`; do iptables -I INPUT -p tcp -m multiport --dports http,https -s $i -j ACCEPT; done
$ for i in `curl https://www.cloudflare.com/ips-v6`; do ip6tables -I INPUT -p tcp -m multiport --dports http,https -s $i -j ACCEPT; done
```

## Do it for your own VPN IPs

IPV4:

``` console
$ iptables -I INPUT -p tcp -m multiport --dports http,https -s <YOUR.VPN.IP.v5> -j ACCEPT;
```

IPV6:

``` console
$ ip6tables -I INPUT -p tcp -m multiport --dports http,https -s <YOUR.VPN.IP.v6> -j ACCEPT;
```

## Drop other sources of traffic

After we allowed cloudflare accces to our servers, we still need to block any other incoming package:

``` console
$ iptables -A INPUT -p tcp --dport http,https -j DROP
```

and the same is true for IPV6

``` console
$ ip6tables -A INPUT -p tcp --dport http,https -j DROP
```

Great now we are using cloudflare to its fullest and are blocking every other traffic that might reach our servers. It is however important to know, that now you will not be able to reach your server without cloudflare. Just in case I am going to show you a little bit of firefighting, should cloudflare decided to drop traffic to your servers.

## Rescue your system and allow traffic from other IPs again

So it happened, cloudflare decided that you validate their terms and have dropped you. Yikes. No matter the reason behind it, your servers will now no longer be reachable from the internet besides through your VPNs.
In order to stop this from happening, we need to drop the rules we have put up before:

``` console
$ iptables -A INPUT -p tcp -m multiport --dports http,https -j ACCEPT
```

and the same is true for IPV6

``` console
$ ip6tables -A INPUT -p tcp -m multiport --dports http,https -j ACCEPT
```

and bam. Your servers should be back in business.

### Sources

<https://www.cloudflare.com/ips>

<https://support.cloudflare.com/hc/en-us/articles/200169166-How-do-I-whitelist-CloudFlare-s-IP-addresses-in-iptables->
