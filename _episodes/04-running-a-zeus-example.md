---
title: "Running a Tensorflow app with GPUs and HPC"
teaching: 45
exercises: 0
questions:
- "How can we run a container on a HPC resource?"
objectives:
- "Run a task on a HPC facility using Singularity."
keypoints:
- "Containers can be used to support cross platform workflows"
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

mkdir $MYSCRATCH/tmp
export TMPDIR=$MYSCRATCH/tmp
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

mkdir $MYSCRATCH/tmp
export TMPDIR=$MYSCRATCH/tmp
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

We now need to submit our jobscript to the scheduler:

~~~
sbatch minst.slm
~~~
{: .source}

~~~
Submitted batch job 2498240
~~~~

It may take a little bit the first time, as the TensorFlow image is pulled down.  You should see output similar to this:
~~~
Docker image path: index.docker.io/tensorflow/tensorflow:latest-gpu
Cache folder set to /group/courses01/cou000/singularity/docker
Creating container runtime...
Extracting data/train-images-idx3-ubyte.gz
Extracting data/train-labels-idx1-ubyte.gz
Extracting data/t10k-images-idx3-ubyte.gz
Extracting data/t10k-labels-idx1-ubyte.gz
2018-02-18 20:33:59.677580: W tensorflow/core/platform/cpu_feature_guard.cc:45] The TensorFlow library wasn't compiled to use SSE4.1 instructions, but these are available on your machine and could speed up CPU computations.
2018-02-18 20:33:59.677620: W tensorflow/core/platform/cpu_feature_guard.cc:45] The TensorFlow library wasn't compiled to use SSE4.2 instructions, but these are available on your machine and could speed up CPU computations.
2018-02-18 20:34:00.148531: I tensorflow/stream_executor/cuda/cuda_gpu_executor.cc:893] successful NUMA node read from SysFS had negative value (-1), but there must be at least one NUMA node, so returning NUMA node zero
2017-02-18 20:34:00.148926: I tensorflow/core/common_runtime/gpu/gpu_device.cc:955] Found device 0 with properties:
major: 3 minor: 0 memoryClockRate (GHz) 0.8885
pciBusID 0000:03:00.0
Total memory: 2.95GiB
Free memory: 2.92GiB
2018-02-18 20:34:00.148954: I tensorflow/core/common_runtime/gpu/gpu_device.cc:976] DMA: 0
2018-02-18 20:34:00.148965: I tensorflow/core/common_runtime/gpu/gpu_device.cc:986] 0:   Y
Initialized!
Step 0 (epoch 0.00), 21.7 ms
Minibatch loss: 8.334, learning rate: 0.010000
Minibatch error: 85.9%
Validation error: 84.6%
Step 100 (epoch 0.12), 20.9 ms
Minibatch loss: 3.235, learning rate: 0.010000
Minibatch error: 4.7%
Validation error: 7.8%
Step 200 (epoch 0.23), 20.5 ms
Minibatch loss: 3.363, learning rate: 0.010000
Minibatch error: 9.4%
Validation error: 4.2%
[...snip...]
Step 8500 (epoch 9.89), 20.5 ms
Minibatch loss: 1.602, learning rate: 0.006302
Minibatch error: 0.0%
Validation error: 0.9%
Test error: 0.8%
