---
title: "Running a sample app in Nimbus"
teaching: 0
exercises: 0
questions:
- "How do you use Containers on Nimbus?"
objectives:
- "Setup and install Docker"
- "Use an Docker container to run a Machine Learning script"
keypoints:
- "First key point."
---

## Let's get started with Nimbus

words

## First we need to setup and install Docker

These instructions are taken from https://docs.docker.com/install/linux/docker-ce/ubuntu/.

First we need to setup a local repository for Docker, we can then install Docker, and use a Docker container.

## Set up the Repository

~~~
$ sudo apt-get update
~~~
{: .source}

~~~
Get:1 http://security.ubuntu.com/ubuntu bionic-security InRelease [83.2 kB]
Get:2 http://nova.clouds.archive.ubuntu.com/ubuntu bionic InRelease [242 kB]                     
Get:3 http://security.ubuntu.com/ubuntu bionic-security/main Sources [19.6 kB]
...etc...
Get:31 http://nova.clouds.archive.ubuntu.com/ubuntu bionic-updates/multiverse amd64 Packages [1728 B]
Get:32 http://nova.clouds.archive.ubuntu.com/ubuntu bionic-updates/multiverse Translation-en [1236 B]
Fetched 26.3 MB in 15s (1713 kB/s)                                                               
Reading package lists... Done
~~~
{: .output}

~~~
$ sudo apt-get install apt-transport-https  ca-certificates  curl  software-properties-common
~~~
{: .source}

~~~
Reading package lists... Done
Building dependency tree       
Reading state information... Done
ca-certificates is already the newest version (20180409).
The following package was automatically installed and is no longer required:
  grub-pc-bin
Use 'sudo apt autoremove' to remove it.
The following additional packages will be installed:
  libcurl4 python3-software-properties
The following NEW packages will be installed:
  apt-transport-https
The following packages will be upgraded:
  curl libcurl4 python3-software-properties software-properties-common
4 upgraded, 1 newly installed, 0 to remove and 23 not upgraded.
Need to get 407 kB of archives.
After this operation, 152 kB of additional disk space will be used.
Do you want to continue? [Y/n] Y
Get:1 http://nova.clouds.archive.ubuntu.com/ubuntu bionic/universe amd64 apt-transport-https all 1.6.1 [1692 B]
Get:2 http://nova.clouds.archive.ubuntu.com/ubuntu bionic-updates/main amd64 curl amd64 7.58.0-2ubuntu3.1 [159 kB]
Get:3 http://nova.clouds.archive.ubuntu.com/ubuntu bionic-updates/main amd64 libcurl4 amd64 7.58.0-2ubuntu3.1 [214 kB]
Get:4 http://nova.clouds.archive.ubuntu.com/ubuntu bionic-updates/main amd64 software-properties-common all 0.96.24.32.2 [9916 B]
Get:5 http://nova.clouds.archive.ubuntu.com/ubuntu bionic-updates/main amd64 python3-software-properties all 0.96.24.32.2 [22.8 kB]
Fetched 407 kB in 3s (134 kB/s)                   
Selecting previously unselected package apt-transport-https.
(Reading database ... 59930 files and directories currently installed.)
Preparing to unpack .../apt-transport-https_1.6.1_all.deb ...
Unpacking apt-transport-https (1.6.1) ...
Preparing to unpack .../curl_7.58.0-2ubuntu3.1_amd64.deb ...
Unpacking curl (7.58.0-2ubuntu3.1) over (7.58.0-2ubuntu3) ...
Preparing to unpack .../libcurl4_7.58.0-2ubuntu3.1_amd64.deb ...
Unpacking libcurl4:amd64 (7.58.0-2ubuntu3.1) over (7.58.0-2ubuntu3) ...
Preparing to unpack .../software-properties-common_0.96.24.32.2_all.deb ...
Unpacking software-properties-common (0.96.24.32.2) over (0.96.24.32.1) ...
Preparing to unpack .../python3-software-properties_0.96.24.32.2_all.deb ...
Unpacking python3-software-properties (0.96.24.32.2) over (0.96.24.32.1) ...
Setting up apt-transport-https (1.6.1) ...
Setting up libcurl4:amd64 (7.58.0-2ubuntu3.1) ...
Processing triggers for libc-bin (2.27-3ubuntu1) ...
Processing triggers for man-db (2.8.3-2) ...
Setting up python3-software-properties (0.96.24.32.2) ...
Processing triggers for dbus (1.12.2-1ubuntu1) ...
Setting up software-properties-common (0.96.24.32.2) ...
Setting up curl (7.58.0-2ubuntu3.1) ...
~~~
{: .output}

