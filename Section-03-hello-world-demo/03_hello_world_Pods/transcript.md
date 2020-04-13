# hello-world pod

Finally, at long last, We're now ready to do our first hello world demo. For this demo I'll create a pod with the apache container running inside it.

The first step is to write a yaml file that defines the pod. So here's a yaml file that I wrote earlier:

```bash
ls -l
cat pod-httpd.yaml
```

You can find a copy of this file inside the configs folder in the course notes.

A big part of day-to-day Kubernetes work, can involve creating these yaml files.

Writing something like this might look a bit scary. However it's not that bad. That's because most of these yaml files have a similar high level structure. Also I rarely write these files from scratch, instead I find an existing yaml file and make copy of it, and then use the copy as template and repurpose it for my needs.

We'll cover more about how to write these files later in the course.

But for now this yaml file is basically saying that:

- We want to create a pod
- The pod's name should be "pod-web"
- we want to assign a key/value label, in this case I'm calling my label 'app' and giving it the value 'web'. I can add more labels here if I want to, but i'll stick with just one for now.
- This pod should only have one container
- the containers name should be 'cntr-apache'
- the container is listening on port 80
- and finally, the container should be built using the official apache image which has the image tag of 'latest'.



Let's now create this pod by feeding this yaml file into the kubectl apply command:

```bash
kubectl apply -f pod-apache.yml
```

It doesn't matter what the filename is, as long as it ends with the yaml extension.

Ok it looks like our pod has been created now. Lets see if we can view it using the kubectl-get-pods command:

```bash
kgp
```

This command lists all the pod in our kube cluster. In our case it shows the pod that we've just created. The pod's name matches what we asked for in our yaml file. Our pod houses a total of one container, and here it says that 1 out of 1 container is ready. The Pod is currently in running status and kubernetes hasn't had a need to restart it.

I'll set the output flag to 'wide' to print out some more info:

```bash
kgp -o wide
```

This gives us bit more detail, in particular which worker node the pod is running on, as well as the pod's IP address. Kubernetes has it's own private internal network and that ip address belongs to that private network.

That means that we can't access that network from outside, such as from my macbook:

```bash
curl http://pods-ip-address
curl: (7) Failed to connect to 172.17.0.8 port 80: Operation timed out
```

So how can we test to see our apache container is actually working. Well the proper is by creating a service object, which we'll do in the next video.

But for now I'll use alternative technique that software developers often use for troubleshooting purposes, and that is port-forwarding.

``` right terminal
kubectl port-forward pod-hello-world 1234:80
# hit enter
```

Her'e we're saying that any traffic our workstations receives on port 1234 should be forwarded to our apache pod at port 80. This is a bit like creating a tunnel between my macbook and the kube cluster, and kubectl is continuously monitoring for any traffic and acting as the go between. That's why it hasn't exited and is hanging.

Now let's try curling our pod using localhost:1234




```
curl http://localhost:1234
```

Bingo that worked.


In the next video we'll look at how to access our pod the proper usign kubernetes services.

















