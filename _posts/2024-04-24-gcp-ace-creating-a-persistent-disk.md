---
layout: single
title:  "Creating a Persistent Disk in GCP"
categories: Cloud
tags: GCP
classes: wide
show_date: true
header:
  teaser: /assets/images/ace-gcp.png
author:
  name     : "Associate Cloud Engineer"
  avatar   : "/assets/images/ace-gcp.png"

sidebar:
  - title: "Topics"
    nav: my-sidebar

---

# Creating a Persistent Disk in GCP
Compute Engine provides persistent disks for use as the primary storage for your virtual machine instances. Like physical hard drives, persistent disks exist independently of the rest of your machine – if a virtual machine instance is deleted, the attached persistent disk continues to retain its data and can be attached to another instance. You can use persistent disks to setup and configure your database servers.

- Create a new VM instance and attach a persistent disk
- Format and mount a persistent disk

## Create a new instance

First, create a Compute Engine virtual machine instance that has only a boot disk.
```sh
Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-04-289d748a3fdf.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_01_9b51dda9fd9c@cloudshell:~ (qwiklabs-gcp-04-289d748a3fdf)$ gcloud config set compute/zone us-central1-c
gcloud config set compute/region us-central1
WARNING: Property validation for compute/zone was skipped.
Updated property [compute/zone].
WARNING: Property validation for compute/region was skipped.
Updated property [compute/region].
student_01_9b51dda9fd9c@cloudshell:~ (qwiklabs-gcp-04-289d748a3fdf)$ export REGION=us-central1
student_01_9b51dda9fd9c@cloudshell:~ (qwiklabs-gcp-04-289d748a3fdf)$ export ZONE=us-central1-c
student_01_9b51dda9fd9c@cloudshell:~ (qwiklabs-gcp-04-289d748a3fdf)$ 
student_01_9b51dda9fd9c@cloudshell:~ (qwiklabs-gcp-04-289d748a3fdf)$ gcloud compute instances create gcelab --zone $ZONE --machine-type e2-standard-2
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-289d748a3fdf/zones/us-central1-c/instances/gcelab].
NAME: gcelab
ZONE: us-central1-c
MACHINE_TYPE: e2-standard-2
PREEMPTIBLE: 
INTERNAL_IP: 10.128.0.2
EXTERNAL_IP: 34.69.205.108
STATUS: RUNNING
student_01_9b51dda9fd9c@cloudshell:~ (qwiklabs-gcp-04-289d748a3fdf)$ 
```

The newly created virtual machine instance will have a default 10 GB persistent disk as the boot disk.

## Create a new persistent disk

Because you want to attach this disk to the virtual machine instance you created in the previous step, the zone must be the same.

create a new disk named `mydisk`:

```sh
student_01_9b51dda9fd9c@cloudshell:~ (qwiklabs-gcp-04-289d748a3fdf)$ gcloud compute disks create mydisk --size=200GB \
--zone $ZONE
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-289d748a3fdf/zones/us-central1-c/disks/mydisk].
NAME: mydisk
ZONE: us-central1-c
SIZE_GB: 200
TYPE: pd-standard
STATUS: READY

New disks are unformatted. You must format and mount a disk before it
can be used. You can find instructions on how to do this at:

https://cloud.google.com/compute/docs/disks/add-persistent-disk#formatting

student_01_9b51dda9fd9c@cloudshell:~ (qwiklabs-gcp-04-289d748a3fdf)$ 
```



## Attaching a disk

### Attaching the persistent disk

You can attach a disk to a running virtual machine. Attach the new disk (`mydisk`) to the virtual machine instance you just created (`gcelab`).

1. Use the following command to attach the disk:

```sh
student_01_9b51dda9fd9c@cloudshell:~ (qwiklabs-gcp-04-289d748a3fdf)$ gcloud compute instances attach-disk gcelab --disk mydisk --zone $ZONE
Updated [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-289d748a3fdf/zones/us-central1-c/instances/gcelab].
student_01_9b51dda9fd9c@cloudshell:~ (qwiklabs-gcp-04-289d748a3fdf)$ 
```

### Finding the persistent disk in the virtual machine

The persistent disk is now available as a block device in the virtual machine instance. Let's take a look.

