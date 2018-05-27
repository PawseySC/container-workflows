---
title: "Running a Tensorflow app with GPUs and HPC"
teaching: 45
exercises: 0
questions:
- "How can we run a container on a HPC resource?"
objectives:
- "First objective."
keypoints:
- "First key point."
---

## Setting up on Zeus

First login using your supplied credentials.

Now we need to get the MINST benchmark, as we did in the Nimbus section.  This time we will download directly to the scratch directory.

~~~
cd $MYSCRATCH
git clone https://github.com/tensorflow/models.git
cd models/tutorials/image/mnist
~~~
{: .source}

~~~
#!/bin/bash --login
#SBATCH --job-name=minst_test
#SBATCH --nodes=1
#SBATCH --time=01:00:00
#SBATCH --account=pawsey0001
#SBATCH --gres=gpu:1
#SBATCH --partition=gpuq
#SBATCH --export=MYGROUP,MYSCRATCH

module swap gcc/4.8.5 gcc/5.5.0
module load cuda

export MY_WORKSCRIPT=${MYSCRATCH}/models/tutorials/image/mnist/convolutional.py
export MY_CONTAINER=docker://tensorflow/tensorflow:latest-gpu
export SINGULARITY_CACHEDIR=${MYGROUP}/singularity

if [ ! -d "$SINGULARITY_CACHEDIR" ]; then
   mkdir -p ${SINGULARITY_CACHEDIR}
fi

srun -N 1 -n 1 --export=ALL singularity exec --nv --bind /scratch ${MY_CONTAINER} python ${MY_WORKSCRIPT}
~~~
{: .source}

The first section contains a number of SLURM pragmas describing the resources we are requesting:
- number of nodes (1)
- maximum run time (1 hour)
- number of GPUs (1)
- SLURM job partition/queue (gpuq)
We also add some description to the job we hope to run as well including:
- its name (minst_test)
- Pawsey project that is going to be charged (pawsey0001)
- imported environment variables (MYSCRATCH, MYGROUP)

~~~

#SBATCH --job-name=minst_test
#SBATCH --nodes=1
#SBATCH --time=01:00:00
#SBATCH --account=pawsey0001
#SBATCH --gres=gpu:1
#SBATCH --partition=gpuq
#SBATCH --export=MYGROUP,MYSCRATCH
~~~
{: .source}

Let's setup the environment
~~~
module swap gcc/4.8.5 gcc/5.5.0
module load cuda

export MY_WORKSCRIPT=${MYSCRATCH}/models/tutorials/image/mnist/convolutional.py
export MY_CONTAINER=docker://tensorflow/tensorflow:latest-gpu
export SINGULARITY_CACHEDIR=${MYGROUP}/singularity

if [ ! -d "$SINGULARITY_CACHEDIR" ]; then
   mkdir -p ${SINGULARITY_CACHEDIR}
fi
~~~
{: .source}

Now we can launch using Singularity

~~~
srun -N 1 -n 1 --export=ALL singularity exec --nv --bind /scratch ${MY_CONTAINER} python ${MY_WORKSCRIPT}
~~~
{: .source}
