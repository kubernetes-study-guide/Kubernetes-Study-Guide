# Course Overview

The course takes a learn-by-doing approach so that you get acquainted with as many of the kubernetes features as quickly as possible. At it's core, Kubernetes is all about using the  `kubectl` command along with yaml files that you feed into it. That's why this course is primarily focused on how to write these yaml files and then feeding them into the `kubectl`. The hope is that you'll understand and learn kubernetes faster by seeing it in action and following along.

## Certified Kubernetes Administrator

In case you're interested, you can get certified by taking the [Certified Kubernetes Administrator](https://www.cncf.io/certification/cka/) exam. This isn't a multiple choice exam, it's actually a hands-on exam where you perform tasks on a live Kubernetes environment. And since this course is a hands-on course, it makes it a good companion guide to help you prepare for the exam.

I should point out that the [exam curriculum](https://github.com/cncf/curriculum) is regularly updated, and this course doesn't quite cover the whole curriculum. However I am planning to create further courses to cover the missing parts in upcoming courses. Also this course is designed to prepare you for using Kubernetes in a day-to-day which means that in some areas it goes beyond what's covered in the curriculum.

In the exam you will be allowed to access all web pages under the [https://kubernetes.io/docs/](https://kubernetes.io/docs/). As well as all pages under [https://github.com/kubernetes/kubernetes](https://github.com/kubernetes/kubernetes). That's why in this course I will regularly refer to these two sources, so that you can find your around them.

## What's covered

In this course you will learn and create:

- Pods
- Services
- Replicasets
- Deploymments
- Ingress rules
- Namespaces
- Jobs and Cronjobs
- Persistant Volumes
- Persistant Volume Claims
- deamonsets
- statefulsets
- Service Accounts
- configmaps
- secrets
- resourcequotas
- roles and rolebindings
- clusterroles and clusterrolebindingss
- networkpolicies
- ...and much much more!


## What's not covered

Kubernetes is a big software and there are some topics that I haven't included in this course, simply because these topics are so big that they need to be a course in their own right. 

- Various ways of installing Kubernetes, we only show how to setup a Kubecluster locally using Minikube (as well as vagrant for a small number of videos).
- Other tools that are also used in the wider Kubernetes ecosystem, e.g. [helm](https://helm.sh/), [traefik](https://traefik.io/),...etc. This course is going to be long enough as it is just with me just covering Kubernetes on it own!
- Kubernetes Administration, e.g. updating existing Kubernetes install to a newer version.
- Cloud platform specific technologies, such as [AWS EKS](https://aws.amazon.com/eks/). This course is going to be platform agnostic, meaning that what we'll cover should work on most, if not all platforms, whether it is On-promise, AWS, Azure, GKE...etc.

I'm planning create more courses in future to cover these areas.


## Bookmarks

[https://towardsdatascience.com/key-kubernetes-concepts-62939f4bc08e?sk=d46386ce56c00701dbf41aa8d308a14d](https://towardsdatascience.com/key-kubernetes-concepts-62939f4bc08e?sk=d46386ce56c00701dbf41aa8d308a14d)

[https://github.com/ramitsurana/awesome-kubernetes](https://github.com/ramitsurana/awesome-kubernetes)

[https://kubectl.docs.kubernetes.io/pages/kubectl_book/getting_started.html](https://kubectl.docs.kubernetes.io/pages/kubectl_book/getting_started.html)

eli5 guides
[https://www.cncf.io/the-childrens-illustrated-guide-to-kubernetes/](https://www.cncf.io/the-childrens-illustrated-guide-to-kubernetes/)
[https://cloud.google.com/kubernetes-engine/kubernetes-comic/](https://cloud.google.com/kubernetes-engine/kubernetes-comic/)
