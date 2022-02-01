---
layout: post
title: Getting the ipv4 and ipv6 of your traefik load balancer
subtitle: and how to use the cname of external-dns for ipv6 traffic.
categories: terraform
tags: [kubernetes, traefik, kubernetes-load-balancer, terraform, external-dns]
---

I was facing a problem in a kubernetes cluster recently where everything was setup using external-dns, cloudflare and traefik. The Issue I was facing was is that by default external-dns only assigns ipv4 addresses to your ingresses and therefore makes ipv6 routing hard.
After quite some research I found, that the default external-dns is actively working on ipv6 support but it is just not there yet.

I decided that waiting for a feature release would have been the wrong option for me, but also that creating my load balancer dynamically through kubernetes itself, with the help of the cluster control manager is my best option.
Therefore a direct output of the ip-addresses, no matter if ipv4 or ipv6 would not have been possible in one run.

## Getting the ipv4 and ipv6 from kubernetes after provisioning

I install traefik with a helm chart and therefore I know for certain that its load balancer, for example, is deployed in my "kube-system" namespace with the service name traefik.
So all I really need to do is to set up my cluster, wait for it to be ready and then get my ip-addresses.

In oder to do so you can use kubectl quite easily:

``` Console
$ kubectl get service traefik -n kube-system --no-headers --output custom-columns="ADDRESS:.status.loadBalancer.ingress[0].ip"  > ./lbipv4.addr.txt
$ kubectl get service traefik -n kube-system --no-headers --output custom-columns="ADDRESS:.status.loadBalancer.ingress[1].ip"  > ./lbipv6.addr.txt   
```

Which will save them in txt files in your current directory.
So great. Now you have 2 txt files with IP-Addresses in them. Well done, but still quite useless. so lets implement it all with terraform.

## Getting the same IPs using Terraform

The first thing we are going to do, is to convert our little script into a standard terraform null-resource. I have chosen this approach becaus it allows me to have my local-exec "depend" on other steps in my pipeline.
This makes an elegant approach where terraform tells me "hey I am done provisioning the cluster and will now start configuring it".

**Please not that I have downloaded the kube-config from the cluster beforehand and have put it in a file called ./kubconfig.yaml**

``` main.tf
resource "null_resource" "lbdns" {
  # write our ips to file
  provisioner "local-exec" {
    command = <<-EOT
     kubectl get service traefik -n kube-system --no-headers --output custom-columns="ADDRESS:.status.loadBalancer.ingress[0].ip" --kubeconfig ./kubeconfig.yaml > ./lbipv4.addr.txt 
     kubectl get service traefik -n kube-system --no-headers --output custom-columns="ADDRESS:.status.loadBalancer.ingress[1].ip" --kubeconfig ./kubeconfig.yaml > ./lbipv6.addr.txt    
    EOT
  }

#  depends_on = [
#   Your Node needs to be ready so its a good idea to depend on it here!!
#  ]
}
```

Great! Now we only need something to actually pick up these addresses. We will use terraforms data provider, because it's a really convenient way for us to read the files without having to worry about it:

```  main.tf
data "local_file" "external-lb-ip-ipv4" {
  filename = "${path.module}/lbipv4.addr.txt"
  depends_on = [
    null_resource.lbdns,
  ]
}
data "local_file" "external-lb-ip-ipv6" {
  filename = "${path.module}/lbipv6.addr.txt"
  depends_on = [
    null_resource.lbdns,
  ]
}
```

And now what we need to do is to use our ip addresses in a productive way. In my case that means using it with the terraform provider for cloudflare.

```  main.tf
resource "cloudflare_record" "lb" {
  zone_id = var.cloudflare_zone
  name    = "lb-${var.cluster_fqdn}"
  value   = trim(data.local_file.external-lb-ip-ipv4.content,"\n")
  type    = "A"
  proxied = true
  depends_on = [
    null_resource.lbdns,
  ]
}
resource "cloudflare_record" "lbv6" {
  zone_id = var.cloudflare_zone
  name    = "lb-${var.cluster_fqdn}"
  value   = trim(data.local_file.external-lb-ip-ipv6.content,"\n")
  type    = "AAAA"
  proxied = true
  depends_on = [
    null_resource.lbdns,
  ]
}
```

So what have we done so far?

- after our cluster has been created
- and traefik has been installed
- we retrieved the ipv4 and the ipv6 IP
- and assigned it to dns entry for our domain using cloudflare (everything else will work just as well)

So essentially what we end up with is a configured dns entry for our load balancer.
Thats not what we wanted thought right?

Well it is actually what we need to provide ipv4 and ipv6 ingress for kubernetes external-dns.
I am using traefik here but anything else with an ingress or service would work just as well.

## Defining an ingress with external-dns

This is how our ingress would normally look like. we simply define the hostname and external-dns will do the rest.

```  YAML
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
  annotations:
    kubernetes.io/ingress.class: traefik
    external-dns.alpha.kubernetes.io/hostname: app.domain.tld
spec:
  rules:
    - host: app.domain.tld
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: my-svc
                port:
                  number: 80
```

What is going to happen now, is that external-dns will only assign the ipv4 entry to your domain. That is really suboptimal because everybody with an ipv6 address will not even receive a DNS record but simply see the site as offline directly.

## Defining an ingress using the previously defined load balancer dns-entry

Luckily there is a work around for us. Usually external-dns will set A records for us automatically, but there is way we can use something else instead. A CName.
by using the **external-dns.alpha.kubernetes.io/target** annotation we can set an ip for our record directly. But if we actually pass a tld, external-dns will add the record to your domain as a CName record and thus provide our services with the valuable ipv6 address:

```  YAML
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
  annotations:
    kubernetes.io/ingress.class: traefik
    external-dns.alpha.kubernetes.io/hostname: app.domain.tld
    external-dns.alpha.kubernetes.io/target: lb-${var.cluster_fqdn}
spec:
  rules:
    - host: app.domain.tld
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: my-svc
                port:
                  number: 80
```

And thats it. Our ingress should now return IPs for both IPv4 and IPv6 and even our ipv6-only customers should be happy with the results.

### Sources

<https://github.com/kubernetes-sigs/external-dns>

<https://registry.terraform.io/providers/cloudflare/cloudflare/latest/docs>

<https://en.wikipedia.org/wiki/CNAME_record>
