Docker on HTCondor (still being composed)
=========================================

Installation for CentOS 6
--------------------------
* run the following as root: 
```bash
yum update
yum install -y docker-io
yum install -y condor
groupadd docker
usermod -aG docker condor
service docker start
service condor start
chkconfig docker on
chkconfig condor on
```
* Verify the Condor can see Docker:
```bash
[user@machine ~]$ condor_status -l | grep Docker
StarterAbilityList = "HasTDP,HasFileTransferPluginMethods,HasJobDeferral,HasJICLocalConfig,HasJICLocalStdin,HasPerFileEncryption,HasDocker,HasFileTransfer,HasReconnect,HasVM,HasMPI"
DockerVersion = "Docker version 1.7.1, build 786b29d/1.7.1"
HasDocker = true
```
* Verify that Condor is working, try a simple job:
```bash
[user@machine ~]$ cat > job.sub
universe = vanilla
executable = /bin/date
output = job.out
error = job.err
queue
ctrl+D

[user@machine ~]$ condor_submit job.sub
Submitting job(s).
1 job(s) submitted to cluster 1.
[user@machine ~]$ cat job.out
Sun Jan 22 11:05:40 UTC 2017
```
* Submit a simple Docker job to Condor:
```bash
[user@machine ~]$ cat > docker_job.sub
universe = docker
docker_image = centos
executable = cat
arguments = /etc/system-release
output = docker_job.out
error = docker_job.err
queue
ctrl+D

[user@machine ~]$ condor_submit docker_job.sub
Submitting job(s).
1 job(s) submitted to cluster 2.
[user@machine ~]$ condor_q


-- Schedd: condor-docker.novalocal : <192.168.1.121:20790?...
 ID      OWNER            SUBMITTED     RUN_TIME ST PRI SIZE CMD
   2.0   cloud-user      1/22 11:17   0+00:00:00 I  0   0.0  cat /etc/system-re

1 jobs; 0 completed, 0 removed, 1 idle, 0 running, 0 held, 0 suspended
[user@machine ~]$ condor_q


-- Schedd: condor-docker.novalocal : <192.168.1.121:20790?...
 ID      OWNER            SUBMITTED     RUN_TIME ST PRI SIZE CMD

0 jobs; 0 completed, 0 removed, 0 idle, 0 running, 0 held, 0 suspended
[user@machine ~]$ cat docker_job.out
CentOS Linux release 7.3.1611 (Core)
```
# Condor Docker job with mounted volumes
* Create directories for input/output volumes
```bash
[user@machine ~]$ mkdir docker_in
[user@machine ~]$ echo hello! > docker_in/infile
[user@machine ~]$ mkdir docker_out
```
* Configure Condor to mount volumes on Docker images, as root:
```bash
[user@machine ~]$ sudo cat > /etc/condor/config.d/docker
#Define volumes to mount:
DOCKER_VOLUMES = DOCKER_IN, DOCKER_OUT

#Define a mount point for each volume:
DOCKER_VOLUME_DIR_DOCKER_IN = /home/user/docker_in:/input:ro
DOCKER_VOLUME_DIR_DOCKER_OUT = /home/user/docker_out:/output:rw

#Configure those volumes to be mounted on each Docker container:
DOCKER_MOUNT_VOLUMES = DOCKER_IN, DOCKER_OUT
ctrl+D
```
* Create the submission script and submit the job
```bash
[user@machine ~]$ cat > docker_volumes_job.sub
universe = docker
docker_image = centos
executable = cp
arguments = /input/infile /output/outfile
output = docker_volumes_job.out
error = docker_volumes_job.err
queue
ctrl+D

[user@machine ~]$ condor_submit docker_volumes_job.sub
Submitting job(s).
1 job(s) submitted to cluster 7.
[user@machine ~]$ cat docker_out/outfile
hello!
```
Installation for CentOS 7
--------------------------

* [Install Condor](https://research.cs.wisc.edu/htcondor/instructions/el/7/stable/)
* [Configure Condor for multiple nodes](https://spinningmatt.wordpress.com/2011/06/12/getting-started-creating-a-multiple-node-condor-pool/)
* All condor logs are in ``/var/log/condor/``
* All condor config files ``/etc/condor/``

Useful links
-------------
 * [Condor security](http://research.cs.wisc.edu/htcondor/manual/v8.2/3_6Security.html)
