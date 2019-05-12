# The Kubernetes Study Guide

This course comes with it's very own study guide which I strongly recommend that you clone to your workstation:

[https://github.com/Sher-Chowdhury/Kubernetes-Study-Guide](https://github.com/Sher-Chowdhury/Kubernetes-Study-Guide)

I've structured this course into a number of parts. Each part contains a list of numbered folders. Each of these numbered folder corresponds to a video on the course. For example the video you're watching now corresponds to the '02_The_Kubernetes_Study_Guide' video.

Each of these folders contains a README file that covers each topic and includes the commands demoed for that video. Most of these folders contains a configs folder, which contains sample config files that are demoed in the videos. This will save you the trouble of manually typing out everything I do.  

This study guide is constantly evolving and is being regularly updated with improvements and new content. Also if you have any suggestions about making changes then please raise an issue or submit a pull request.

## Notations and where to find help

Throughout this course we'll be using the kubectl command. kubectl is the main command used for performing day-to-day kubernetes work. kubectl has a lot of built in reference docs. That includes lots of man pages:

```bash
man kubectl<tab><tab>
```

Also you can access a lot more info by running:

```bash
kubectl explain xxxxx
```

Where 'xxxxx', is set to something like 'pod.spec'.

The output of some commands are quite long, on those occasions we'll only show an extract using 3-dot notation, and truncate out the rest:

```text
$ kubectl describe pods
...
output of interest
...
```
