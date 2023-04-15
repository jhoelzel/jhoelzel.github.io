---
layout: post
title: Simplifying Kubernetes Developement for the Desktop with Rancher Desktop
subtitle: and Rising Above Docker's Shortcomings 
categories: kubernetes
tags: [kubernetes, rancher-desktop, kubernetes-windows, automation]
---

In this article I would like to introduce rancher-desktop to those who dont know it yet. I have recently switched away from docke-desktop and am now a happy little camper with a lot of joy while wokring.

**TLDR**
> Rancher Desktop, is a must-have tool for professional Kubernetes engineers looking to simplify container management and Kubernetes configuration on the (Windows) Desktop, all while benefiting from a reliable, proven foundation and the expertise of a global leader in open source solutions. By embracing the open source community and overcoming Docker's shortcomings, Rancher Desktop is the ideal choice for today's Kubernetes professionals. With its versatility, ease of use, and commitment to open source, Rancher Desktop is my go-to tool for Kubernetes management.

[Rancher Desktop](https://rancherdesktop.io/), an open-source desktop application developed by SUSE, is available for Mac, Windows, and Linux, streamlining Kubernetes and container management right on your desktop. SUSE, a global leader in innovative, reliable, and enterprise-grade open source solutions, is renowned for their expertise in Linux, Kubernetes, and cloud infrastructure.

With the flexibility to choose your Kubernetes version and the ability to build, push, pull, and run container images using containerd or Moby (dockerd), Rancher Desktop is a versatile tool for professional Kubernetes engineers. 

**Say goodbye to the need for a registry for local development** — your built container images can be run by Kubernetes instantly. Shortly I will publish an article on how to use it, but I am locally developing all my kubernetes projects in a container using VS-Code Remote containers and am directly deploying them to my single node Computer.

## Why I Choose Rancher Desktop Over Docker?

I fully admit that im a Windows User and like to click shiny things. I have used Docker Desktop since its release and it has been a companion tool for me ever since. Truth be told, their kubernetes integration was horrible and I never really got a cluster to remain stable on my machine, especially when I hybernated it over night. I used to have a seperate cluster running in my Office but this year I gifted myself a current gen I9 for christmas and decided that having 32 Cores ought to be enough for my development needs. Another big factor for me still was the actual policy of Docker - the company not the tool.

Docker has faced challenges in adapting to the rise of open source initiatives, partly due to its focus on monetizing its own products. This approach has led some organizations to develop their own solutions, often surpassing Docker's offerings. Although they actively contributes to the open source community, it has prioritized its intellectual property and revenue generation. Docker's once synonymous relationship with containers has been hindered by its struggle to innovate and keep up with the latest developments in container initiatives.

Furthermore the deprecation of Dockershim in Kubernetes 1.20 and its removal in 1.22 has further diminished Docker's relevance in Kubernetes and thefore for me. In contrast, Rancher Desktop offers a Kubernetes-centric solution that leverages the power of open source projects while addressing Docker's shortcomings and neatly packaging them in a package.

## Why is migration almost necessary?

As of January 31, 2022, the licensing model of Docker Desktop has changed fundamentally. The prerequisite for using the free Docker Desktop version is that the deploying company has fewer than 250 employees and earns less than $10 million (approx. €9,093,200) per year. These conditions can be met by small companies. However, larger companies must now decide to upgrade to a paid subscription or to look for alternatives. This feels a lot like the pied piper.

Also as of this march Docker has sent an email to users with "organization" accounts on Docker Hub, **warning that their accounts and images will be deleted unless they upgrade to a paid team plan**. This has caused significant anxiety for open source maintainers who use the platform to host images for their communities. The cost of a team plan is $420 per year, which is a significant burden for many open source projects that receive little or no funding. Additionally, Docker's definition of what is allowable for their Open Source program was out of touch, ruling out anything other than spare-time projects or those wholly donated to an open-source foundation. This move by Docker highlights the funding problem in the open source community, and how companies can forget their roots once they start making significant revenue.

When Docker started to **limit the pulls of public open-source images** like Go, Prometheus, and NATS, many open source projects have already moved away from Docker Hub anyway. Docker eventually acknowledged their mistake and reversed their decision to sunset their Free Team plan. 

They recognized that their policy and communication were both flawed, and after listening to feedback from their community, they decided to make changes. But in my opinion the cat is already out of the bag and going back now feels a little bit like returning to an abusive ex.

Finally the containerization has matured immensly and there are many other players at play that can solve my issue competently.

### Moby: a modular toolkit for containerization

[Moby](https://mobyproject.org/) is an open-source project created by Docker with the aim of making software containerization faster and more straightforward and especially less political!
Moby provides a modular "Lego set" of toolkit components that can be assembled to create customized container-based systems. These components include container build tools, a container registry, orchestration tools, a runtime, and more.
It is intended for engineers, integrators, and enthusiasts looking to modify, hack, fix, experiment, invent, and build systems based on containers. It is not designed for people seeking a commercially supported system, but rather for those who want to work and learn with open-source code or are themselves open like Rancher-Destop.

This of course sounds a lot like docker, but here me out:

#### Principles
Moby follows strong principles that prioritize modularity, flexibility, and an open-minded approach to user experience. 
It is open to community contributions and follows the following guidelines:

- Modular: the project includes lots of components that have well-defined functions and APIs that work together.
- Batteries included but swappable: Moby includes enough components to build fully featured container systems, but its modular architecture ensures that most of the components can be swapped by different implementations.
- Usable security: Moby provides secure defaults without compromising usability.
- Developer focused: The APIs are intended to be functional and useful to build powerful tools.

#### Relationship with Docker

Moby is ***made up of the open-source components that Docker and the community have built*** for the Docker Project. Docker is committed to using Moby as the upstream for the Docker Product, but other projects are also encouraged to use Moby as an upstream and to repurpose its components. Moby welcomes external maintainers and contributors.

Now this sounds great but what was still missing was an interface for me to click shiny buttons on. Therefore I am really glad I found Rancher Desktop.

## All open source means zero vendor lock-in

Zero lock-in is ensured by relying solely on 100% open source components, such as Moby, containerd, k3s, kubectl, and other trusted projects within the cloud-native community. This approach prevents any lock-in into a proprietary stack, ensuring flexibility and freedom of choice.

## Container Management Made Easy

Rancher Desktop caters to your preferred container engine, whether it's Moby/dockerd, which utilizes the Docker CLI, or containerd, which employs nerdctl—a Docker-compatible CLI provided by the containerd project. Built on the solid foundation of Electron, Rancher Desktop simplifies user experience by wrapping other tools and leveraging a virtual machine on MacOS and Linux, or Windows Subsystem for Linux v2 on Windows systems. All you need is to download and run the application.

Since Rancher Desktop comes equipped with k3s, a lightweight certified Kubernetes distribution. With just a few clicks, you can choose your desired Kubernetes version, reset Kubernetes, or even reset Kubernetes and the entire container runtime.

## my Dev Container has not changed but my processes improved

With Moby and Rancher desktop I can still simply mount my docker.sock into my [dev container](https://github.com/jhoelzel/devcontainer/tree/master/golang/k8s-go-client%20helm%20kubectl) and work for all my clients in isolation and "dependecy-fludidity".

``` json
"mounts": [
	"source=/var/run/docker.sock,target=/var/run/docker.sock,type=bind",
],
```

and mount the kubeconfig on your computer. (or copy it by hand whichever your prefer)

``` json
"runArgs": [
	//kube keys 
	 "-v",
	"${env:HOME}${env:USERPROFILE}/.kube:/home/vscode/.kube:ro"
]
```
This enables me to build container images with separated sources and also legally empowers me to separate everything. With Gitops setup for each client I can simply reset my local kubernetes cluster to factory settings, mount my kubeconfig and off I am. ( I admit that a computer with NVME, DDR5 and I9 makes this much more pleasureable.).

As I have mentioned before, the integrated container registry also empowers me to not have to upload all my images to a hub and then redownload them again simply because I have to. Which saves time, disk space and overall nerves.

## It includes a dashboard for kubernetes

More veteraned readers might know of a little tool called Lens-Desktop by Mirantis. When kubernetes-lens came out it was the greatest software to see what is going on in a clusters and I have suggested it to basically everyone working with k8s saying its a must have tool and also made many teams install it. It used to be free and open but now requires user accounts and is monitizing in the same way the docker corp is. [You can find more about the issue here](https://github.com/lensapp/lens/issues/5444)

Therfore I am very happy to see SuSe integrate a simply dashboard into the cluster directly. It is the same Dashboard integrated into Rancher itself but super usefull for local development! And if you like to have more, you can simply run Rancher itself in your local cluster to provision external ones too. This is great for bare-metal deployments and more!

[Rancher, the big one](https://www.rancher.com/), is a complete software stack for teams adopting containers. It addresses the operational and security challenges of managing multiple Kubernetes clusters across any infrastructure, while providing DevOps teams with integrated tools for running containerized workloads. 

## it's k3s!!!

As mentioned before, Rancher-Desktop comes with my favorite kubernetes distribution, which is also as you might have guessed Rancher Labs powered: [K3s](k3s.io). 
k3s is a lightweight Kubernetes distribution. It was designed to be easy to install, run, and maintain on any infrastructure, including bare-metal servers, virtual machines, and the cloud. In September 2020, k3s became a Cloud Native Computing Foundation (CNCF) project, which means ***it must pass the same software conformance tests that other CNCF-certified distributions for Kubernetes*** must pass. This ensures that configurations built for Kubernetes will work with k3s.

You can also find out more about it in my recent post [Comparing k3s with Kubernetes How k3s is Often the Better Choice](https://www.hoelzel.it/kubernetes/2023/04/01/k3s-is-Often-the-Better-Choice.html). While reading it, you can keep in mind that your local and your cloud cluster can now aboslutetly run on the excat same stack. Well, sure there is some WSL involved but once you overlook the tiny performance hit, that is to be expected for a dev env, and really inevideable on windows anyway. 

## German quality with a long positive history

As a german myself I am also very happy to see SuSe succed in this space! S.u.S.E. actually is an acronym for "Software- und System-Entwicklung" which translates to "Software and System Development" and has a long stance of being open and very developer friendly. Of course it is by now an international company with many customers worldwide but it has always stood for the principle of "here take this its free" and "if you need help though, we offer paid support for everything". Which is, in my honest opinion, how Docker itself should have handled things.

SUSE's involvement in the Kubernetes ecosystem goes beyond its products and services. The company actively contributes to the Kubernetes open-source community, participating in the development of new features and enhancements, and collaborating with other industry players to promote the adoption and advancement of containerization and Kubernetes technologies.

As Americans you might love them too because they [keep your country safe](https://ranchergovernment.com/)

And yeah, thats why I always have put my trust in them.