1. SSH into the virtual machine:

   ```sh
   student_01_9b51dda9fd9c@cloudshell:~ (qwiklabs-gcp-04-289d748a3fdf)$ gcloud compute ssh gcelab --zone $ZONE
   WARNING: The private SSH key file for gcloud does not exist.
   WARNING: The public SSH key file for gcloud does not exist.
   WARNING: You do not have an SSH key for gcloud.
   WARNING: SSH keygen will be executed to generate a key.
   This tool needs to create the directory [/home/student_01_9b51dda9fd9c/.ssh] before being able to generate SSH keys.
   
   Do you want to continue (Y/n)?  y
   
   Generating public/private rsa key pair.
   Enter passphrase (empty for no passphrase): 
   Enter same passphrase again: 
   Your identification has been saved in /home/student_01_9b51dda9fd9c/.ssh/google_compute_engine
   Your public key has been saved in /home/student_01_9b51dda9fd9c/.ssh/google_compute_engine.pub
   The key fingerprint is:
   SHA256:V91shEwl+OxdEjlcLM7rPQl1i33TEF09o77whGVVvOU student_01_9b51dda9fd9c@cs-22481000744-default
   The key's randomart image is:
   +---[RSA 3072]----+
   |             =oOX|
   |            ..XO*|
   |            .=+*O|
   |           . +B+E|
   |        S . =.+==|
   |         . o =o++|
   |            +.o.+|
   |             o.o.|
   |                .|
   +----[SHA256]-----+
   Warning: Permanently added 'compute.5071977469066933336' (ED25519) to the list of known hosts.
   Linux gcelab 5.10.0-28-cloud-amd64 #1 SMP Debian 5.10.209-2 (2024-01-31) x86_64
   
   The programs included with the Debian GNU/Linux system are free software;
   the exact distribution terms for each program are described in the
   individual files in /usr/share/doc/*/copyright.
   
   Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
   permitted by applicable law.
   Creating directory '/home/student-01-9b51dda9fd9c'.
   student-01-9b51dda9fd9c@gcelab:~$ 
   ```

   

Now find the disk device by listing the disk devices in `/dev/disk/by-id/.`:

```sh
student-01-9b51dda9fd9c@gcelab:~$ ls -l /dev/disk/by-id/
total 0
lrwxrwxrwx 1 root root  9 Apr 24 18:09 google-persistent-disk-0 -> ../../sda
lrwxrwxrwx 1 root root 10 Apr 24 18:09 google-persistent-disk-0-part1 -> ../../sda1
lrwxrwxrwx 1 root root 11 Apr 24 18:09 google-persistent-disk-0-part14 -> ../../sda14
lrwxrwxrwx 1 root root 11 Apr 24 18:09 google-persistent-disk-0-part15 -> ../../sda15
lrwxrwxrwx 1 root root  9 Apr 24 18:11 google-persistent-disk-1 -> ../../sdb
lrwxrwxrwx 1 root root  9 Apr 24 18:09 scsi-0Google_PersistentDisk_persistent-disk-0 -> ../../sda
lrwxrwxrwx 1 root root 10 Apr 24 18:09 scsi-0Google_PersistentDisk_persistent-disk-0-part1 -> ../../sda1
lrwxrwxrwx 1 root root 11 Apr 24 18:09 scsi-0Google_PersistentDisk_persistent-disk-0-part14 -> ../../sda14
lrwxrwxrwx 1 root root 11 Apr 24 18:09 scsi-0Google_PersistentDisk_persistent-disk-0-part15 -> ../../sda15
lrwxrwxrwx 1 root root  9 Apr 24 18:11 scsi-0Google_PersistentDisk_persistent-disk-1 -> ../../sdb
student-01-9b51dda9fd9c@gcelab:~$ 
```

You found the file, the default name is:

```
scsi-0Google_PersistentDisk_persistent-disk-1.
```

If you want a different device name, when you attach the disk, you would specify the `device-name` parameter. For example, to specify a device name, when you attach the disk you would use the command:

```
gcloud compute instances attach-disk gcelab --disk mydisk --device-name <YOUR_DEVICE_NAME> --zone $ZONE
```

### Formatting and mounting the persistent disk

Once you find the block device, you can partition the disk, format it, and then mount it using the following Linux utilities:

- `mkfs:` creates a filesystem
- `mount`: attaches to a filesystem

Make a mount point:

```sh
student-01-9b51dda9fd9c@gcelab:~$ sudo mkdir /mnt/mydisk
student-01-9b51dda9fd9c@gcelab:~$ 
```