~~~
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
~~~
{: .source}

~~~
OK
~~~
{: .output}

~~~
$ sudo apt-key fingerprint 0EBFCD88
~~~
{: .source}

~~~
pub   rsa4096 2017-02-22 [SCEA]
      9DC8 5822 9FC7 DD38 854A  E2D8 8D81 803C 0EBF CD88
uid           [ unknown] Docker Release (CE deb) <docker@docker.com>
sub   rsa4096 2017-02-22 [S]
~~~
{: .output}

Let's check something here before we continue.  We need to make sure we're using the right version Ubuntu:

~~~
$ lsb_release -cs
~~~
{: .source}

~~~
xenial
~~~
{: .output}

~~~
$ sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
~~~
{: .source}

~~~
Get:1 http://security.ubuntu.com/ubuntu bionic-security InRelease [83.2 kB]                                                
Get:2 https://download.docker.com/linux/ubuntu bionic InRelease [64.4 kB]                                                  
Hit:3 http://nova.clouds.archive.ubuntu.com/ubuntu bionic InRelease                                             
Get:4 http://nova.clouds.archive.ubuntu.com/ubuntu bionic-updates InRelease [83.2 kB]
Hit:5 http://nova.clouds.archive.ubuntu.com/ubuntu bionic-backports InRelease
Get:6 http://nova.clouds.archive.ubuntu.com/ubuntu bionic-updates/main amd64 Packages [100 kB]
Get:7 http://nova.clouds.archive.ubuntu.com/ubuntu bionic-updates/universe amd64 Packages [62.1 kB]
Fetched 393 kB in 4s (91.2 kB/s)  
Reading package lists... Done
~~~
{: .output}

## Now we can install Docker (Community Edition)

~~~
$ sudo apt-get update
~~~
{: .source}

~~~
Get:1 http://security.ubuntu.com/ubuntu bionic-security InRelease [83.2 kB]                  
Hit:2 https://download.docker.com/linux/ubuntu bionic InRelease                                          
Hit:3 http://nova.clouds.archive.ubuntu.com/ubuntu bionic InRelease                                                           
Hit:4 http://nova.clouds.archive.ubuntu.com/ubuntu bionic-updates InRelease
Hit:5 http://nova.clouds.archive.ubuntu.com/ubuntu bionic-backports InRelease
Fetched 83.2 kB in 2s (39.8 kB/s)
Reading package lists... Done
~~~
{. output}

~~~
$ sudo apt-get install docker-ce
~~~
{: .source}

~~~
Reading package lists... Done
Building dependency tree       
Reading state information... Done
The following additional packages will be installed:
  aufs-tools cgroupfs-mount libltdl7 pigz
Suggested packages:
  mountall
The following NEW packages will be installed:
  aufs-tools cgroupfs-mount docker-ce libltdl7 pigz
