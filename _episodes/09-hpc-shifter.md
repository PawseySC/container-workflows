---
title: "Run containers on HPC with Shifter"
teaching: 10
exercises: 0
questions:
objectives:
- Learn how to manage and run containers with Shifter

keypoints:
---

### Pulling and managing images with Shifter ###

At present Docker has some features that make it unsuitable for running on HPC sytems, most notably the requirement to run as root. To enable the use of containers on Pawsey HPC systems, Shifter is made available.

To use it, we need first to load the corresponding module:

```
> module load shifter
```

The command to pull container images is very similar to Docker, for instance:

```
> shifter pull ubuntu
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
> shifter pull biocontainers/blast:v2.2.31_cv2
> shifter pull busybox
```

Similar again to Docker, we can list locally pulled images with:

```
> shifter images
biocontainers/blast              v2.2.31_cv2                  6c7abe0caf53   2018-12-19T22:33:46   729.12MB     index.docker.io
library/busybox                  latest                       7dc9d60af829   2018-12-19T22:31:48   704.00KB     index.docker.io
library/ubuntu                   latest                       d71fc6939e16   2018-12-19T22:30:41   29.94MB      index.docker.io
```

and remove undesired images with:

```
> shifter rmi busybox
removed image index.docker.io/library/busybox/latest
```


### Running images with Shifter ###

Let us change directory to our group directory with

```
> cd $MYGROUP
```

and then run `ls` using the Ubuntu image we just pulled:

```
> shifter run ubuntu ls
```

The output will display the content of the current host directory!

A few differences in behaviour can be noticed compared to Docker, such that using Shifter typically requires to specify less options and flags:

- by default, some relevant directories in the Pawsey HPC filesystems are mounted in the containers; these include `/group`, `/scratch`, `/pawsey` and `/tmp`.
  **NOTE**: `/home` is NOT mounted instead;
- if running from a mapped host directory, this becomes the working directory at container runtime;
- the host user is automatically set for the container;
- Shifter automatically removes containers after execution is terminated;

As an additional example, you might want to run:

```
> shifter run biocontainers/blast:v2.2.31_cv2 blastp -help
```

Finally, no flag is required to run a container interactively. To launch an interactive shell from within the container, just run it without any commands, for isntance:

```
> shifter run ubuntu

> exit
```

### Using Shifter with a job scheduler ###

Shifter is compatible with _SLURM_, the job scheduler installed on Pawsey HPC systems.

The following script permits to execute the bioinformatics example of a previous lesson using the Shifter and the job scheduler:

```
#!/bin/bash -l
  
#SBATCH --account=<your-pawsey-project>
#SBATCH --partition=workq
#SBATCH --ntasks=1
#SBATCH --time=00:05:00
#SBATCH --export=NONE
#SBATCH --job-name=blast

module load shifter

# download sample inputs
wget http://www.uniprot.org/uniprot/P04156.fasta
curl -O ftp://ftp.ncbi.nih.gov/refseq/D_rerio/mRNA_Prot/zebrafish.1.protein.faa.gz
gunzip zebrafish.1.protein.faa.gz

# prepare database
srun --export=all shifter run biocontainers/blast:v2.2.31_cv2 makeblastdb -in zebrafish.1.protein.faa -dbtype prot

# align with BLAST
srun --export=all shifter run biocontainers/blast:v2.2.31_cv2 blastp -query P04156.fasta -db zebrafish.1.protein.faa -out results.txt
```

Now you can create a test directory,

```
> mkdir blast_example
> cd blast_example
```

use your favourite text editor to copy paste the script above in a file called `blast.sh` (remember to specify your Pawsey account ID!),

and then submit this script using SLURM:

```
> sbatch blast.sh
```


### Conclusion ###
Shifter has a quite simple syntax that allows to pull, manage and run containers on HPC systems.
