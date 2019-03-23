# jobs

Containers by design are supposed to run a single primary process (along with any subprocesses spawned from the primary proceess). That process is either a continiously running process, as is the case of httpd or mysql docker images, or can be short lived processes, that does a specific task and then exits out. All the workload kube objects we've seen so far, Pods, ReplicaSets, Deployments are all designed for running pods with the primary container running a continious process. 

There are occasions where you have a docker image that's designed to run a short-lived tasks. If you create a single container pod from this image, then that pod will end and kubernetes will think that pod has failed and it will try to keep restarting, which is not what you want.

So if you want to run a shortlived pod on an adhoc basis, then you need to create a (controller) 'job' object. Here's what a job looks like:

```yaml
---
apiVersion: batch/v1
kind: Job
metadata:
  name: job-countdown
spec:
  completions: 4  # this causes the pod to run successfully 4 times, one after another. The default is 1 if not set.
  parallelism: 2  # specifies how many pods runs simultaneously, in parrallel. The default is 1 if not set. 
  template:
    spec:
      containers:
        - name: cntr-countdown
          image: centos:latest
          command:
            - "bin/bash"
            - "-c"
            - "for i in 9 8 7 6 5 4 3 2 1 ; do echo $i ; sleep 10 ; done"
      restartPolicy: Never
```

This ends up creating:

```bash
$ kubectl get jobs -o wide
NAME            COMPLETIONS   DURATION   AGE   CONTAINERS       IMAGES          SELECTOR
job-countdown   4/4           3m10s      15m   cntr-countdown   centos:latest   controller-uid=f8c94a67-41c8-11e9-9566-080027d15c4c
```

As soon as the job exists, it spun up the pod and waited for it to end:

```bash
$ kubectl get pod -o wide
NAME                  READY   STATUS      RESTARTS   AGE   IP           NODE       NOMINATED NODE   READINESS GATES
job-countdown-26jm2   0/1     Completed   0          13m   172.17.0.8   minikube   <none>           <none>
job-countdown-cwzjj   0/1     Completed   0          14m   172.17.0.7   minikube   <none>           <none>
job-countdown-rqq9d   0/1     Completed   0          13m   172.17.0.7   minikube   <none>           <none>
job-countdown-tjnqx   0/1     Completed   0          14m   172.17.0.8   minikube   <none>           <none>
```

And here's the output of the pod's log:

```bash
$ kubectl logs job-countdown-b26rw
9
8
7
6
5
4
3
2
1
```

The job along with it's pods continues to exist after the job's pods have finished running. That's so that you can the logs for troubleshooting purposes. However in reality you're unlikely to need the job or its pods again, so you need to do a cleanup. Also to manually run the job again you do:

```bash
$ kubectl delete -f configs          # this does the cleanup
job.batch "job-countdown" deleted

$ kubectl apply -f configs
job.batch/job-countdown created    # this runs the job again
```

This ends up deleting the previous job and it's corresponding pods, and then start again.

The cleanup part can be automated do by using a higher level controller object known as cronjobs, covered later. 






