---

layout: single
title:  "Deploying Jobs on Google Kubernetes Engine"
categories: Cloud
tags: GCP
classes: wide
show_date: true
header:
  overlay_image: /assets/images/k8s-banner.png
  og_image: /assets/images/k8s-banner.png
  teaser: /assets/images/pca-gcp.png
author:
  name     : "Professional Cloud Architect"
  avatar   : "/assets/images/pca-gcp.png"

sidebar:
  - title: "Topics"
    nav: my-sidebar

---
# Deploying Jobs on Google Kubernetes Engine

In GKE, a Job is a controller object that represents a finite task.  Jobs manage a task as it runs to completion, rather than managing an  ongoing desired state such as maintaining the total number of running  Pods.

CronJobs perform finite, time-related tasks that run once or  repeatedly at a time that you specify using Job objects to complete  their tasks.

- Define, deploy and clean up a GKE Job
- Define, deploy and clean up a GKE CronJob

## Define and deploy a Job manifest

In this task, you create a Job, inspect its status, and then remove it.

```sh
Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-03-7569ff0296bd.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_02_c4beb0285b69@cloudshell:~ (qwiklabs-gcp-03-7569ff0296bd)$ export my_region=us-east1
export my_cluster=autopilot-cluster-1
student_02_c4beb0285b69@cloudshell:~ (qwiklabs-gcp-03-7569ff0296bd)$ source <(kubectl completion bash)
student_02_c4beb0285b69@cloudshell:~ (qwiklabs-gcp-03-7569ff0296bd)$ gcloud container clusters get-credentials $my_cluster --region $my_region
Fetching cluster endpoint and auth data.
kubeconfig entry generated for autopilot-cluster-1.
student_02_c4beb0285b69@cloudshell:~ (qwiklabs-gcp-03-7569ff0296bd)$ kubectl get nodes
NAME                                                 STATUS   ROLES    AGE   VERSION
gk3-autopilot-cluster-1-nap-iw5ihjci-65ca1e83-9vk9   Ready    <none>   74s   v1.29.4-gke.1043002
student_02_c4beb0285b69@cloudshell:~ (qwiklabs-gcp-03-7569ff0296bd)$ 
```

### Create and run a Job

Let's create a sample Job that computes the value of Pi to 2,000 places and then prints the result.

1. Create and open a file called `example-job.yaml` 

```yaml
student_02_c4beb0285b69@cloudshell:~ (qwiklabs-gcp-03-7569ff0296bd)$ cat example-job.yaml 
apiVersion: batch/v1
kind: Job
metadata:
  # Unique key of the Job instance
  name: example-job
spec:
  template:
    metadata:
      name: example-job
    spec:
      containers:
      - name: pi
        image: perl:5.34
        command: ["perl"]
        args: ["-Mbignum=bpi", "-wle", "print bpi(2000)"]
      # Do not restart containers after they exit
      restartPolicy: Never
student_02_c4beb0285b69@cloudshell:~ (qwiklabs-gcp-03-7569ff0296bd)$ 
```

```sh
student_02_c4beb0285b69@cloudshell:~ (qwiklabs-gcp-03-7569ff0296bd)$ kubectl apply -f example-job.yaml
Warning: autopilot-default-resources-mutator:Autopilot updated Job default/example-job: defaulted unspecified 'cpu' resource for containers [pi] (see http://g.co/gke/autopilot-defaults).
job.batch/example-job created
student_02_c4beb0285b69@cloudshell:~ (qwiklabs-gcp-03-7569ff0296bd)$ 
```

```sh
student_02_c4beb0285b69@cloudshell:~ (qwiklabs-gcp-03-7569ff0296bd)$ kubectl describe job example-job
Name:             example-job
Namespace:        default
Selector:         batch.kubernetes.io/controller-uid=b43189af-5981-466e-9837-e6f1e0772120
Labels:           batch.kubernetes.io/controller-uid=b43189af-5981-466e-9837-e6f1e0772120
                  batch.kubernetes.io/job-name=example-job
                  controller-uid=b43189af-5981-466e-9837-e6f1e0772120
                  job-name=example-job
Annotations:      autopilot.gke.io/resource-adjustment:
                    {"input":{"containers":[{"name":"pi"}]},"output":{"containers":[{"limits":{"cpu":"500m","ephemeral-storage":"1Gi","memory":"2Gi"},"request...
                  autopilot.gke.io/warden-version: 2.9.37
Parallelism:      1
Completions:      1
Completion Mode:  NonIndexed
Start Time:       Mon, 10 Jun 2024 01:17:28 +0000
Completed At:     Mon, 10 Jun 2024 01:21:39 +0000
Duration:         4m11s
Pods Statuses:    0 Active (0 Ready) / 1 Succeeded / 0 Failed
Pod Template:
  Labels:  batch.kubernetes.io/controller-uid=b43189af-5981-466e-9837-e6f1e0772120
           batch.kubernetes.io/job-name=example-job
           controller-uid=b43189af-5981-466e-9837-e6f1e0772120
           job-name=example-job
  Containers:
   pi:
    Image:      perl:5.34
    Port:       <none>
    Host Port:  <none>
    Command:
      perl
    Args:
      -Mbignum=bpi
      -wle
      print bpi(2000)
    Limits:
      cpu:                500m
      ephemeral-storage:  1Gi
      memory:             2Gi
    Requests:
      cpu:                500m
      ephemeral-storage:  1Gi
      memory:             2Gi
    Environment:          <none>
    Mounts:               <none>
  Volumes:                <none>
Events:
  Type    Reason            Age    From            Message
  ----    ------            ----   ----            -------
  Normal  SuccessfulCreate  4m52s  job-controller  Created pod: example-job-zww22
  Normal  Completed         42s    job-controller  Job completed
student_02_c4beb0285b69@cloudshell:~ (qwiklabs-gcp-03-7569ff0296bd)$ 
```

