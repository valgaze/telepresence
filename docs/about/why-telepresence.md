---
layout: doc
weight: 1
title: "Why Telepresence?"
categories: about
---

<link rel="stylesheet" href="{{ "/css/mermaid.css" | prepend: site.baseurl }}">
<script src="{{ "/js/mermaid.min.js" | prepend: site.baseurl }}"></script>
<script>mermaid.initialize({
   startOnLoad: true,
   cloneCssStyles: false,
 });
</script>

Let's assume you have a web service which listens on port 8080, and has a Dockerfile which gets built to an image called `examplecom/servicename`.
Your service depends on other Kubernetes `Service` instances (`thing1` and `thing2`), and on a cloud database.

The Kubernetes production environment looks like this:

<div class="mermaid">
graph LR
  subgraph Kubernetes in Cloud
    code["k8s.Pod: servicename"]
    s1["k8s.Service: servicename"]---code
    code---s2["k8s.Service: thing1"]
    code---s3["k8s.Service: thing2"]
    code---c1>"Cloud Database (AWS RDS)"]
  end
</div>

### The slow status quo

If you need that cloud database and those two services to directly test your software, you will need to do the following to test a change:

1. Change your code.
2. Build a Docker image.
3. Push the Docker image to a Docker registry in the cloud.
4. Update the Kubernetes cluster to use your new image.
5. Wait for the image to download.

This is slow.

<div class="mermaid">
graph TD
  subgraph Laptop
    code["Source code for servicename"]==>local["Docker image"]
    kubectl
  end
  subgraph Kubernetes in Cloud
    local==>registry["Docker registry"]
    registry==>deployment["k8s.Deployment: servicename"]
    kubectl==>deployment
    s1["k8s.Service: servicename"]---deployment
    deployment---s2["k8s.Service: thing1"]
    deployment---s3["k8s.Service: thing2"]
    deployment---c1>"Cloud Database (AWS RDS)"]
  end
</div>

### A fast development cycle with Telepresence

Telepresence works by running your code *locally*, as a normal local process, and then forwarding forwarding requests to/from the remote Kubernetes cluster.

<div class="mermaid">
graph TD
  subgraph Laptop
    code["Source code for servicename"]==>local["local process"]
    local---client[Telepresence client]
  end
  subgraph Kubernetes in Cloud
    client-.-proxy["k8s.Pod: Telepresence proxy"]
    s1["k8s.Service: servicename"]---proxy
    proxy---s2["k8s.Service: thing1"]
    proxy---s3["k8s.Service: thing2"]
    proxy---c1>"Cloud Database (AWS RDS)"]
  end
</div>

This means development is fast: you only have to change your code and restart your process.
Many web frameworks also do automatic code reload, in which case you won't even need to restart.