0 upgraded, 5 newly installed, 0 to remove and 24 not upgraded.
Need to get 34.2 MB of archives.
After this operation, 182 MB of additional disk space will be used.
Do you want to continue? [Y/n] Y
Get:1 http://nova.clouds.archive.ubuntu.com/ubuntu xenial/universe amd64 pigz amd64 2.3.1-2 [61.1 kB]
Get:2 https://download.docker.com/linux/ubuntu xenial/stable amd64 docker-ce amd64 18.03.1~ce-0~ubuntu [34.0 MB]
Get:3 http://nova.clouds.archive.ubuntu.com/ubuntu xenial/universe amd64 aufs-tools amd64 1:3.2+20130722-1.1ubuntu1 [92.9 kB]
Get:4 http://nova.clouds.archive.ubuntu.com/ubuntu xenial/universe amd64 cgroupfs-mount all 1.2 [4,970 B]
Get:5 http://nova.clouds.archive.ubuntu.com/ubuntu xenial/main amd64 libltdl7 amd64 2.4.6-0.1 [38.3 kB]
Fetched 34.2 MB in 3s (10.7 MB/s)  
Selecting previously unselected package pigz.
(Reading database ... 54012 files and directories currently installed.)
Preparing to unpack .../pigz_2.3.1-2_amd64.deb ...
Unpacking pigz (2.3.1-2) ...
Selecting previously unselected package aufs-tools.
Preparing to unpack .../aufs-tools_1%3a3.2+20130722-1.1ubuntu1_amd64.deb ...
Unpacking aufs-tools (1:3.2+20130722-1.1ubuntu1) ...
Selecting previously unselected package cgroupfs-mount.
Preparing to unpack .../cgroupfs-mount_1.2_all.deb ...
Unpacking cgroupfs-mount (1.2) ...
Selecting previously unselected package libltdl7:amd64.
Preparing to unpack .../libltdl7_2.4.6-0.1_amd64.deb ...
Unpacking libltdl7:amd64 (2.4.6-0.1) ...
Selecting previously unselected package docker-ce.
Preparing to unpack .../docker-ce_18.03.1~ce-0~ubuntu_amd64.deb ...
Unpacking docker-ce (18.03.1~ce-0~ubuntu) ...
Processing triggers for man-db (2.7.5-1) ...
Processing triggers for libc-bin (2.23-0ubuntu10) ...
Processing triggers for ureadahead (0.100.0-19) ...
Processing triggers for systemd (229-4ubuntu21.2) ...
Setting up pigz (2.3.1-2) ...
Setting up aufs-tools (1:3.2+20130722-1.1ubuntu1) ...
Setting up cgroupfs-mount (1.2) ...
Setting up libltdl7:amd64 (2.4.6-0.1) ...
Setting up docker-ce (18.03.1~ce-0~ubuntu) ...
Processing triggers for libc-bin (2.23-0ubuntu10) ...
Processing triggers for systemd (229-4ubuntu21.2) ...
Processing triggers for ureadahead (0.100.0-19) ...
~~~
{: .output}

Let's check that everything installer properly

~~~
$ sudo docker run hello-world
~~~
{: .source}

~~~
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
9bb5a5d4561a: Pull complete
Digest: sha256:f5233545e43561214ca4891fd1157e1c3c563316ed8e237750d59bde73361e77
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/engine/userguide/
~~~
{: .output}

If you get the 'Hello from Docker' message then all is well.

~~~
git clone https://github.com/tensorflow/models.git
~~~
{: .source}

~~~
Cloning into 'models'...
remote: Counting objects: 17327, done.
remote: Compressing objects: 100% (2/2), done.
remote: Total 17327 (delta 2), reused 2 (delta 2), pack-reused 17323
Receiving objects: 100% (17327/17327), 469.82 MiB | 5.41 MiB/s, done.
Resolving deltas: 100% (10311/10311), done.
Checking connectivity... done.
Checking out files: 100% (2248/2248), done.
~~~
{: .output}

## Now we can run something

First we need to download the sample ML algorithm we will run.
~~~
$ ls models/tutorials/image/mnist
~~~
{: .source}

~~~
BUILD  convolutional.py  __init__.py
~~~
{: .output}

We ned to edit convolutional.py to set NUM_EPOCS to 1 (since we don’t have a GPU here, this will take forever at the current value).

