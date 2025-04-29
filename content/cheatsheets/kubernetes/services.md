---
title: Services
linktitle: Services
type: book
date: "2022-03-26T00:00:00+01:00"
tags:
  - Docker
  - Kubernetes
# Prev/next pager order (if `docs_section_pager` enabled in `params.toml`)
weight: 30
---

Kubernetes Networking with Services

<!--more-->
## Overview

Kubernetes **Service** object is the way in Kubernetes to expose applications, both on the network and to the outside world. Exposing the app could be ```external``` like from a web browser and ```internal``` like accessing it from maybe another Pod or application on the same cluster that's talking to it.

🥁 Services nail both these scenarios.

![k8s-services-apiserver](/images/uploads/k8s-services-apiserver.png)

Let's look at ```services``` with an example below:
![k8s-services-clusterIP](/images/uploads/k8s-services-clusterIP.png)

There's a Pod (<span style="color:green">*Green*</span>) hosting a web front end needing to talk to a few of backend Pods (<span style="color:blue">*Blue*</span>) down below. Well, we slip in a ```service``` object in the middle. ```service``` object is just a Kubernetes API object like a ```pod``` or ```deployment``` or anything else, meaning we define it in a YAML manifest, and we create it by throwing that manifest at the API server.

## ClusterIP
Once a ```service``` is created it provides a stable IP (a.k.a. **ClusterIP**) and DNS name, so, a single IP and DNS name that then load balancers requests it receives to the backend Pods.

Then if one of the (<span style="color:blue">*Blue*</span>) Pods die or gets replaced by another, it's all good, because the service is watching, and it just updates the list that it holds are valid, healthy Pods. But importantly, it never changes the stable and reliable **ClusterIP** and DNS name for the ```service```. That never changes. In fact, part of the contract we have with Kubernetes is that once this ```service``` is defined, that it's IP and DNS will never, ever, ever, ever change. It is a just a high‑level stable abstraction point for multiple Pods.

## Load Balancing
Also it provides basic load balancing. The way that a Pod belongs to a service or makes it onto the list of Pods that a service will forward traffic to is via ```labels```. See below how the Pod manifest got a label called "web". Well,  just put the same label in the service manifest under the label selector, and the service is going to send traffic to that Pod.
![k8s-services-selector](/images/uploads/k8s-services-selector.PNG)

<img align="right" width="200" height="200" src="/images/uploads/k8s-endpoint.png">
As well, every time you create a service, Kubernetes automatically creates an endpoint object or an endpoint slice,
depending on your version of Kubernetes. Either way, it's just a dynamic list of healthy Pods that match the service's label selector.

### Internal Access

We already said that a ```service``` gets a **ClusterIP**, and as the name suggests, that is for **inside** the cluster. And we also said that the name of the service gets registered with the internal DNS service, and every container uses this DNS service when it is resolving names to IPs.
![k8s-services-internal-access](/images/uploads/k8s-services-internal-access.png)

Four Pods inside the cluster(<span style="color:green">*Green*</span>) wanting to talk to backend Pods(<span style="color:blue">*Blue*</span>), so long as they know the name of the service in front of the Pods, it fires that off to the internal DNS service, and it gets back the ```cluster IP```.

And then from there on, it just sends traffic to that ```cluster IP```, and the cluster takes care of getting it to individual Pods.

### External Access

* **NodePort**: A ```service``` also gets a network port. Well, that port can be mapped on every cluster node to point back to the cluster IP.

  So, in below example, the service has a port of 30001, and that's mapped on every node in the cluster, meaning we can sit outside of the cluster and send requests literally to any node on that port, and Kubernetes makes sure that it's routed to the ```cluster IP``` and eventually the Pods behind it.

  And we call this a ```NodePort```. Again, it's in the name! Every node gets the port mapped.

  ![k8s-services-nodeport](/images/uploads/k8s-services-external-access-nodeport.png)

* **LoadBalancer**: For external access there's a third type of ```service``` object: **LoadBalancer** type, and it seamlessly integrates with your cloud provider's native load balancers to provide access from over the internet.

  And the :sparkling_heart:	part, Kubernetes literally does it all, and I mean **all** right? You literally just define a YAML file that says type equals ```LoadBalancer```, and Kubernetes does the rest.
  ![k8s-services-loadbalancer](/images/uploads/k8s-services-external-access-loadbalancer.png)

{{% callout note %}}
* There's three major service types, and each one is useful for a different requirement. At the bottom is ClusterIP, and this is the default, right, so if you don't explicitly set a type, that's what you'll get. Now, it is a stable IP within a cluster, so a ClusterIP only makes the service available from inside the cluster.

* Next up, there's the NodePort that we're going with. This takes this ClusterIP, which is needed for routing within the cluster, and it adds a cluster‑wide TCP or UDP port on top. In fact, it's what we just saw when we assigned it a random port above 30000 and tied the service to that port on every node in the cluster.

* Finally we have the LoadBalancer type, which works with Cloud providers loadbalancer and exposes a stable endpoint to the internet.
![k8s-services-hierarchy](/images/uploads/k8s-services-hierarchy.png)
{{% /callout %}}




## Create Services

### Imperative

```
cloud_user@k8s-control:~$ kubectl expose pod hello-pod --name=hello-svc --target-port=8080 --type=NodePort
service/hello-svc exposed
cloud_user@k8s-control:~$ kubectl get svc
NAME         TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)          AGE
hello-svc    NodePort    10.99.123.2   <none>        8080:31790/TCP   12s
kubernetes   ClusterIP   10.96.0.1     <none>        443/TCP          12d
```
### Declarative



## Further Read
The take‑home point is that ```services``` provide reliable networking for Pods.
