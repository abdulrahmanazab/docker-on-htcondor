Docker on HTCondor (still being composed)
=========================================

Installation for CentOS 6
--------------------------
run the following as root: 
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
Now verify the Condor can see Docker:
```bash
[user@machine ~]$ condor_status -l | grep Docker
StarterAbilityList = "HasTDP,HasFileTransferPluginMethods,HasJobDeferral,HasJICLocalConfig,HasJICLocalStdin,HasPerFileEncryption,HasDocker,HasFileTransfer,HasReconnect,HasVM,HasMPI"
DockerVersion = "Docker version 1.7.1, build 786b29d/1.7.1"
HasDocker = true
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
