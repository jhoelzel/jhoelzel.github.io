---
layout: post
title: The perfect azure development container
subtitle: including alpine, azure-cli, terraform, golang, kubectl, helm and all your friends.
categories: hints
tags: [alpine, azure-cli, kubernetes, terraform, golang, kubectl, helm]s
---

While working with azure I think I might have found my perfect setup for azure.
As a Windows user with a trusty surface book, a WSL based setup is what I used for many years and at times I have simply rented a VPS when using working with clients.
WSL is really cool but using it as a freelancer has one mayor drawback: if I want to switch between versions of software easily there is no uncomplicated way to do that.
Yes yes, there is software that would configure my dev environment depending on setup scripts or enable me to switch context depending on simple commands but there is a much easier way: Creating your own development environment in a reusable docker container with the help of Visual Studio code.

here is a short overview of what we are going to do in this article:

- using docker for workload isolation
- using docker in combination with visual studio code
  - remote explorer extension
  - attaching visual studio to running container
  - defining your workspace in a running container
- defining what we need for an azure development environment
  - What do we want to build
  - What tools to we need
    - golang:alpine
    - terraform
    - kubectl
    - helm
    - azure cli
    - oh my zsh
    - git
  - defining our workspace
- building the docker container
- defining the dev container variables
  - defining visual studio code plugins
  - defining workspace settings
  - linking local configuration files into the container
- using the container
  - starting and stopping with vs code
  - starting and stopping manually
  - adjusting your time
  - rebuilding your container

## using docker for workload isolation

Using docker on your desktop is of course nothing new. Ever since its release it has not been easier to run a diverse software stack on your host system.
One of its nicer features is Volume mounting. Which means you can link one of the folders on your host system directly into a docker container and use is as if it would reside in the same container. <https://docs.docker.com/storage/volumes/>

A simple example of everyday use would be the mounting of a folder directly into an nginx deployment:

``` Console
 docker run -d \
  --name devtest \
  --mount source=./html,target=/app \
  nginx:latest
```

other srsadasdasd##########################################################################
https://medium.com/@markmcwhirter/alternative-acme-via-cert-manager-a9e9e7f105e0
https://docs.bitnami.com/tutorials/create-openldap-server-kubernetes/