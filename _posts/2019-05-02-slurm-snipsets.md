---
layout: post
title:  "Slurm snipsets"
author: "Sagiv Barhoom"
date:   2020-04-06
categories: Linux 
background: '/img/posts/be-linux.jpg.jpg'
---
# ```Slurm``` - Cluster management and job scheduling system
Slurm is an open-source, fault-tolerant, and highly scalable cluster management and job scheduling system for large and small Linux clusters. 
here are some snippets  which I use.
[slurm tutorial](https://slurm.schedmd.com/tutorials.html)

## sinfo
View information about Slurm nodes and partitions.
### sinfo example:
```bash
[sagiv@hpc_gate ~]$ sinfo  
PARTITION AVAIL  TIMELIMIT  NODES  STATE NODELIST
debugnodes*  up   infinite      2   idle cn[01-02]
worknodes    up   infinite      1    mix cn11
worknodes    up   infinite      7  alloc cn[12-14,17-20]
worknodes    up   infinite     12   idle cn[01-10,15-16]
gpunodes     up   infinite      4   idle gpu[1-4]
```
## scontrol 
Use this to see the details of all the nodes you can use
### scontrol examples:
```bash
scontrol show node "nodename"
scontrol show job 140036
```

## sbatch
Submit a batch script to Slurm.
For example if ```submit-pdb_prep-trimers.sh``` is a bash script, the following command will run it on partition ```worknodes``
### sbatch examples:
```bash
sbatch -p worknodes submit-pdb_prep-trimers.sh
```

## squeue
View information about jobs located in the Slurm scheduling queue.
### squeue examples:
```bash
squeue -u $USER       # squeue *my  jobs*
squeue -p  worknodes  # squeue * in partittion ''worknodes'*
```
### Tip for you:
* Use ``` watch  squeue -u $USER ``` to monitor your queue
* Create "profiles" for ```sbatch```, for example, this is an example for nice template fo pdb_prep which use 30 CPU's partially.
  For examples:
  ```
  # SBATCH --job-name=pdb_prep
  # SBATCH --ntasks=1
  # SBATCH --cpus-per-task=30
  # SBATCH --mail-user=your@mail.here
  ```

## srun
Run parallel jobs
The command ```srun``` will first create a resource allocation in which to run the parallel job,  if it is necessary.
### useful options:
```
-n, --ntasks=ntasks         number of tasks to run  
-p, --partition=partition   partition requested   
-c, --cpus-per-task=ncpus   number of cpus required per task 
-t, --time=minutes          time limit
--pty Execute task zero in pseudo terminal mode. 
       Implicitly sets: --unbuffered. --error and --output to /dev/null for all tasks except task zero
       for example use --pty bash for shell
```
### srun Examples:
```bash
srun -p gpunodes -n 16 -N 1 --pty bash
srun -p gpunodes -n 16 --pty bash
srun -p gpunodes -n 1 -c32 --pty bash
srun --partition=gpu --ntasks 1 --cpu-per-task=32 --pty bash
```

## scancel
Use ```scancel``` to signal jobs or job steps that are under the control.

### scancel Examples:
```bash
scancel  63041                #  cancel job 63041
scancel  -u $USER -p worknode #  cancel *my jobs* om partittion 'worknode'
```

## sacct 
Displays accounting data for all jobs and job steps in the Slurm job accounting log or Slurm database
For example - get statistics on completed jobs by jobID:
```
[sagiv@hpc_gate ~]$  sacct -j 143922 -o JobID,JobName,MaxRSS,Elapsed,Start,End
       JobID    JobName     MaxRSS    Elapsed               Start                 End
------------ ---------- ---------- ---------- ------------------- -------------------
143922             wrap              00:00:30 2019-05-01T11:29:07 2019-05-01T11:29:37
143922.batch      batch      1808K   00:00:30 2019-05-01T11:29:07 2019-05-01T11:29:37
143922.exte+     extern       348K   00:00:30 2019-05-01T11:29:07 2019-05-01T11:29:37
```
