# cronjobs

Earlier we looked at job objects. When you create a job object, the corresponding pods are spun up straight away. If you want these pods to run on a regular cron like basis, then you need create the job when as soon as the scheduled time arrives. 

The crude way of doing this is by creating a linux cron job (or jenkins job), that runs 'kubectl apply' command, followed by 'kubectl delete' cleanup task. The proper way to do this is by creating type of object known as cronjobs. Cronjobs are controller objects. 


```yaml
---
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: cronjob-countdown
spec:
  schedule: "*/2 * * * *"     # run every job every couple of minutes. 
  jobTemplate:
    spec:
      template:
        spec:
          restartPolicy: OnFailure
          containers:
            - name: cntr-countdown
              image: centos:latest
              command:
                - "/bin/bash"
                - "-c"
                - |
                  for i in 9 8 7 6 5 4 3 2 1 ; do
                    echo $i
                    sleep 10
                  done
```

Once you've applied this, you'll end up with:

```bash
$ kubectl get cronjobs
NAME                SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
cronjob-countdown   */2 * * * *   False     0        2m5s            4m57s

$ kubectl get jobs
NAME                           COMPLETIONS   DURATION   AGE
cronjob-countdown-1552087800   1/1           94s        4m1s
cronjob-countdown-1552087920   1/1           93s        2m1s
cronjob-countdown-1552088040   0/1           1s         1s

$ kubectl get pods
NAME                                 READY   STATUS      RESTARTS   AGE
cronjob-countdown-1552087800-zfv7j   0/1     Completed   0          4m6s
cronjob-countdown-1552087920-78nms   0/1     Completed   0          2m6s
cronjob-countdown-1552088040-2g67b   1/1     Running     0          6s
```

By default cronjobs will store the last 3 successful and failed jobs along with corresponding pods in history. That's why you get a few extra objects shown above. But you update thes. 