```sh
student_02_c4beb0285b69@cloudshell:~ (qwiklabs-gcp-03-7569ff0296bd)$ kubectl get pods
NAME                READY   STATUS      RESTARTS   AGE
example-job-zww22   0/1     Completed   0          4m22s
student_02_c4beb0285b69@cloudshell:~ (qwiklabs-gcp-03-7569ff0296bd)$ 
```

### Clean up and delete the Job

When a Job completes, the Job stops creating Pods. The Job API object is not removed when it completes, which allows you to view its status.  Pods created by the Job are not deleted, but they are terminated.  Retention of the Pods allows you to view their logs and interact with  them.

```sh
student_02_c4beb0285b69@cloudshell:~ (qwiklabs-gcp-03-7569ff0296bd)$ kubectl get jobs
NAME          COMPLETIONS   DURATION   AGE
example-job   1/1           4m11s      6m1s
student_02_c4beb0285b69@cloudshell:~ (qwiklabs-gcp-03-7569ff0296bd)$ 
```

```sh
student_02_c4beb0285b69@cloudshell:~ (qwiklabs-gcp-03-7569ff0296bd)$ kubectl logs example-job-zww22
3.1415926535897932384626433832795028841971693993751058209749445923078164062862089986280348253421170679821480865132823066470938446095505822317253594081284811174502841027019385211055596446229489549303819644288109756659334461284756482337867831652712019091456485669234603486104543266482133936072602491412737245870066063155881748815209209628292540917153643678925903600113305305488204665213841469519415116094330572703657595919530921861173819326117931051185480744623799627495673518857527248912279381830119491298336733624406566430860213949463952247371907021798609437027705392171762931767523846748184676694051320005681271452635608277857713427577896091736371787214684409012249534301465495853710507922796892589235420199561121290219608640344181598136297747713099605187072113499999983729780499510597317328160963185950244594553469083026425223082533446850352619311881710100031378387528865875332083814206171776691473035982534904287554687311595628638823537875937519577818577805321712268066130019278766111959092164201989380952572010654858632788659361533818279682303019520353018529689957736225994138912497217752834791315155748572424541506959508295331168617278558890750983817546374649393192550604009277016711390098488240128583616035637076601047101819429555961989467678374494482553797747268471040475346462080466842590694912933136770289891521047521620569660240580381501935112533824300355876402474964732639141992726042699227967823547816360093417216412199245863150302861829745557067498385054945885869269956909272107975093029553211653449872027559602364806654991198818347977535663698074265425278625518184175746728909777727938000816470600161452491921732172147723501414419735685481613611573525521334757418494684385233239073941433345477624168625189835694855620992192221842725502542568876717904946016534668049886272327917860857843838279679766814541009538837863609506800642251252051173929848960841284886269456042419652850222106611863067442786220391949450471237137869609563643719172874677646575739624138908658326459958133904780275901
student_02_c4beb0285b69@cloudshell:~ (qwiklabs-gcp-03-7569ff0296bd)$
```

The output will show that the job wrote the first two thousand digits of pi to the Pod log.

```sh
student_02_c4beb0285b69@cloudshell:~ (qwiklabs-gcp-03-7569ff0296bd)$ kubectl delete job example-job
job.batch "example-job" deleted
student_02_c4beb0285b69@cloudshell:~ (qwiklabs-gcp-03-7569ff0296bd)$ 
```

If you try to query the logs again the command will fail as the Pod can no longer be found.

