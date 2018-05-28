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
~~~
ssh couXXX@zeus.pawsey.org.au
~~~
{: .source}

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
#SBATCH --time=00:15:00
#SBATCH --account=courses01
#SBATCH --gres=gpu:1
#SBATCH --partition=gpuq
#SBATCH --reservation=courseq-gpu
#SBATCH --export=MYGROUP,MYSCRATCH

module load broadwell
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
- Number of nodes (1)
- Maximum run time (15 minutes)
- Number of GPUs (1)
- SLURM job partition/queue (gpuq)
We also add some description to the job we hope to run as well including:
- Job name (minst_test)
- Pawsey project that is going to be charged (courses01)
- Imported environment variables (MYSCRATCH, MYGROUP)
- Reservation of nodes to use (courseq-gpu)

~~~

#SBATCH --job-name=minst_test
#SBATCH --nodes=1
#SBATCH --time=00:15:00
#SBATCH --account=courses01
#SBATCH --gres=gpu:1
#SBATCH --partition=gpuq
#SBATCH --reservation=courseq-gpu
#SBATCH --export=MYGROUP,MYSCRATCH

~~~
{: .source}

Let's setup the environment
~~~
module load broadwell
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

The job launch command is not too different from a normal SLURM command line.  We still launch with the number of nodes (`-N 1`) and tasks (`-n 1`)
we want to use.  Instead of simply running the Python script, we now launch a Singularity container, and from within that container we run Python.

The other important Singularity specific directives include:

`--nv`
  * Instructs Singularity to use host NVIDIA GPU
  * Requires a recent NVIDIA driver

`--bind`
  * Specify which host directories we want to mount inside the container
  * This will mount `/scratch` on Zeus to `/scratch` inside the TensorFlow container
  * NOTE: Bind points must exist in the container prior to launch
