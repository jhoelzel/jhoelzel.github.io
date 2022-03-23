---
layout: post
title: Redeploying an Argo-CD Autopilot project
subtitle: or first you break it, then you have to fix it.
categories: argocd
tags: [argocd, argocd-autopilot, kubernetes]
---

## What is Argo-CD Autopilot

> The Argo-CD Autopilot is a tool which offers an opinionated way of installing Argo-CD and managing GitOps repositories.

If you use argo yourself, you probably have an idea or two on how to structure your repositories for maximum effect. But for new projects I like to have my manifests generated. 
When you start from scratch, the Argo-CD Autopilot provides a nice and easy way to provision multiple applications and more, but that's beyond the scope of this article.
If you want to learn more, simply visit them on Github: 

<https://github.com/argoproj-labs/argocd-autopilot/>


## Redeploying an existing Argo-CD project

So all of this sounds sweet, but sooner or later you will make a mistake, or a tornado takes out your servers and you need to redeploy your already bootstrapped repository.
In high hopes you run the bootstrap command again:

```
$ argocd-autopilot repo bootstrap 
```

and we wait in excitement.... But what's that? We are greeted with an error:

```
#INFO cloning repo: https://github.com/jhoelzel/argocd-simpleapp.git
Enumerating objects: 29, done.
Counting objects: 100% (29/29), done.
Compressing objects: 100% (25/25), done.
Total 29 (delta 4), reused 25 (delta 1), pack-reused 0
INFO using revision: "", installation path: ""    
FATAL folder bootstrap already exist in: /bootstrap 
```

Great. Of course the repository exists. We created it together autopilot don't you remember?
It does not.

So in order to get your cluster up and running again you will have to repeat some steps that it does for you when you first bootstrap the cluster:

### Clone the repository

If you have not already done so, clone your repository to a local folder.

```
$ git clone git@github.com:jhoelzel/argocd-simpleapp.git
$ cd argocd-simpleapp
```

### Bootstrap ArgoCD again

First we start with installing Argo itself, after that we redeploy our root.yaml application which will pick up on our existing manifests:

```
$ kustomize build bootstrap/argo-cd | kubectl apply -f -
$ kubectl apply -f bootstrap/argo-cd.yaml
$ kubectl apply -f bootstrap/root.yaml
```

And presto. Everything is working or is it?
Nope, we first need to recreate our Git secret in order for argo to have access to our repository too:

### Recreate the autopilot secret from cli

When you bootstrapped your cluster the last time, you where providing the autopilot with a github token and your associated username.
In order for argo to get access to it again, we need to recreate it:

```
$ kubectl -n argocd create secret generic autopilot-secret --from-literal git_username=<your_username> --from-literal git-token=<your_token>
```

And voila, our cluster is provisioning itself from our repository again.
Feels good to be home.


### Get the new admin password from the initial secret

Finally we would like to watch the rollout of our manifests and maybe even discover errors that we didn't think of before.
Therefore we need the login password to the new argocd initial user (if you have not defined SAML or other in your repository).

Argo-CD saves it for us in a secret called "argocd-initial-admin-secret", so all we finally have to do is retrieve it:

```
$ kubectl get secret argocd-initial-admin-secret -n argocd -ogo-template='{{printf "%s\n" (index (index . "data") "password" | base64decode)}}'
```

Enjoy your old cluster back the way you left it.


### Sources

<https://github.com/argoproj-labs/argocd-autopilot/>

<https://github.com/argoproj-labs/argocd-autopilot/issues/26>

