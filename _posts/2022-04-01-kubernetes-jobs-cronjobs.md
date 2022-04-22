---
layout: single
title:  "Kubernetes Jobs and CronJobs"
date:   2022-04-01 13:55:04 +0530
categories: Kubernetes
tags: kubeadm minikube
author: "Pradeep Gadde"
classes: wide
toc: true
show_date: true
header:
  teaser: /assets/images/cronjob-128.png
author:
  name     : "Kubernetes"
  avatar   : "/assets/images/kubernetes.png"

sidebar:
  - title: "Blog"
   
    text: "Checkout other topics"
    nav: my-sidebar
---

# Kubernetes Jobs and CronJobs

## Jobs



A Job creates one or more Pods and will continue to retry execution of  the Pods until a specified number of them successfully terminate. A simple case is to create one Job object in order to reliably run one Pod to completion. The Job object will start a new Pod if the first Pod fails or is deleted (for example due to a node hardware failure or a node reboot).



```sh
lab@k8s1:~$ kubectl get job
No resources found in default namespace.
lab@k8s1:~$ kubectl get job -A
No resources found
lab@k8s1:~$
```

```sh
lab@k8s1:~$ kubectl create job -h
Create a job with the specified name.

Examples:
  # Create a job
  kubectl create job my-job --image=busybox

  # Create a job with a command
  kubectl create job my-job --image=busybox -- date

  # Create a job from a cron job named "a-cronjob"
  kubectl create job test-job --from=cronjob/a-cronjob

Options:
      --allow-missing-template-keys=true: If true, ignore any errors in templates when a field or map key is missing in
the template. Only applies to golang and jsonpath output formats.
      --dry-run='none': Must be "none", "server", or "client". If client strategy, only print the object that would be
sent, without sending it. If server strategy, submit server-side request without persisting the resource.
      --field-manager='kubectl-create': Name of the manager used to track field ownership.
      --from='': The name of the resource to create a Job from (only cronjob is supported).
      --image='': Image name to run.
  -o, --output='': Output format. One of:
json|yaml|name|go-template|go-template-file|template|templatefile|jsonpath|jsonpath-as-json|jsonpath-file.
      --save-config=false: If true, the configuration of current object will be saved in its annotation. Otherwise, the
annotation will be unchanged. This flag is useful when you want to perform kubectl apply on this object in the future.
      --show-managed-fields=false: If true, keep the managedFields when printing objects in JSON or YAML format.
      --template='': Template string or path to template file to use when -o=go-template, -o=go-template-file. The
template format is golang templates [http://golang.org/pkg/text/template/#pkg-overview].
      --validate=true: If true, use a schema to validate the input before sending it

Usage:
  kubectl create job NAME --image=image [--from=cronjob/name] -- [COMMAND] [args...] [options]

Use "kubectl options" for a list of global command-line options (applies to all commands).
lab@k8s1:~$
```

```yaml
lab@k8s1:~$ cat job.yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: pi
spec:
  template:
    spec:
      containers:
      - name: pi
        image: perl
        command: ["perl",  "-Mbignum=bpi", "-wle", "print bpi(2000)"]
      restartPolicy: Never
  backoffLimit: 4
lab@k8s1:~$
```

```sh
lab@k8s1:~$ kubectl create -f job.yaml
job.batch/pi created
lab@k8s1:~$
```

```sh
lab@k8s1:~$ kubectl get job
NAME   COMPLETIONS   DURATION   AGE
pi     0/1           4s         4s
lab@k8s1:~$ kubectl get job
NAME   COMPLETIONS   DURATION   AGE
pi     0/1           6s         6s
lab@k8s1:~$ kubectl get job
NAME   COMPLETIONS   DURATION   AGE
pi     0/1           7s         7s

lab@k8s1:~$ kubectl get job
NAME   COMPLETIONS   DURATION   AGE
pi     1/1           36s        44s
lab@k8s1:~$
```

```sh
lab@k8s1:~$ kubectl describe job
Name:             pi
Namespace:        default
Selector:         controller-uid=b2835c58-d4fd-4d9f-b715-b26fdecd2c19
Labels:           controller-uid=b2835c58-d4fd-4d9f-b715-b26fdecd2c19
                  job-name=pi
Annotations:      <none>
Parallelism:      1
Completions:      1
Completion Mode:  NonIndexed
Start Time:       Fri, 01 Apr 2022 02:16:45 -0700
Completed At:     Fri, 01 Apr 2022 02:17:21 -0700
Duration:         36s
Pods Statuses:    0 Running / 1 Succeeded / 0 Failed
Pod Template:
  Labels:  controller-uid=b2835c58-d4fd-4d9f-b715-b26fdecd2c19
           job-name=pi
  Containers:
   pi:
    Image:      perl
    Port:       <none>
    Host Port:  <none>
    Command:
      perl
      -Mbignum=bpi
      -wle
      print bpi(2000)
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Events:
  Type    Reason            Age   From            Message
  ----    ------            ----  ----            -------
  Normal  SuccessfulCreate  58s   job-controller  Created pod: pi--1-vb4sm
  Normal  Completed         22s   job-controller  Job completed
lab@k8s1:~$
```



