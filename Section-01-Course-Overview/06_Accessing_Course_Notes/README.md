# The Kubernetes Study Guide

This course comes with it's very own study guide which you need to download from Github:

[https://github.com/Sher-Chowdhury/Kubernetes-Study-Guide](https://github.com/Sher-Chowdhury/Kubernetes-Study-Guide)

You can download this guide by doing a git clone:

```bash
git clone git@github.com:Sher-Chowdhury/Kubernetes-Study-Guide.git
```

This guide is made up number of sections and Each of these sections have a collection fo topic folders. These sections and topic folders corresponds to the videos. For example the video you're watching right now corresponds to the '02_The_Kubernetes_Study_Guide' video.

Each folder contains a README file that covers each topic and includes the commands demoed for that video. Most of these folders contains a configs folder. This contains sample yaml files that are demoed in the corresponding videos. This will save you the trouble of manually typing out everything I do.  

This study guide is regularly being updated with new content and it also goes into more detail for each video, so it's worth having this guide open as you watch the videos.

## Notations and where to find help

Throughout this course we'll be using the kubectl command. kubectl is the main command used for performing day-to-day kubernetes work. kubectl has a lot of built in reference docs. That includes lots of man pages:

```bash
man kubectl<tab><tab>
```

Also you can access more info by running:

```bash
kubectl explain xxxxx
```

Where 'xxxxx', is set to something like `pod.spec.containers.name`.

The output of some commands are quite long, on those occasions we'll only show an extract using 3-dot notation, and truncate out the rest:

```text
$ kubectl describe pods
...
output of interest
...
```