```sh
student_02_c4beb0285b69@cloudshell:~ (qwiklabs-gcp-03-7569ff0296bd)$ kubectl get pods
No resources found in default namespace.
student_02_c4beb0285b69@cloudshell:~ (qwiklabs-gcp-03-7569ff0296bd)$ kubectl logs example-job-zww22
Error from server (NotFound): pods "example-job-zww22" not found
student_02_c4beb0285b69@cloudshell:~ (qwiklabs-gcp-03-7569ff0296bd)$ 
```

## Define and deploy a CronJob manifest

You can create CronJobs to perform finite, time-related tasks that run once or repeatedly at a time that you specify.

In this task, you create and run a CronJob, and then you clean up and delete the Job.

This CronJob deploys a new container every minute that prints the time, date and "Hello, World!".

1. Create and open a file called `example-cronjob.yaml`

```yaml
student_02_c4beb0285b69@cloudshell:~ (qwiklabs-gcp-03-7569ff0296bd)$ cat example-cronjob.yaml 
apiVersion: batch/v1
kind: CronJob
metadata:
  name: hello
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: hello
            image: busybox
            args:
            - /bin/sh
            - -c
            - date; echo "Hello, World!"
          restartPolicy: OnFailure
student_02_c4beb0285b69@cloudshell:~ (qwiklabs-gcp-03-7569ff0296bd)$ 
```

CronJobs use the required `schedule` field, which accepts a time in the Unix standard `crontab` format.



All CronJob times are in UTC:

- The first value indicates the minute (between 0 and 59).
- The second value indicates the hour (between 0 and 23).
- The third value indicates the day of the month (between 1 and 31).
- The fourth value indicates the month (between 1 and 12).
- The fifth value indicates the day of the week (between 0 and 6).



The `schedule` field also accepts * and ? as wildcard values. Combining / with ranges specifies that the task should repeat at a  regular interval. In the example, `*/1 * * * *` indicates that the task should repeat every minute of every day of every month.

```sh
student_02_c4beb0285b69@cloudshell:~ (qwiklabs-gcp-03-7569ff0296bd)$ kubectl apply -f example-cronjob.yaml
Warning: autopilot-default-resources-mutator:Autopilot updated CronJob default/hello: defaulted unspecified 'cpu' resource for containers [hello] (see http://g.co/gke/autopilot-defaults).
cronjob.batch/hello created
student_02_c4beb0285b69@cloudshell:~ (qwiklabs-gcp-03-7569ff0296bd)$ 
```

```sh
student_02_c4beb0285b69@cloudshell:~ (qwiklabs-gcp-03-7569ff0296bd)$ kubectl get jobs
NAME             COMPLETIONS   DURATION   AGE
hello-28633048   1/1           5s         22s
student_02_c4beb0285b69@cloudshell:~ (qwiklabs-gcp-03-7569ff0296bd)$ 
```

```sh
student_02_c4beb0285b69@cloudshell:~ (qwiklabs-gcp-03-7569ff0296bd)$ kubectl describe job hello-28633048
Name:             hello-28633048
Namespace:        default
Selector:         batch.kubernetes.io/controller-uid=adcad3d9-a3d0-4656-8642-4d1ad84a2068
Labels:           batch.kubernetes.io/controller-uid=adcad3d9-a3d0-4656-8642-4d1ad84a2068
                  batch.kubernetes.io/job-name=hello-28633048
                  controller-uid=adcad3d9-a3d0-4656-8642-4d1ad84a2068
                  job-name=hello-28633048
Annotations:      batch.kubernetes.io/cronjob-scheduled-timestamp: 2024-06-10T01:28:00Z
Controlled By:    CronJob/hello
Parallelism:      1
Completions:      1
Completion Mode:  NonIndexed
Start Time:       Mon, 10 Jun 2024 01:28:00 +0000
Completed At:     Mon, 10 Jun 2024 01:28:05 +0000
Duration:         5s
Pods Statuses:    0 Active (0 Ready) / 1 Succeeded / 0 Failed
Pod Template:
  Labels:  batch.kubernetes.io/controller-uid=adcad3d9-a3d0-4656-8642-4d1ad84a2068
           batch.kubernetes.io/job-name=hello-28633048
           controller-uid=adcad3d9-a3d0-4656-8642-4d1ad84a2068
           job-name=hello-28633048
  Containers:
   hello:
    Image:      busybox
    Port:       <none>
    Host Port:  <none>
    Args:
      /bin/sh
      -c
      date; echo "Hello, World!"
    Limits:
      cpu:                500m
      ephemeral-storage:  1Gi
      memory:             2Gi
    Requests:
      cpu:                500m
      ephemeral-storage:  1Gi
      memory:             2Gi
    Environment:          <none>
    Mounts:               <none>
  Volumes:                <none>
Events:
  Type    Reason            Age   From            Message
  ----    ------            ----  ----            -------
  Normal  SuccessfulCreate  59s   job-controller  Created pod: hello-28633048-ck7dd
  Normal  Completed         53s   job-controller  Job completed
student_02_c4beb0285b69@cloudshell:~ (qwiklabs-gcp-03-7569ff0296bd)$ 
```

