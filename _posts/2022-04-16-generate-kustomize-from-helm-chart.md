---
layout: post
title: Generating kustomize overlays from helm charts
subtitle: a simple script that will make you happy
categories: helm
tags: [helm, kustomize, kubernetes]
---

Today we are converting a helm chart to an actual kustomize deployment.

## using Kustomize built-in plugin

Since Kustomize v4.1.0 you can simply render helm charts by using the integration provided by kustomize.
So Kustomize can be use Helm charts for resource generation, right out of the box:

So let's do that. In this example I am working with a simply minecraft deployment, but of course any other helm chart will do.
This is what your Kustomize file could look like:

``` YAML
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
namespace: default
helmCharts:
- name: minecraft
  includeCRDs: false
  valuesInline:
    minecraftServer:
      eula: true
      difficulty: hard
      rcon:
        enabled: true
  releaseName: moria
  version: 3.1.3
  repo: https://itzg.github.io/minecraft-server-charts
```


``` Console
$ kustomize build . --enable-helm
```

And of course if you don't want to simply look at it you can save it directly to file:

``` Console
$ kustomize build . --enable-helm >FileName.yaml
```

What you will end up with us one big YAML file, that you can modify as you please.

## Using Helm v3 Post Rendering

Helm v3 has yet another approach to allow you to modify your helm deployments after yaml generation. The Post-Render hook:

"Post rendering gives chart installers the ability to manually manipulate, configure, and/or validate rendered manifests before they are installed by Helm."
<https://helm.sh/docs/topics/advanced/#post-rendering>

Post rendering can be used with helm- install, upgrade, and template, simply by appending -post-renderer to your command and a common example looks like this:

``` Console
$ helm install the-chart kubecost/cost-analyzer --post-renderer ./path/to/executable
```

Wait what? "/path/to/executable"? Really?.

Yes what helm would like us to apply is yet another tool, that we provide ourselves.
The documentation looks clear and the example is simple:

``` Go
package main

import (
    "log"
    "os"

    "helm.sh/helm/v3/pkg/action"
    "helm.sh/helm/v3/pkg/cli"
)

func main() {
    settings := cli.New()

    actionConfig := new(action.Configuration)
    // You can pass an empty string instead of settings.Namespace() to list all ns
    if err := actionConfig.Init(settings.RESTClientGetter(), settings.Namespace(), os.Getenv("HELM_DRIVER"), log.Printf); err != nil {
        log.Printf("%+v", err)
        os.Exit(1)
    }

    client := action.NewList(actionConfig)
    // Only list deployed
    client.Deployed = true
    results, err := client.Run()
    if err != nil {
        log.Printf("%+v", err)
        os.Exit(1)
    }

    for _, rel := range results {
        log.Printf("%+v", rel)
    }
}
```

There is much more to configure, including storage backends and a lot more. The question if you are going to like, use or want this approach is another topic.
You can read more here <https://helm.sh/docs/topics/advanced/#post-rendering>. It is impressive and if that is your cup of tea, enjoy!

I really like Helm and its charts, but this does not really feel like an approach I would like to implement into my piepelines and I also do not like the monster yaml approach taken by the kustomize integrated plugin. So what can I do?

## Using helm and a script to create a kustomize deployment at a specific version

The little script below will do it all for you. Using helm, it will render a chart at a specific version with specific vaules for you into *separate* files which you can easily kustomize along the way. Not only that, but since we are using kustomize to deploy things later, we can have all our changes applied and modify them if we like too.

The general structure of this is simple:

/myKustomizeDeployment
    /helm
        /tmp
        /source
    /values
    /base
        /chart-name/
    kustomize.yaml


Lets us start with an empty folder and deploy the script below into it.
In the variables part you will find everything necessary to add your chart to your local helm stack, but simply not use helm afterwards at all. This makes this approach for instance perfect for pipelines, where the devops team generates the manifests, deploys them as kusztomize scripts into the pipeline and is happy using them.


``` bash

#!/bin/bash
 
######################################################################
##
##   Helm Chart to kustomize converter
##   Written By: Johannes Hölzel
##   Update on: April 16.2022
##
##   Requirements:
##    Helm
##
######################################################################
# Version of the Chart
CHARTVERSION="1.90.1"
# Name of the Chart
CHARTNAME=kubecost/cost-analyzer
# Repourl of the chart
CHARTURL=https://kubecost.github.io/cost-analyzer/
# Your release name 
RELEASENAME=kubecost
# Kubernetes Namespace we are going to use
NAMESPACE=kubecost
# Chart repository name for helm
CHARTREPONAME=kubecost
# path to your value file
VALUEFILEPATH=./values/values-prod.yaml

# add the helm repository
helm repo add $CHARTREPONAME $CHARTURL
# create source directory
mkdir -p source
# fetch chart into dir
helm fetch --untar --untardir source $CHARTNAME --version $CHARTVERSION
# create base dir for kustomize
mkdir -p tmp


yaml_copy () {
    echo $1
    D=$1
    if [ -d "${D}" ]; then
        # prefix is the chart name, even if its a subchart
        prefix=$(basename $(dirname " ${D}"))
        #create an output directory
        mkdir output/$prefix -p
        for filename in ${D}/*.yaml; do
            echo subchart: $prefix
            ## lets get the current filename and filedirectory
            f="$(basename -- $filename)"
            fd="$(dirname $filename)"
            #copy your files to the new destination
            cp $fd/$f output/$prefix/$f
        done;
    fi

}
## generate the helm chart with applied values into our template dir
helm template $RELEASENAME --output-dir tmp --namespace $NAMESPACE --values $VALUEFILEPATH kubecost/cost-analyzer
## pretty copy them into ../base instead of the helm directory
yaml_copy ${PWD}/tmp/cost-analyzer/templates
for D in ${PWD}/tmp/cost-analyzer/charts/*/; do
    yaml_copy $D"templates"
done

```

Run the script and you will end up with a nice folder structure for you and separate yaml files for kustomize.

Don't forget that you can get the current versions of your helm chart with:

``` Console
$ helm search repo kubecost/cost-analyzer --versions
```

Et voila, you should be able to convert your helm charts tzo kustomizations without any problems.