```sh
lab@k8s1:~$ kubectl describe pod pi--1-vb4sm
Name:         pi--1-vb4sm
Namespace:    default
Priority:     0
Node:         k8s3/10.210.40.175
Start Time:   Fri, 01 Apr 2022 02:16:45 -0700
Labels:       controller-uid=b2835c58-d4fd-4d9f-b715-b26fdecd2c19
              job-name=pi
Annotations:  <none>
Status:       Succeeded
IP:           10.244.3.15
IPs:
  IP:           10.244.3.15
Controlled By:  Job/pi
Containers:
  pi:
    Container ID:  docker://2fb0c18484eaa3eb211fc7ce480764101f08b835df12630083114e160de35f29
    Image:         perl
    Image ID:      docker-pullable://perl@sha256:e85f67a5e83029c49a4ce2a91fbfd86a4073d1d2c171a3fa8957d0d5f52a975c
    Port:          <none>
    Host Port:     <none>
    Command:
      perl
      -Mbignum=bpi
      -wle
      print bpi(2000)
    State:          Terminated
      Reason:       Completed
      Exit Code:    0
      Started:      Fri, 01 Apr 2022 02:17:16 -0700
      Finished:     Fri, 01 Apr 2022 02:17:20 -0700
    Ready:          False
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-kwcz6 (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             False
  ContainersReady   False
  PodScheduled      True
Volumes:
  kube-api-access-kwcz6:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age    From               Message
  ----    ------     ----   ----               -------
  Normal  Scheduled  4m54s  default-scheduler  Successfully assigned default/pi--1-vb4sm to k8s3
  Normal  Pulling    4m53s  kubelet            Pulling image "perl"
  Normal  Pulled     4m24s  kubelet            Successfully pulled image "perl" in 28.584816314s
  Normal  Created    4m24s  kubelet            Created container pi
  Normal  Started    4m23s  kubelet            Started container pi
lab@k8s1:~$
```



```sh
lab@k8s1:~$ kubectl delete jobs pi
job.batch "pi" deleted
lab@k8s1:~$ kubectl get jobs
No resources found in default namespace.
lab@k8s1:~$
```



Re-create the Job again.



```sh
lab@k8s1:~$ kubectl create -f job.yaml
job.batch/pi created
lab@k8s1:~$
```

```sh
lab@k8s1:~$ kubectl get jobs
NAME   COMPLETIONS   DURATION   AGE
pi     1/1           8s         14s
lab@k8s1:~$ kubectl get pods pi--1-hdwp2
NAME          READY   STATUS      RESTARTS   AGE
pi--1-hdwp2   0/1     Completed   0          27s
lab@k8s1:~$
```

```sh
lab@k8s1:~$ kubectl  logs pi--1-hdwp2
3.1415926535897932384626433832795028841971693993751058209749445923078164062862089986280348253421170679821480865132823066470938446095505822317253594081284811174502841027019385211055596446229489549303819644288109756659334461284756482337867831652712019091456485669234603486104543266482133936072602491412737245870066063155881748815209209628292540917153643678925903600113305305488204665213841469519415116094330572703657595919530921861173819326117931051185480744623799627495673518857527248912279381830119491298336733624406566430860213949463952247371907021798609437027705392171762931767523846748184676694051320005681271452635608277857713427577896091736371787214684409012249534301465495853710507922796892589235420199561121290219608640344181598136297747713099605187072113499999983729780499510597317328160963185950244594553469083026425223082533446850352619311881710100031378387528865875332083814206171776691473035982534904287554687311595628638823537875937519577818577805321712268066130019278766111959092164201989380952572010654858632788659361533818279682303019520353018529689957736225994138912497217752834791315155748572424541506959508295331168617278558890750983817546374649393192550604009277016711390098488240128583616035637076601047101819429555961989467678374494482553797747268471040475346462080466842590694912933136770289891521047521620569660240580381501935112533824300355876402474964732639141992726042699227967823547816360093417216412199245863150302861829745557067498385054945885869269956909272107975093029553211653449872027559602364806654991198818347977535663698074265425278625518184175746728909777727938000816470600161452491921732172147723501414419735685481613611573525521334757418494684385233239073941433345477624168625189835694855620992192221842725502542568876717904946016534668049886272327917860857843838279679766814541009538837863609506800642251252051173929848960841284886269456042419652850222106611863067442786220391949450471237137869609563643719172874677646575739624138908658326459958133904780275901
lab@k8s1:~$
```

