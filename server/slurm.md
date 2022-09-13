# Setup HPC cluster with slurm

Slurm is a job scheduler, very useful to manage run simulations for users on a HPC cluster. 

Here is described the process to install Slurm on a small cluster made of a single node and running on Ubuntu 20.04.

## Install Slurm 

Slurm can be installed directly from the package manager `apt`:

```
# apt-get update
# apt-get upgrade
# apt-get install slurm-wlm -y
```

This package includes:
- `slurmctld` which is the central management daemon of slurm 
- `slurmd` which is the compute node daemon of Slurm
- other related tools such as `munge` (required for secure authentication on nodes by Slurm jobs) and a database to store job history, queue and so on.

To check that the tools are properly installed and check the version you can do:

```
$ slurmd --version
$ slurmctld -V
slurm-wlm 19.05.5
```

## Configure Slurm

### Configuration tool

Slurm can be configured with the help of a configurator tool, either using the online version from the Slurm website or by running locally the configurator tool.

Note that the configurator tool from the Slurm website works for the latest release of Slurm and that options compatibility with an older version is not ensured (in my case it was definitely a problem).

Running locally the configurator tool can be done using an simple web server for example `python3 -m http.server` in the folder */usr/share/doc/slurmctld/* or by simply opening the file *slurm-wlm-configurator.html* or the light version *slurm-wlm-configurator.easy.html* within the same folder. 

After using the configurator tool a configuration file is generated which must be saved in */etc/slurm-llnl/slurm.conf*.

### Template

Another possibility to configure Slurm is to get a template of the configuration file and modify it to your need.

```
cd /etc/slurm-llnl/
cp /usr/share/doc/slurmctld/examples/slurm.conf.simple.gz .
gzip -d slurm.conf.simple.gz
mv slurm.conf.simple slurm.conf
```

Edit the configuration file */etc/slurm-llnl/slurm.conf*.

Set the control machine info where `hostName` must be the name given by `hostname -s`:

```
SlurmctldHost=hostName
```

Check the scheduler algorithm according to consumable ressources of interest. In our case, CPU cores, it reads:

```
SelectType=select/cons_res
SelectTypeParameters=CR_Core
```

Set the cluster name:

```
ClusterName=cluster
```

Add the compute nodes:

```
NodeName=hostName CPUs=48 State=UNKNOWN
```

**Remark:**
Note that for a cluster with several nodes it is important to add the node IP address and also define it for the slurmctld node host:

```
SlurmctldHost=node01(192.168.1.50)
NodeName=cid01 NodeAddr=192.168.1.51 CPUs=32 State=UNKNOWN
```

Slurm runs jobs on partitions, which are groups nodes. 
In our case, with a single node we have:

```
PartitionName=cluster Nodes=hostName Default=YES MaxTime=7-00:00 State=UP
```

Slurm controls access to system resources (CPU, memory, I/O, etc.) using the *cgroup* linux kernel feature. 
In order for the system to communicate to Slurm the available ressources, a */etc/slurm-llnl/cgroup.conf* file is added with the following content:

```
CgroupMountpoint="/sys/fs/cgroup"
CgroupAutomount=yes
CgroupReleaseAgentDir="/etc/slurm-llnl/cgroup"
AllowedDevicesFile="/etc/slurm-llnl/cgroup_allowed_devices_file.conf"
ConstrainCores=no
TaskAffinity=no
ConstrainRAMSpace=yes
ConstrainSwapSpace=no
ConstrainDevices=no
AllowedRamSpace=100
AllowedSwapSpace=0
MaxRAMPercent=100
MaxSwapPercent=100
MinRAMSpace=30
```

As defined by the variable `AllowedDevicesFile` a whitelist file */etc/slurm-llnl/cgroup_allowed_devices_file.conf* is created to allow access to ressources (note that this setup is highly permissive and might be not suited depending on the use case):

```
/dev/null
/dev/urandom
/dev/zero
/dev/sda*
/dev/cpu/*/*
/dev/pts/*
```

Since this setup is based on a single node architecture, there is no need to configure other compute nodes. If it was necessary the Slurm node daemon must be installed along with the `munge` authentication tool and its key on each node. 

Start the controller daemon and the node daemon:

```
# service slurmctld start
# service slurmd start
```

References:
- [Raspberry cluster from Medium article](https://glmdev.medium.com/building-a-raspberry-pi-cluster-784f0df9afbd)
- [Blog post on Signac website](https://signac.io/development/2020/06/26/local-SLURM-environment.html)

## Test Slurm

To make sure everything is working correctly, first use the basic Slurm commands `sinfo`, `squeue -a` and after that prepare a simple job using a `job.sh` file:

```
#!/bin/sh

# Slurm batch directives

#SBATCH -J jobName
#SBATCH -N 1
#SBATCH --ntasks-per-node=5
#SBATCH -t 07-00:00:00
#SBATCH -o ./%x-%j.out
#SBATCH -e ./%x-%j.err
# #SBATCH --mail-type=BEGIN,END
# #SBATCH --mail-user=user.name@company.com

# Run test
# ./code

```

Run it and check status, allocated ressources and so on:

```
$ sbatch ./job.sh
$ squeue -a
$ sinfo
$ sinfo -o "%n %a %C"
$ sinfo -o "%.8P %.5a %.6D %.6t %.8N %.9B %.11A %.14C %.6D %.8O"
$ squeue -o "%.18i %.9P %.8j %.8u %.2t %.10M %.8C"
```

## Add mail support

If you want to add email support to receive an email when a job start or end you need to install a Postfix mail server (see mail.md) and modify the */etc/slurm-llnl/slurm.conf* file to tell Slurm to use the Postfix server to send the email:

```
MailProg=/usr/bin/mail
```

References: 
- [StackOverflow post 1](https://stackoverflow.com/questions/60916882/how-does-one-implement-the-e-maling-option-for-slurm)
- [StackOverflow post 2](https://stackoverflow.com/questions/53003230/how-to-configure-the-content-of-slurm-notification-emails)



