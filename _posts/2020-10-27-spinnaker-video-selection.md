---
layout: post
title: "Spinnaker video selection"
excerpt: "A selection of three videos about Spinnaker."
date: 2020-10-27
tags: [Video Section, CI/CD]
feature_image: __GHOST_URL__/content/images/2020/10/karla-car-58AiTToabyE-unsplash-1.jpg
---

If you work with containerised applications or Kubernetes, you've probably ran into Spinnaker. Spinnaker is an open source, multi-cloud continuous delivery platform for releasing software. Spinnaker was initially created and then open sourced by Netflix. Now it has a huge following and a large number of contributors with Google probably being the biggest one.

There are many videos and materials about Spinnaker on the inter webs. I selected three that I believe are worth your time.

### Spinnaker: continuous delivery from first principles to production

The presenters are [Steven Kim](https://www.linkedin.com/in/stevenjkim/) from Google and [Tom Feiner](https://twitter.com/tom_feiner) from Waze. This video is already getting old but Steven mentions few really valuable things that make it very relevant even 3 years after. One of those things are the principles of Continuous Delivery. These are:

* Immutable Infrastructure([DigitalOcean](https://www.digitalocean.com/community/tutorials/what-is-immutable-infrastructure),[HashiCorp](https://www.hashicorp.com/resources/what-is-mutable-vs-immutable-infrastructure))
* Deployment strategies
  + [Blue/Green](https://en.wikipedia.org/wiki/Blue-green_deployment)
  + Rolling Blue/Green
  + [Canary](https://martinfowler.com/bliki/CanaryRelease.html)
* Automation (Repeatability and Consistency)

* Immutable Infrastructure [[1](https://www.digitalocean.com/community/tutorials/what-is-immutable-infrastructure)] [[2](https://www.hashicorp.com/resources/what-is-mutable-vs-immutable-infrastructure)]

In the end of the video there is a fun and interesting demo. Also, through out the video you get to hear about real world examples from Waze and how they managed to solve them with Spinnaker. Cool stuff!

### Using Kubernetes, Spinnaker and Istio to Manage a Multi-cloud Environment (Cloud Next '18)

The presenter of this video is Ameer Abbas. As the title says, he talks about Docker, Kubernetes, Spinnaker and Istio. He gives a really clear and concise explanation of why we need them, how we use these tools and how they all fit together.

![](__GHOST_URL__/content/images/2020/10/KubernetesArchitecture.png)

While talking about these tools he also touches on the following aspects:

* Why multicloud? When do we need multiple clouds/clusters/namespaces?
* [Federation](https://platform9.com/blog/kubernetes-federation-what-it-is-and-how-to-set-it-up/) vs. Cluster Independence.
* Dev and the DevOps experience.
* Separating applications from network functions.
* Routing, Resilience and Security.
* Proxies ([Envoy](https://www.envoyproxy.io)) and service mesh.

Great presenter, great talk packed with interesting information. He excellently paints the whole picture. Well worth the 45 minutes.

### Deploying Spinnaker in Kubernetes cluster

The third video is a more practical one from the [Just me and Opensource](Deploying Spinnaker in Kubernetes cluster) YouTube channel. It explains how to install Spinnaker in a K8S cluster on bare metal. The presentation lacks detailed explanations of the commands being run so it's not perfect. However, all the steps are clearly specified and well executed and the flow has a nice pace. As an added benefit, the author provides all the installation steps he follows in the video [in a written form on Github](https://github.com/justmeandopensource/kubernetes/blob/master/docs/setup-spinnaker.md).

Some of the things that he mentions that you might want to expand on:

* [Dynamic Volume Provisioning](https://kubernetes.io/docs/concepts/storage/dynamic-provisioning/) for Kubernetes.
* [Minio](https://min.io), a Kubernetes native, high performance, object storage.

### Credits

* Cover Photo by [Karla Car](https://unsplash.com/@karlacar?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/s/photos/sailboat?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)