Next, format the disk with a single `ext4` filesystem using the [mkfs](http://manpages.ubuntu.com/manpages/xenial/man8/mkfs.8.html) tool. This command deletes all data from the specified disk:

```sh
student-01-9b51dda9fd9c@gcelab:~$ sudo mkfs.ext4 -F -E lazy_itable_init=0,lazy_journal_init=0,discard /dev/disk/by-id/scsi-0Google_PersistentDisk_persistent-disk-1
mke2fs 1.46.2 (28-Feb-2021)
Discarding device blocks: done                            
Creating filesystem with 52428800 4k blocks and 13107200 inodes
Filesystem UUID: 26369752-6ee8-4000-89cb-3dcbef4bea6b
Superblock backups stored on blocks: 
        32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208, 
        4096000, 7962624, 11239424, 20480000, 23887872

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (262144 blocks): done
Writing superblocks and filesystem accounting information: done     

student-01-9b51dda9fd9c@gcelab:~$ 
```

Now use the [mount](http://manpages.ubuntu.com/manpages/xenial/man8/mount.8.html) tool to mount the disk to the instance with the `discard` option enabled:

```sh
student-01-9b51dda9fd9c@gcelab:~$ sudo mount -o discard,defaults /dev/disk/by-id/scsi-0Google_PersistentDisk_persistent-disk-1 /mnt/mydisk
student-01-9b51dda9fd9c@gcelab:~$ 
```

That's it!

### Automatically mount the disk on restart

By default the disk will not be remounted if your virtual machine  restarts. To make sure the disk is remounted on restart, you need to add an entry into `/etc/fstab`.

Open `/etc/fstab` in nano to edit:

```sh
student-01-9b51dda9fd9c@gcelab:~$ sudo cat /etc/fstab
# /etc/fstab: static file system information
UUID=b897c09d-bbfb-447a-9e3d-2279671b1fcf / ext4 rw,discard,errors=remount-ro,x-systemd.growfs 0 1
UUID=7EE9-6E8D /boot/efi vfat defaults 0 0
/dev/disk/by-id/scsi-0Google_PersistentDisk_persistent-disk-1 /mnt/mydisk ext4 defaults 1 1
student-01-9b51dda9fd9c@gcelab:~$ 
```

For migrating data from a persistent disk to another region, reorder the following steps in which they should be performed:

1. Attach disk
2. Create disk
3. Create snapshot
4. Create instance
5. Unmount file system(s) 

(5, 3, 2, 4, 1)

Unmount file system(s)  > Create snapshot > Create disk >  Create instance >  Attach disk

### Local SSDs

Compute Engine can also attach local SSDs. Local SSDs are physically attached to the server hosting the virtual machine instance to which they are mounted. This tight coupling offers superior performance, with very high input/output operations per second (IOPS) and very low latency compared to persistent disks.

Local SSD performance offers:

- Less than 1 ms of latency
- Up to 680,000 read IOPs and 360,000 write IOPs

These performance gains require certain trade-offs in availability, durability, and flexibility. Because of these trade-offs, local SSD storage is not automatically replicated and all data can be lost in the event of a host error or a user configuration error that makes the disk unreachable. Users must take extra precautions to backup their data. To maximize the local SSD performance, you'll need to use a special Linux image that supports NVMe.

```sh
student-01-9b51dda9fd9c@gcelab:~$ history 
    1  ls -l /dev/disk/by-id/
    2  sudo mkdir /mnt/mydisk
    3  sudo mkfs.ext4 -F -E lazy_itable_init=0,lazy_journal_init=0,discard /dev/disk/by-id/scsi-0Google_PersistentDisk_persistent-disk-1
    4  sudo mount -o discard,defaults /dev/disk/by-id/scsi-0Google_PersistentDisk_persistent-disk-1 /mnt/mydisk
    5  sudo nano /etc/fstab
    6  cat /etc/fstabab
    7  sudo cat /etc/fstab
    8  history 
student-01-9b51dda9fd9c@gcelab:~$ exit
logout
Connection to 34.69.205.108 closed.
student_01_9b51dda9fd9c@cloudshell:~ (qwiklabs-gcp-04-289d748a3fdf)$ history 
    1  gcloud config set compute/zone us-central1-c
    2  gcloud config set compute/region us-central1
    3  export REGION=us-central1
    4  export ZONE=us-central1-c
    5  gcloud compute instances create gcelab --zone $ZONE --machine-type e2-standard-2
    6  gcloud compute disks create mydisk --size=200GB --zone $ZONE
    7  gcloud compute instances attach-disk gcelab --disk mydisk --zone $ZONE
    8  gcloud compute ssh gcelab --zone $ZONE
    9  history 
student_01_9b51dda9fd9c@cloudshell:~ (qwiklabs-gcp-04-289d748a3fdf)$ 
```