If you run minikube-help then you'll see all the available commands. 

```bash
minikube help
```

Here we can see that if you want to ssh into the minikube vm, you just run:

```bash
minikube ssh
```

After you're logged in, you can then switch to root user if you want. 

```bash

```

ok I'll exit out now. 

In kubernetes there is a really cool app you can install called Kubernetes Dashboard, This provides 
a powerful graphical web interface that let's you view and manage your kubecluster visually. This dashboard is quite straight forward to install. However the nice thing about minikube is that this dashboards comes preinstalled.
You can access the web ui by running the dashboard command:

```bash
minikube dashboard
```

I recommend exploring this dashboard as you go through the course. 

Finally once you have finished working with your kube cluster, you can run the stop command:

```bash
minikube stop
```

This will shutdown your minikube vm. However if you want to delete it altogether then you use the delete command:

```bash
minikube delete
```

That's it for this video see you in the next one. 