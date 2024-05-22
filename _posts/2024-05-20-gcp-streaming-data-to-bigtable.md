---

layout: single
title:  "Streaming Data to Bigtable"
categories: Cloud
tags: GCP
classes: wide
show_date: true
header:
  overlay_image: /assets/images/gcp-banner.png
  og_image: /assets/images/gcp-banner.png
  teaser: /assets/images/ace-gcp.png
author:
  name     : "Associate Cloud Engineer"
  avatar   : "/assets/images/ace-gcp.png"

sidebar:
  - title: "Topics"
    nav: my-sidebar

---
# Streaming Data to Bigtable

[Bigtable](https://cloud.google.com/bigtable/) is Google's fully managed, scalable NoSQL database service. Bigtable is ideal for storing large amounts of data in a key-value store and for  use cases such as personalization, ad tech, financial tech, digital  media, and Internet of Things (IoT). Bigtable supports high read and  write throughput at low latency for fast access to large amounts of data for processing and analytics.

For streaming data from sensors, Bigtable can handle high writes to capture large volumes of real-time data.

In this lab, you use commands to create a Bigtable instance with a  table to store simulated traffic sensor data. Then you launch a Dataflow pipeline to load the simulated streaming data from Pub/Sub into  Bigtable. While the Dataflow job loads streaming data from Pub/Sub into  Bigtable, you verify that the table is being successfully populated. You complete the lab by stopping the streaming job and deleting the  Bigtable data.

- Create a Bigtable instance using Google Cloud CLI (`gcloud` CLI) commands.
- Create a Bigtable table with column families using Cloud Bigtable CLI (`cbt` CLI) commands.
- Launch a Dataflow pipeline to read streaming data from Pub/Sub and write into Bigtable.
- Verify the streaming data loaded into Bigtable.
- Delete a Bigtable table and a Bigtable instance using commands.

## Create a Bigtable instance and table using commands

To create a new table in Bigtable, you first need to create a  Bigtable instance to store your table. To create a Bigtable instance,  you can use the Google Cloud console, [`gcloud` CLI commands](https://cloud.google.com/bigtable/docs/creating-instance#gcloud), or [`cbt` CLI commands](https://cloud.google.com/bigtable/docs/creating-instance#cbt).

In this task, you use Cloud Shell to first run `gcloud` CLI commands to create a new Bigtable instance, and then run `cbt` CLI commands to connect to Bigtable and create a new table.

```sh
Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-00-9a9345cbcce6.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_00_e5bcc3b525da@cloudshell:~ (qwiklabs-gcp-00-9a9345cbcce6)$ gcloud bigtable instances create sandiego \
--display-name="San Diego Traffic Sensors" \
--cluster-storage-type=SSD \
--cluster-config=id=sandiego-traffic-sensors-c1,zone=us-east5-a,nodes=1
Creating bigtable instance sandiego...done.                                                                                                                                        
student_00_e5bcc3b525da@cloudshell:~ (qwiklabs-gcp-00-9a9345cbcce6)$ 
```

### Configure the Bigtable CLI

To connect to Bigtable using `cbt` CLI commands, you first need to use Cloud Shell to update the `.cbtrc` configuration file with your project ID and your Bigtable instance ID.

1. To modify the `.cbtrc` file with the project ID and instance ID, run the following commands:

```sh
student_00_e5bcc3b525da@cloudshell:~ (qwiklabs-gcp-00-9a9345cbcce6)$ echo project = `gcloud config get-value project` \
    >> ~/.cbtrc
Your active configuration is: [cloudshell-1341]
student_00_e5bcc3b525da@cloudshell:~ (qwiklabs-gcp-00-9a9345cbcce6)$ echo instance = sandiego \
    >> ~/.cbtrc
student_00_e5bcc3b525da@cloudshell:~ (qwiklabs-gcp-00-9a9345cbcce6)$ cat ~/.cbtrc 
project = qwiklabs-gcp-00-9a9345cbcce6
instance = sandiego
student_00_e5bcc3b525da@cloudshell:~ (qwiklabs-gcp-00-9a9345cbcce6)$ 
```

### Create a Bigtable table with column families

After you configure the `.cbtrc` configuration file in Cloud Shell, you can run a simple `cbt` CLI command to create a new Bigtable table with column families.

- To create a new table named **current_conditions** with one column family named **lane**, run the following command:

```sh
student_00_e5bcc3b525da@cloudshell:~ (qwiklabs-gcp-00-9a9345cbcce6)$ cbt createtable current_conditions \
    families="lane"
2024/05/22 02:11:13 -creds flag unset, will use gcloud credential
student_00_e5bcc3b525da@cloudshell:~ (qwiklabs-gcp-00-9a9345cbcce6)$ cbt ls
2024/05/22 02:11:20 -creds flag unset, will use gcloud credential
current_conditions
student_00_e5bcc3b525da@cloudshell:~ (qwiklabs-gcp-00-9a9345cbcce6)$ 
```

## Simulate streaming traffic sensor data into Pub/Sub

In this task, you run a streaming data simulator from a Compute  Engine virtual machine (VM) that has been created for this lab. To begin this task, you will enter commands on a VM named **training-vm** to set up your environment and download the necessary files for the streaming data simulator.

### Connect to the VM

1. In the Google Cloud console, on the **Navigation menu**, click **Compute Engine > VM instances**.

2. Locate the line with the instance called **training-vm**, and under **Connect**, click **SSH**.

   A terminal window for **training-vm** will open.

   The **training-vm** is installing some software in the  background. In the next step, you verify that setup is complete by  checking the contents of the new directory.

3. To list the contents of the directory named **training**, run the following command:

```sh
Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-00-9a9345cbcce6.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_00_e5bcc3b525da@cloudshell:~ (qwiklabs-gcp-00-9a9345cbcce6)$ gcloud compute ssh --zone "us-central1-f" "training-vm" --project "qwiklabs-gcp-00-9a9345cbcce6"
WARNING: The private SSH key file for gcloud does not exist.
WARNING: The public SSH key file for gcloud does not exist.
WARNING: You do not have an SSH key for gcloud.
WARNING: SSH keygen will be executed to generate a key.
This tool needs to create the directory [/home/student_00_e5bcc3b525da/.ssh] before being able to generate SSH keys.

Do you want to continue (Y/n)?  y

Generating public/private rsa key pair.
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/student_00_e5bcc3b525da/.ssh/google_compute_engine
Your public key has been saved in /home/student_00_e5bcc3b525da/.ssh/google_compute_engine.pub
The key fingerprint is:
SHA256:9/hHaSkOFtG9qq4r4LeJKUa4CfciXVfE8G94uNy7wYc student_00_e5bcc3b525da@cs-808072148539-default
The key's randomart image is:
+---[RSA 3072]----+
|      .o   . .   |
|       .o . . .  |
|       ..  .   . |
|        .+.   .  |
| .     .S =. . o |
|o o o .. Bo+o =  |
|.* + o  o.E+o+   |
|+ = ooo.  .=. .  |
| o oo.ooo++...   |
+----[SHA256]-----+
Warning: Permanently added 'compute.3648272316701388484' (ED25519) to the list of known hosts.
Linux training-vm 4.9.0-12-amd64 #1 SMP Debian 4.9.210-1 (2020-01-20) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
student-00-e5bcc3b525da@training-vm:~$ 
```



```sh
student-00-e5bcc3b525da@training-vm:~$ ls /training
bq_magic.sh  project_env.sh  sensor_magic.sh
student-00-e5bcc3b525da@training-vm:~$ cat /training/bq_magic.sh 
#! /bin/bash

bq mk --dataset $DEVSHELL_PROJECT_ID:demos

bq load --skip_leading_rows=1 --source_format=CSV demos.average_speeds gs://cloud-training/gcpdei/results-20180608-102960.csv timestamp:TIMESTAMP,latitude:FLOAT,longitude:FLOAT,highway:STRING,direction:STRING,lane:INTEGER,speed:FLOAT,sensorId:STRING
bq load --skip_leading_rows=1 --source_format=CSV demos.current_conditions gs://cloud-training/gcpdei/results-20180608-102960.csv timestamp:TIMESTAMP,latitude:FLOAT,longitude:FLOAT,highway:STRING,direction:STRING,lane:INTEGER,speed:FLOAT,sensorId:STRING
student-00-e5bcc3b525da@training-vm:~$ cat /training/project_env.sh 
#! /bin/bash

# Create the DEVSHELL_PROJECT_ID on a VM
curl "http://metadata.google.internal/computeMetadata/v1/project/project-id" -H "Metadata-Flavor: Google" > Project_ID
awk '{print "export DEVSHELL_PROJECT_ID=" $0, "\n" "export BUCKET=" $0, "\n" "export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64/jre" }' Project_ID > env.txt
source env.txt
echo $DEVSHELL_PROJECT_ID


student-00-e5bcc3b525da@training-vm:~$ cat /training/sensor_magic.sh 
#! /bin/bash

# User tasks:
#  1. copy repo to ~/training-data-analyst
#  2. create $DEVSHELL_PROJECT_ID
#
# Install PIP
# sudo apt-get install -y python-pip
# Use PIP to install pubsub API
# sudo pip install -U google-cloud-pubsub
# Download the data file
gsutil cp gs://cloud-training-demos/sandiego/sensor_obs2008.csv.gz ~/training-data-analyst/courses/streaming/publish/
# cd to directory
cd ~/training-data-analyst/courses/streaming/publish/
# Run sensor simulator
python3 ./send_sensor_data.py --speedFactor=60 --project $DEVSHELL_PROJECT_IDstudent-00-e5bcc3b525da@training-vm:~$ 
```

### Run script to simulate streaming data

1. To download a code repository for use in this lab, run the following command:

```sh
student-00-e5bcc3b525da@training-vm:~$ git clone https://github.com/GoogleCloudPlatform/training-data-analyst
Cloning into 'training-data-analyst'...
remote: Enumerating objects: 64939, done.
remote: Counting objects: 100% (76/76), done.
remote: Compressing objects: 100% (42/42), done.
remote: Total 64939 (delta 35), reused 71 (delta 34), pack-reused 64863
Receiving objects: 100% (64939/64939), 697.16 MiB | 34.15 MiB/s, done.
Resolving deltas: 100% (41501/41501), done.
student-00-e5bcc3b525da@training-vm:~$ source /training/project_env.sh
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100    28  100    28    0     0   9810      0 --:--:-- --:--:-- --:--:-- 14000
qwiklabs-gcp-00-9a9345cbcce6
student-00-e5bcc3b525da@training-vm:~$ /training/sensor_magic.sh
Copying gs://cloud-training-demos/sandiego/sensor_obs2008.csv.gz...
- [1 files][ 34.6 MiB/ 34.6 MiB]                                                
Operation completed over 1 objects/34.6 MiB.                                     
INFO: Creating pub/sub topic sandiego
INFO: Sending sensor data from 2008-11-01 00:00:00
INFO: Publishing 477 events from 2008-11-01 00:00:00
INFO: Sleeping 5.0 seconds
INFO: Publishing 477 events from 2008-11-01 00:05:00
INFO: Sleeping 5.0 seconds
INFO: Publishing 477 events from 2008-11-01 00:10:00
INFO: Sleeping 5.0 seconds
INFO: Publishing 477 events from 2008-11-01 00:15:00
INFO: Sleeping 5.0 seconds
INFO: Publishing 477 events from 2008-11-01 00:20:00
INFO: Sleeping 5.0 seconds
INFO: Publishing 477 events from 2008-11-01 00:25:00
INFO: Sleeping 5.0 seconds

```

This script reads sample data from a CSV file and publishes it to Pub/Sub. This script will send one hour of data in one minute.

Let the script continue to run in the current terminal, and continue with the next tasks.

## Launch a Dataflow pipeline to write data from Pub/Sub into Bigtable

In this task, you open a second SSH terminal on **training_vm** and run commands to launch a Dataflow job to write streaming data from Pub/Sub into Bigtable.

This script sets the `$DEVSHELL_PROJECT_ID` and `$BUCKET` environment variables in the new terminal window.

### Launch a Dataflow Pipeline

1. To navigate to the code directory in the new terminal, run the following command:

```sh
student_00_e5bcc3b525da@cloudshell:~ (qwiklabs-gcp-00-9a9345cbcce6)$ gcloud compute ssh --zone "us-central1-f" "training-vm" --project "qwiklabs-gcp-00-9a9345cbcce6"
Linux training-vm 4.9.0-12-amd64 #1 SMP Debian 4.9.210-1 (2020-01-20) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Wed May 22 02:12:45 2024 from 34.87.33.251
student-00-e5bcc3b525da@training-vm:~$ source /training/project_env.sh
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100    28  100    28    0     0   4274      0 --:--:-- --:--:-- --:--:--  4666
qwiklabs-gcp-00-9a9345cbcce6
student-00-e5bcc3b525da@training-vm:~$ cd ~/training-data-analyst/courses/streaming/process/sandiego
student-00-e5bcc3b525da@training-vm:~/training-data-analyst/courses/streaming/process/sandiego$ cat run_oncloud.sh 
#!/bin/bash

if [ "$#" -lt 3 ]; then
   echo "Usage:   ./run_oncloud.sh project-name bucket-name classname [options] "
   echo "Example: ./run_oncloud.sh cloud-training-demos cloud-training-demos CurrentConditions --bigtable"
   exit
fi

PROJECT=$1
shift
BUCKET=$1
shift
MAIN=com.google.cloud.training.dataanalyst.sandiego.$1
shift

echo "Launching $MAIN project=$PROJECT bucket=$BUCKET $*"

export PATH=/usr/lib/jvm/java-8-openjdk-amd64/bin/:$PATH
mvn compile -e exec:java \
 -Dexec.mainClass=$MAIN \
      -Dexec.args="--project=$PROJECT \
      --stagingLocation=gs://$BUCKET/staging/ $* \
      --tempLocation=gs://$BUCKET/staging/ \
      --region=$REGION \
      --workerMachineType=e2-standard-2 \
      --runner=DataflowRunner"


# If you run into quota problems, add this option the command line above
#     --maxNumWorkers=2 
# In this case, you will not be able to view autoscaling, however.
student-00-e5bcc3b525da@training-vm:~/training-data-analyst/courses/streaming/process/sandiego$ 
```

This script takes three required arguments to run a Dataflow job:

- Project ID
- Cloud Storage bucket name
- Java classname
- optional fourth argument for options

In the next steps, you use the `--bigtable` option to direct the Dataflow pipeline to write data into Bigtable.

To launch the Dataflow pipeline to read from Pub/Sub and write into Bigtable, run the following command:

```sh
student-00-e5bcc3b525da@training-vm:~/training-data-analyst/courses/streaming/process/sandiego$ ./run_oncloud.sh $DEVSHELL_PROJECT_ID $BUCKET CurrentConditions --bigtable
{snip}
INFO: Adding BigQueryIO.Write/StreamingInserts/StreamingWriteTables/ShardTableWrites as step s9
May 22, 2024 2:23:04 AM org.apache.beam.runners.dataflow.DataflowPipelineTranslator$Translator addStep
INFO: Adding BigQueryIO.Write/StreamingInserts/StreamingWriteTables/TagWithUniqueIds as step s10
May 22, 2024 2:23:04 AM org.apache.beam.runners.dataflow.DataflowPipelineTranslator$Translator addStep
INFO: Adding BigQueryIO.Write/StreamingInserts/StreamingWriteTables/Reshuffle/Window.Into()/Window.Assign as step s11
May 22, 2024 2:23:04 AM org.apache.beam.runners.dataflow.DataflowPipelineTranslator$Translator addStep
INFO: Adding BigQueryIO.Write/StreamingInserts/StreamingWriteTables/Reshuffle/GroupByKey as step s12
May 22, 2024 2:23:04 AM org.apache.beam.runners.dataflow.DataflowPipelineTranslator$Translator addStep
INFO: Adding BigQueryIO.Write/StreamingInserts/StreamingWriteTables/Reshuffle/ExpandIterable as step s13
May 22, 2024 2:23:04 AM org.apache.beam.runners.dataflow.DataflowPipelineTranslator$Translator addStep
INFO: Adding BigQueryIO.Write/StreamingInserts/StreamingWriteTables/GlobalWindow/Window.Assign as step s14
May 22, 2024 2:23:04 AM org.apache.beam.runners.dataflow.DataflowPipelineTranslator$Translator addStep
INFO: Adding BigQueryIO.Write/StreamingInserts/StreamingWriteTables/StreamingWrite as step s15
May 22, 2024 2:23:04 AM org.apache.beam.runners.dataflow.DataflowRunner run
INFO: Staging pipeline description to gs://qwiklabs-gcp-00-9a9345cbcce6/staging/
May 22, 2024 2:23:04 AM org.apache.beam.runners.dataflow.util.PackageUtil tryStagePackage
INFO: Uploading <44472 bytes, hash tgtISjBOQ_j9YktVuqr0pQ> to gs://qwiklabs-gcp-00-9a9345cbcce6/staging/pipeline-tgtISjBOQ_j9YktVuqr0pQ.pb
May 22, 2024 2:23:04 AM org.apache.beam.runners.dataflow.DataflowRunner run
INFO: Dataflow SDK version: 2.20.0
May 22, 2024 2:23:05 AM org.apache.beam.runners.dataflow.DataflowRunner run
INFO: To access the Dataflow monitoring console, please navigate to https://console.cloud.google.com/dataflow/jobs//2024-05-21_19_23_04-2027131398950959761?project=qwiklabs-gcp-00-9a9345cbcce6
May 22, 2024 2:23:05 AM org.apache.beam.runners.dataflow.DataflowRunner run
INFO: Submitted job: 2024-05-21_19_23_04-2027131398950959761
May 22, 2024 2:23:05 AM org.apache.beam.runners.dataflow.DataflowRunner run
INFO: To cancel the job using the 'gcloud' tool, run:
> gcloud dataflow jobs --project=qwiklabs-gcp-00-9a9345cbcce6 cancel --region= 2024-05-21_19_23_04-2027131398950959761
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 30.391 s
[INFO] Finished at: 2024-05-22T02:23:05+00:00
[INFO] Final Memory: 81M/782M
[INFO] ------------------------------------------------------------------------
student-00-e5bcc3b525da@training-vm:~/training-data-analyst/courses/streaming/process/sandiego$ 
```

### Explore the Dataflow pipeline

1. In the Google Cloud console, on the **Navigation menu**, under **Analytics**, click **Dataflow > Jobs**.
2. Click on the new pipeline job name.
3. Locate the **write:cbt** step in the pipeline graph, and to see the details of the writer, click on the down arrow next to **write:cbt**.
4. Click on the provided writer, and review the details provided within **Step info**.

## Verify streaming data loaded into Bigtable

In a previous task, you already configured the `.cbtrc` configuration file in Cloud Shell. You can now run a simple `cbt` CLI command to query the first five records of the table.

- To see the first five rows of data and their values in the **lane** column family, run the following command:

```sh
student_00_e5bcc3b525da@cloudshell:~ (qwiklabs-gcp-00-9a9345cbcce6)$ cbt read current_conditions count=5     columns="lane:.*"
2024/05/22 02:26:15 -creds flag unset, will use gcloud credential
----------------------------------------
15#S#1#9223370811322975807
  lane:direction                           @ 1970/01/15-04:25:31.800000
    "S"
  lane:highway                             @ 1970/01/15-04:25:31.800000
    "15"
  lane:lane                                @ 1970/01/15-04:25:31.800000
    "1.0"
  lane:latitude                            @ 1970/01/15-04:25:31.800000
    "32.706184"
  lane:longitude                           @ 1970/01/15-04:25:31.800000
    "-117.120565"
  lane:sensorId                            @ 1970/01/15-04:25:31.800000
    "32.706184,-117.120565,15,S,1"
  lane:speed                               @ 1970/01/15-04:25:31.800000
    "73.1"
  lane:timestamp                           @ 1970/01/15-04:25:31.800000
    "2008-11-01 09:30:00"

----------------------------------------
15#S#1#9223370811323275807
  lane:direction                           @ 1970/01/15-04:25:31.500000
    "S"
  lane:highway                             @ 1970/01/15-04:25:31.500000
    "15"
  lane:lane                                @ 1970/01/15-04:25:31.500000
    "1.0"
  lane:latitude                            @ 1970/01/15-04:25:31.500000
    "32.723248"
  lane:longitude                           @ 1970/01/15-04:25:31.500000
    "-117.115543"
  lane:sensorId                            @ 1970/01/15-04:25:31.500000
    "32.723248,-117.115543,15,S,1"
  lane:speed                               @ 1970/01/15-04:25:31.500000
    "71.2"
  lane:timestamp                           @ 1970/01/15-04:25:31.500000
    "2008-11-01 09:25:00"

----------------------------------------
15#S#1#9223370811323575807
  lane:direction                           @ 1970/01/15-04:25:31.200000
    "S"
  lane:highway                             @ 1970/01/15-04:25:31.200000
    "15"
  lane:lane                                @ 1970/01/15-04:25:31.200000
    "1.0"
  lane:latitude                            @ 1970/01/15-04:25:31.200000
    "32.706184"
  lane:longitude                           @ 1970/01/15-04:25:31.200000
    "-117.120565"
  lane:sensorId                            @ 1970/01/15-04:25:31.200000
    "32.706184,-117.120565,15,S,1"
  lane:speed                               @ 1970/01/15-04:25:31.200000
    "72.9"
  lane:timestamp                           @ 1970/01/15-04:25:31.200000
    "2008-11-01 09:20:00"

----------------------------------------
15#S#1#9223370811323875807
  lane:direction                           @ 1970/01/15-04:25:30.900000
    "S"
  lane:highway                             @ 1970/01/15-04:25:30.900000
    "15"
  lane:lane                                @ 1970/01/15-04:25:30.900000
    "1.0"
  lane:latitude                            @ 1970/01/15-04:25:30.900000
    "32.706184"
  lane:longitude                           @ 1970/01/15-04:25:30.900000
    "-117.120565"
  lane:sensorId                            @ 1970/01/15-04:25:30.900000
    "32.706184,-117.120565,15,S,1"
  lane:speed                               @ 1970/01/15-04:25:30.900000
    "73.0"
  lane:timestamp                           @ 1970/01/15-04:25:30.900000
    "2008-11-01 09:15:00"

----------------------------------------
15#S#1#9223370811324175807
  lane:direction                           @ 1970/01/15-04:25:30.600000
    "S"
  lane:highway                             @ 1970/01/15-04:25:30.600000
    "15"
  lane:lane                                @ 1970/01/15-04:25:30.600000
    "1.0"
  lane:latitude                            @ 1970/01/15-04:25:30.600000
    "32.706184"
  lane:longitude                           @ 1970/01/15-04:25:30.600000
    "-117.120565"
  lane:sensorId                            @ 1970/01/15-04:25:30.600000
    "32.706184,-117.120565,15,S,1"
  lane:speed                               @ 1970/01/15-04:25:30.600000
    "73.0"
  lane:timestamp                           @ 1970/01/15-04:25:30.600000
    "2008-11-01 09:10:00"

student_00_e5bcc3b525da@cloudshell:~ (qwiklabs-gcp-00-9a9345cbcce6)$ 
```

## Stop streaming jobs and delete Bigtable data

In this final task, you stop the streaming data job and delete the Bigtable instance and table using commands.

### Stop simulated streaming data

1. In the first SSH terminal with the streaming data simulator, to stop the simulation, press CONTROL+C.

### Stop the Dataflow job

1. In the Google Cloud console, on the **Navigation menu**, click **Dataflow > Jobs**.
2. Click on the pipeline job name.
3. Click **Stop**.
4. Select **Cancel**, and then click **Stop job**.

### Delete a Bigtable table and instance

1. To delete the Bigtable table, in Cloud Shell, run the following command:

```sh
student_00_e5bcc3b525da@cloudshell:~ (qwiklabs-gcp-00-9a9345cbcce6)$ cbt deletetable current_conditions
2024/05/22 02:28:40 -creds flag unset, will use gcloud credential
student_00_e5bcc3b525da@cloudshell:~ (qwiklabs-gcp-00-9a9345cbcce6)$ gcloud bigtable instances delete sandiego
Delete instance sandiego. Are you sure?

Do you want to continue (Y/n)?  y

student_00_e5bcc3b525da@cloudshell:~ (qwiklabs-gcp-00-9a9345cbcce6)$ 
```



## History

```sh
student_00_e5bcc3b525da@cloudshell:~ (qwiklabs-gcp-00-9a9345cbcce6)$ history 
    1  gcloud bigtable instances create sandiego --display-name="San Diego Traffic Sensors" --cluster-storage-type=SSD --cluster-config=id=sandiego-traffic-sensors-c1,zone=us-east5-a,nodes=1
    2  echo project = `gcloud config get-value project`     >> ~/.cbtrc
    3  echo instance = sandiego     >> ~/.cbtrc
    4  cat ~/.cbtrc 
    5  cbt createtable current_conditions     families="lane"
    6  cbt ls
    7  ls
    8  cbt read current_conditions count=5     columns="lane:.*"
    9  cbt deletetable current_conditions
   10  gcloud bigtable instances delete sandiego
   11  history 
student_00_e5bcc3b525da@cloudshell:~ (qwiklabs-gcp-00-9a9345cbcce6)$ 
```

```sh
student-00-e5bcc3b525da@training-vm:~$ history 
    1  ls /training
    2  cat /training/bq_magic.sh 
    3  cat /training/project_env.sh 
    4  cat /training/sensor_magic.sh 
    5  git clone https://github.com/GoogleCloudPlatform/training-data-analyst
    6  source /training/project_env.sh
    7  /training/sensor_magic.sh
    8  history 
student-00-e5bcc3b525da@training-vm:~$ 
```

```sh
student-00-e5bcc3b525da@training-vm:~/training-data-analyst/courses/streaming/process/sandiego$ history 
    1  source /training/project_env.sh
    2  cd ~/training-data-analyst/courses/streaming/process/sandiego
    3  cat run_oncloud.sh 
    4  ./run_oncloud.sh $DEVSHELL_PROJECT_ID $BUCKET CurrentConditions --bigtable
    5  history 
student-00-e5bcc3b525da@training-vm:~/training-data-analyst/courses/streaming/process/sandiego$ 
```