~~~
# CVDF mirror of http://yann.lecun.com/exdb/mnist/
SOURCE_URL = 'https://storage.googleapis.com/cvdf-datasets/mnist/'
WORK_DIRECTORY = 'data'
IMAGE_SIZE = 28
NUM_CHANNELS = 1
PIXEL_DEPTH = 255
NUM_LABELS = 10
VALIDATION_SIZE = 5000  # Size of the validation set.
SEED = 66478  # Set to None for random seed.
BATCH_SIZE = 64
NUM_EPOCHS = 1
EVAL_BATCH_SIZE = 64
EVAL_FREQUENCY = 100  # Number of steps between evaluations.
~~~
{: .output}

~~~
$ sudo docker run --rm -it -v /home/ubuntu/models/tutorials/image/mnist/:/notebooks tensorflow/tensorflow bash
~~~
{: .source}

~~~
root@503d2d42cc5c:/notebooks#
~~~
{: .output}

We look at the local notebooks directory now we will the contents we saw before

~~~
root@503d2d42cc5c:/notebooks# ls
~~~
{: .source}

~~~
BUILD  __init__.py  convolutional.py
~~~
{: .output}

Now we can simply run our example.  This will be running in our Tensorflow container. It should take about 5 mins, so take a few moments to breathe.
~~~
root@503d2d42cc5c:/notebooks# python convolutional.py
~~~
{: .source}

~~~
/usr/local/lib/python2.7/dist-packages/h5py/__init__.py:36: FutureWarning: Conversion of the second argument of issubdtype from `float` to `np.floating` is deprecated. In future, it will be treated as `np.float64 == np.dtype(float).type`.
  from ._conv import register_converters as _register_converters
Successfully downloaded train-images-idx3-ubyte.gz 9912422 bytes.
Successfully downloaded train-labels-idx1-ubyte.gz 28881 bytes.
Successfully downloaded t10k-images-idx3-ubyte.gz 1648877 bytes.
Successfully downloaded t10k-labels-idx1-ubyte.gz 4542 bytes.
Extracting data/train-images-idx3-ubyte.gz
Extracting data/train-labels-idx1-ubyte.gz
Extracting data/t10k-images-idx3-ubyte.gz
Extracting data/t10k-labels-idx1-ubyte.gz
2018-05-26 08:59:55.416198: I tensorflow/core/platform/cpu_feature_guard.cc:140] Your CPU supports instructions that this TensorFlow binary was not compiled to use: FMA
Initialized!
Step 0 (epoch 0.00), 8.6 ms
Minibatch loss: 8.334, learning rate: 0.010000
Minibatch error: 85.9%
Validation error: 84.6%
Step 100 (epoch 0.12), 449.7 ms
Minibatch loss: 3.240, learning rate: 0.010000
Minibatch error: 4.7%
Validation error: 7.6%
Step 200 (epoch 0.23), 443.6 ms
Minibatch loss: 3.371, learning rate: 0.010000
Minibatch error: 10.9%
Validation error: 4.7%
Step 300 (epoch 0.35), 443.7 ms
Minibatch loss: 3.170, learning rate: 0.010000
Minibatch error: 6.2%
Validation error: 3.2%
Step 400 (epoch 0.47), 444.7 ms
Minibatch loss: 3.222, learning rate: 0.010000
Minibatch error: 6.2%
Validation error: 2.6%
Step 500 (epoch 0.58), 443.5 ms
Minibatch loss: 3.173, learning rate: 0.010000
Minibatch error: 6.2%
Validation error: 2.7%
Step 600 (epoch 0.70), 452.4 ms
Minibatch loss: 3.125, learning rate: 0.010000
Minibatch error: 4.7%
Validation error: 2.1%
Step 700 (epoch 0.81), 444.8 ms
Minibatch loss: 3.006, learning rate: 0.010000
Minibatch error: 3.1%
Validation error: 2.2%
Step 800 (epoch 0.93), 446.5 ms
Minibatch loss: 3.065, learning rate: 0.010000
Minibatch error: 6.2%
Validation error: 1.9%
Test error: 1.8%
~~~
{: .output}
