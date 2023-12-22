# SLURM
* Simple Linux Utility for Resource Management
* it is master and worker node architecture
* it can't be described as client server because

```
reason being, server is always passively open and listens for the request and serves them.
here client is the initiator

whereas in concept of master-minion master asks for the data from the worker node, it is not waiting for worker node request.
here masteris is the initiator
```

## Compnents of SLURM
### Batch systems
we need run our jobs on group of clusters in batches. to get the maximum utilisation of the clusters
* Open PBS
* PBS pro
### Scheduler 
### Job Accounting
### Resource Manager 
runs in the background to manage all the resources 

## Architecture of SLURM
![slurm architecture](./arch.gif)

## commands for SLURM
* #### sacct                 
* #### salloc
* #### sattach
* #### sbatch 
* #### sbcast 
* #### scancel 
* #### scontrol 
* #### sinfo 
* #### sprio 
* #### squeue 
* #### srun 
* #### sshare 
* #### ssat 
* #### strigger 
* #### sview 

## Authentication Service
[MUNGE](https://github.com/dun/munge/wiki) it is a authentication service used within the cluster. used to authenticate internal services.

uses munge key to authenticate and encrypt services between slurmctl and slurmd.

default port : 6817 (slurmctld)
default port : 6818 (slurmd)
default port : 6819 (slurmdbd)

FLOW of MUNGE service
1. key generation
1. message encryption
1. message authentication
1. access control