```sh
lab@k8s1:~$ kubectl delete pods pi--1-hdwp2
pod "pi--1-hdwp2" deleted

```

```sh
lab@k8s1:~$ kubectl get job
NAME   COMPLETIONS   DURATION   AGE
pi     1/1           8s         2m10s
lab@k8s1:~
```

## CronJobs

A *CronJob* creates [Jobs](https://kubernetes.io/docs/concepts/workloads/controllers/job/) on a repeating schedule.

```sh
lab@k8s1:~$ kubectl create cronjob -h
Create a cron job with the specified name.

Aliases:
cronjob, cj

Examples:
  # Create a cron job
  kubectl create cronjob my-job --image=busybox --schedule="*/1 * * * *"

  # Create a cron job with a command
  kubectl create cronjob my-job --image=busybox --schedule="*/1 * * * *" -- date

Options:
      --allow-missing-template-keys=true: If true, ignore any errors in templates when a field or map key is missing in
the template. Only applies to golang and jsonpath output formats.
      --dry-run='none': Must be "none", "server", or "client". If client strategy, only print the object that would be
sent, without sending it. If server strategy, submit server-side request without persisting the resource.
      --field-manager='kubectl-create': Name of the manager used to track field ownership.
      --image='': Image name to run.
  -o, --output='': Output format. One of:
json|yaml|name|go-template|go-template-file|template|templatefile|jsonpath|jsonpath-as-json|jsonpath-file.
      --restart='': job's restart policy. supported values: OnFailure, Never
      --save-config=false: If true, the configuration of current object will be saved in its annotation. Otherwise, the
annotation will be unchanged. This flag is useful when you want to perform kubectl apply on this object in the future.
      --schedule='': A schedule in the Cron format the job should be run with.
      --show-managed-fields=false: If true, keep the managedFields when printing objects in JSON or YAML format.
      --template='': Template string or path to template file to use when -o=go-template, -o=go-template-file. The
template format is golang templates [http://golang.org/pkg/text/template/#pkg-overview].
      --validate=true: If true, use a schema to validate the input before sending it

Usage:
  kubectl create cronjob NAME --image=image --schedule='0/5 * * * ?' -- [COMMAND] [args...] [flags] [options]

Use "kubectl options" for a list of global command-line options (applies to all commands).
lab@k8s1:~$
```



```yaml
lab@k8s1:~$ cat cronjob.yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: hello
spec:
  schedule: "* * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: hello
            image: busybox:1.28
            imagePullPolicy: IfNotPresent
            command:
            - /bin/sh
            - -c
            - date; echo Hello from the Kubernetes cluster
          restartPolicy: OnFailure
lab@k8s1:~$
```

This example CronJob manifest prints the current time and a hello message every minute:

```sh
lab@k8s1:~$ kubectl create -f cronjob.yaml
cronjob.batch/hello created
lab@k8s1:~$
```

```sh
lab@k8s1:~$ kubectl get cronjob
NAME    SCHEDULE    SUSPEND   ACTIVE   LAST SCHEDULE   AGE
hello   * * * * *   False     0        <none>          5s
```

```sh
lab@k8s1:~$ kubectl describe cronjob
Name:                          hello
Namespace:                     default
Labels:                        <none>
Annotations:                   <none>
Schedule:                      * * * * *
Concurrency Policy:            Allow
Suspend:                       False
Successful Job History Limit:  3
Failed Job History Limit:      1
Starting Deadline Seconds:     <unset>
Selector:                      <unset>
Parallelism:                   <unset>
Completions:                   <unset>
Pod Template:
  Labels:  <none>
  Containers:
   hello:
    Image:      busybox:1.28
    Port:       <none>
    Host Port:  <none>
    Command:
      /bin/sh
      -c
      date; echo Hello from the Kubernetes cluster
    Environment:     <none>
    Mounts:          <none>
  Volumes:           <none>
Last Schedule Time:  Fri, 01 Apr 2022 02:33:00 -0700
Active Jobs:         <none>
Events:
  Type    Reason            Age   From                Message
  ----    ------            ----  ----                -------
  Normal  SuccessfulCreate  23s   cronjob-controller  Created job hello-27480093
  Normal  SawCompletedJob   19s   cronjob-controller  Saw completed job: hello-27480093, status: Complete
lab@k8s1:~$
```



After some more time,



```sh
lab@k8s1:~$ kubectl describe cronjob
Name:                          hello
Namespace:                     default
Labels:                        <none>
Annotations:                   <none>
Schedule:                      * * * * *
Concurrency Policy:            Allow
Suspend:                       False
Successful Job History Limit:  3
Failed Job History Limit:      1
Starting Deadline Seconds:     <unset>
Selector:                      <unset>
Parallelism:                   <unset>
Completions:                   <unset>
Pod Template:
  Labels:  <none>
  Containers:
   hello:
    Image:      busybox:1.28
    Port:       <none>
    Host Port:  <none>
    Command:
      /bin/sh
      -c
      date; echo Hello from the Kubernetes cluster
    Environment:     <none>
    Mounts:          <none>
  Volumes:           <none>
Last Schedule Time:  Fri, 01 Apr 2022 02:38:00 -0700
Active Jobs:         <none>
Events:
  Type    Reason            Age    From                Message
  ----    ------            ----   ----                -------
  Normal  SuccessfulCreate  5m57s  cronjob-controller  Created job hello-27480093
  Normal  SawCompletedJob   5m53s  cronjob-controller  Saw completed job: hello-27480093, status: Complete
  Normal  SuccessfulCreate  4m57s  cronjob-controller  Created job hello-27480094
  Normal  SawCompletedJob   4m56s  cronjob-controller  Saw completed job: hello-27480094, status: Complete
  Normal  SuccessfulCreate  3m57s  cronjob-controller  Created job hello-27480095
  Normal  SawCompletedJob   3m56s  cronjob-controller  Saw completed job: hello-27480095, status: Complete
  Normal  SuccessfulCreate  2m57s  cronjob-controller  Created job hello-27480096
  Normal  SuccessfulDelete  2m56s  cronjob-controller  Deleted job hello-27480093
  Normal  SawCompletedJob   2m56s  cronjob-controller  Saw completed job: hello-27480096, status: Complete
  Normal  SuccessfulCreate  117s   cronjob-controller  Created job hello-27480097
  Normal  SawCompletedJob   116s   cronjob-controller  Saw completed job: hello-27480097, status: Complete
  Normal  SuccessfulDelete  116s   cronjob-controller  Deleted job hello-27480094
  Normal  SuccessfulCreate  57s    cronjob-controller  Created job hello-27480098
  Normal  SawCompletedJob   56s    cronjob-controller  Saw completed job: hello-27480098, status: Complete
  Normal  SuccessfulDelete  56s    cronjob-controller  Deleted job hello-27480095
lab@k8s1:~$
```
We can see every minute, a new job gets created, and hence a new pod as well.
But from the jobs list, we see only the latest 3 jobs.
```sh
lab@k8s1:~$ kubectl get jobs
NAME             COMPLETIONS   DURATION   AGE
hello-27480097   1/1           1s         2m21s
hello-27480098   1/1           1s         81s
hello-27480099   1/1           1s         21s
pi               1/1           8s         14m
lab@k8s1:~$
```
Similarly, there are only three latest Pods.

```sh
lab@k8s1:~$ kubectl get pods
NAME                      READY   STATUS      RESTARTS   AGE
hello-27480097--1-rqzxj   0/1     Completed   0          2m44s
hello-27480098--1-lkj9b   0/1     Completed   0          104s
hello-27480099--1-nsqpw   0/1     Completed   0          44s
web-79d88c97d6-4w4nl      1/1     Running     0          2d15h
web-79d88c97d6-7ktck      1/1     Running     0          2d15h
web-79d88c97d6-8tvsk      1/1     Running     0          2d15h
web-79d88c97d6-96sd9      1/1     Running     0          2d15h
web-79d88c97d6-9tzlh      1/1     Running     0          2d15h
web-79d88c97d6-brtgx      1/1     Running     0          2d15h
web-79d88c97d6-kngc4      1/1     Running     0          2d15h
web-79d88c97d6-p5vfg      1/1     Running     0          2d15h
web-79d88c97d6-rbhpr      1/1     Running     0          2d15h
lab@k8s1:~$
```
We can look at logs from these pods.

```sh
lab@k8s1:~$ kubectl logs hello-27480098--1-lkj9b
Fri Apr  1 09:38:01 UTC 2022
Hello from the Kubernetes cluster
lab@k8s1:~$
```

```sh
lab@k8s1:~$ kubectl logs hello-27480099--1-nsqpw
Fri Apr  1 09:39:01 UTC 2022
Hello from the Kubernetes cluster
lab@k8s1:~$
```



```sh
lab@k8s1:~$ kubectl delete cronjob hello
cronjob.batch "hello" deleted
lab@k8s1:~$ kubectl get job
NAME   COMPLETIONS   DURATION   AGE
pi     1/1           8s         21m
lab@k8s1:~$ kubectl get cronjob
No resources found in default namespace.
lab@k8s1:~$ kubectl delete job pi
job.batch "pi" deleted
lab@k8s1:~$
```

```sh
lab@k8s1:~$ kubectl get cronjob,job
No resources found in default namespace.
lab@k8s1:~$
```

This concludes our discussion on Kubernetes Jobs and CronJobs.