```sh
student_02_c4beb0285b69@cloudshell:~ (qwiklabs-gcp-03-7569ff0296bd)$ kubectl get pods
NAME                   READY   STATUS      RESTARTS   AGE
hello-28633048-ck7dd   0/1     Completed   0          117s
hello-28633049-z7kvx   0/1     Completed   0          57s
student_02_c4beb0285b69@cloudshell:~ (qwiklabs-gcp-03-7569ff0296bd)$ 
```

```sh
student_02_c4beb0285b69@cloudshell:~ (qwiklabs-gcp-03-7569ff0296bd)$ kubectl logs hello-28633048-ck7dd
Mon Jun 10 01:28:03 UTC 2024
Hello, World!
student_02_c4beb0285b69@cloudshell:~ (qwiklabs-gcp-03-7569ff0296bd)$ kubectl logs hello-28633049-z7kvx
Mon Jun 10 01:29:02 UTC 2024
Hello, World!
student_02_c4beb0285b69@cloudshell:~ (qwiklabs-gcp-03-7569ff0296bd)$ 
```

```sh
student_02_c4beb0285b69@cloudshell:~ (qwiklabs-gcp-03-7569ff0296bd)$ kubectl get jobs
NAME             COMPLETIONS   DURATION   AGE
hello-28633048   1/1           5s         3m4s
hello-28633049   1/1           5s         2m4s
hello-28633050   1/1           4s         64s
hello-28633051   0/1           4s         4s
student_02_c4beb0285b69@cloudshell:~ (qwiklabs-gcp-03-7569ff0296bd)$ kubectl get pods
NAME                   READY   STATUS      RESTARTS   AGE
hello-28633049-z7kvx   0/1     Completed   0          2m10s
hello-28633050-ngwhf   0/1     Completed   0          70s
hello-28633051-nmjzw   0/1     Completed   0          10s
student_02_c4beb0285b69@cloudshell:~ (qwiklabs-gcp-03-7569ff0296bd)$ 
```

By default Kubernetes sets the Job history limits so that only the last  three successful and last failed jobs are retained so this list will  only contain the most recent three of four jobs:

```sh
student_02_c4beb0285b69@cloudshell:~ (qwiklabs-gcp-03-7569ff0296bd)$ kubectl delete cronjob hello
cronjob.batch "hello" deleted
student_02_c4beb0285b69@cloudshell:~ (qwiklabs-gcp-03-7569ff0296bd)$ 
```

```sh
student_02_c4beb0285b69@cloudshell:~ (qwiklabs-gcp-03-7569ff0296bd)$ kubectl get jobs
No resources found in default namespace.
student_02_c4beb0285b69@cloudshell:~ (qwiklabs-gcp-03-7569ff0296bd)$ kubectl get pods
No resources found in default namespace.
student_02_c4beb0285b69@cloudshell:~ (qwiklabs-gcp-03-7569ff0296bd)$ 
```

All the Jobs were removed.

## History

```sh
student_02_c4beb0285b69@cloudshell:~ (qwiklabs-gcp-03-7569ff0296bd)$ history 
    1  export my_region=us-east1
    2  export my_cluster=autopilot-cluster-1
    3  source <(kubectl completion bash)
    4  gcloud container clusters get-credentials $my_cluster --region $my_region
    5  kubectl get nodes
    6  kubectl get pods
    7  kubectl get nodes
    8  kubectl get pods -A
    9  nano example-job.yaml
   10  cat example-job.yaml 
   11  kubectl apply -f example-job.yaml
   12  kubectl describe job example-job
   13  kubectl get pods
   14  kubectl get nodes
   15  kubectl get pods
   16  kubectl get nodes
   17  kubectl get pods
   18  kubectl apply -f example-job.yaml
   19  kubectl get pods
   20  kubectl describe job example-job
   21  kubectl get pods
   22  kubectl describe job example-job
   23  kubectl get jobs
   24  kubectl logs example-job-zww22
   25  kubectl delete job example-job
   26  kubectl get pods
   27  kubectl logs example-job-zww22
   28  nano example-cronjob.yaml
   29  cat example-cronjob.yaml 
   30  kubectl apply -f example-cronjob.yaml
   31  kubectl get jobs
   32  kubectl describe job hello-28633048
   33  kubectl get pods
   34  kubectl logs hello-28633048-ck7dd
   35  kubectl logs hello-28633049-z7kvx
   36  kubectl get jobs
   37  kubectl get pods
   38  kubectl delete cronjob hello
   39  kubectl get jobs
   40  kubectl get pods
   41  history 
student_02_c4beb0285b69@cloudshell:~ (qwiklabs-gcp-03-7569ff0296bd)$ 
```

