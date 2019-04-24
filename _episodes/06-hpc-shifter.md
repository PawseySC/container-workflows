---
title: "Run containers on HPC with Shifter"
teaching: 20
exercises: 0
questions:
objectives:
- Learn how to manage and run containers on a HPC cluster with Shifter

keypoints:
---

### Why not Docker on HPC? ###

There are a few issues preventing Docker from being used as a container engine on HPC systems:

* Security: Docker requires root privileges
* Batch systems: doesn't integrate well with schedulers
* Underlying kernel: usually requires an up-to-date kernel

Fortunately, a number of alternatives are available to run containers at HPC facilities, including:

* [Shifter](https://docs.nersc.gov/programming/shifter/overview/): developed by NERSC and Cray, Docker-like interface, MPI support 
* [CSCS Shifter](https://user.cscs.ch/tools/containers/): forked by CSCS, adds features including GPU support 
* [Singularity](https://www.sylabs.io/singularity/): originally developed by LBL, has its own image format and can run Docker containers as well

At the moment, Pawsey is using CSCS Shifter on its HPC systems, and therefore this will be the tool of choice in this tutorial.


### Pulling and managing images with Shifter ###

To use Shifter on Pawsey HPC systems, we need first to load the corresponding module:

```
> module load shifter
```

In principle, the command to pull container images is very similar to Docker, `shifter pull`.  
However, to avoid disk quota issues on Pawsey HPC systems, the following syntax is recommended, that makes use of the `sg` linux command, for instance:

```
> sg $PAWSEY_PROJECT -c 'shifter pull ubuntu'
# image     : index.docker.io/library/ubuntu/latest
# cacheDir  : /group/shifterrepos/mdelapierre/.shifter/cache
# tmpDir    : /tmp
# imageDir  : /group/shifterrepos/mdelapierre/.shifter/images
> save image layers ...
> pulling        : sha256:f85999a86bef2603a9e9a4fa488a7c1f82e471cbb76c3b5068e54e1a9320964a
> pulling        : sha256:da1315cffa03c17988ae5c66f56d5f50517652a622afc1611a8bdd6c00b1fde3

> extracting     : /group/shifterrepos/mdelapierre/.shifter/cache/sha256:f85999a86bef2603a9e9a4fa488a7c1f82e471cbb76c3b5068e54e1a9320964a.tar
> make squashfs ...
> create metadata ...
# created: /group/shifterrepos/mdelapierre/.shifter/images/index.docker.io/library/ubuntu/latest.squashfs
# created: /group/shifterrepos/mdelapierre/.shifter/images/index.docker.io/library/ubuntu/latest.meta
```

```
> sg $PAWSEY_PROJECT -c 'shifter pull centos'
> sg $PAWSEY_PROJECT -c 'shifter pull busybox'
```

Similar again to Docker, we can list locally pulled images with `shifter images`:

```
> shifter images
library/centos                   latest                       ea4b646d9000   2018-11-27T07:05:23   69.62MB      index.docker.io
library/busybox                  latest                       7dc9d60af829   2018-12-19T22:31:48   704.00KB     index.docker.io
library/ubuntu                   latest                       d71fc6939e16   2018-12-19T22:30:41   29.94MB      index.docker.io
```

and remove undesired images with `shifter rmi`:

```
> shifter rmi busybox
removed image index.docker.io/library/busybox/latest
```


### Running images with Shifter ###

Let us change directory to our group directory with:

```
> cd $MYGROUP
```

and then run `ls` using the Ubuntu image we just pulled, via `shifter run`:

```
> shifter run ubuntu ls
```

The output will display the content of the current host directory!

A few differences in behaviour can be noticed compared to Docker, such that using Shifter typically requires to specify less options and flags:

- by default, some relevant directories in the Pawsey HPC filesystems are mounted in the containers; these include `/group`, `/scratch`, `/pawsey` and `/tmp`.  
  **Note**: `/home` is NOT mounted instead;
- if running from a mapped host directory, this becomes the working directory at container runtime;
- the host user is automatically set for the container;
- Shifter automatically removes containers after execution is terminated.

As additional examples, you might want to run:

```
> shifter run ubuntu ls /
> shifter run ubuntu whoami
```

Note how no flag is required to run a container interactively. To launch an interactive shell from within the container, just run it without any commands, for instance:

```
> shifter run ubuntu

> exit
```

Shifter has support to run containers exploiting MPI parallelism and GPU acceleration (the latter only through CSCS Shifter).

* `shifter run --mpi` allows containers to take advantage on inter-node communication on the host fabric. The container image needs to have been built with MPI libraries thatare ABI compatible with the host MPI libraries;

* to run GPU enabled containers, no extra flags are required.


### Using Shifter with a job scheduler ###

Shifter is compatible with **SLURM**, the job scheduler installed on Pawsey HPC systems.

As an example, the following script uses a Ubuntu container to output the machine hostname:

```
#!/bin/bash -l
  
#SBATCH --account=<your-pawsey-project>
#SBATCH --partition=workq
#SBATCH --ntasks=1
#SBATCH --time=00:05:00
#SBATCH --export=NONE
#SBATCH --job-name=blast

module load shifter

srun --export=all shifter run ubuntu hostname
```

Now use your favourite text editor to copy paste the script above in a file called `hostname.sh` somewhere under `$MYSCRATCH` or `$MYGROUP` (remember to specify your Pawsey Project ID in the script!),

and then submit this script using SLURM:

```
> sbatch hostname.sh
```


### Building container images for HPC ###

Shifter does not allow to build container images. The best way to create an image to be pulled and run on HPC is to use Docker on a distinct machine (see previous episode).


### Conclusion ###
Shifter has a quite simple syntax that allows to pull, manage and run containers on HPC systems.
