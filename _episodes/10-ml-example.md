---
title: "Containers for machine learning"
teaching: 20
exercises: 0
questions:
objectives:
- Deploy a machine learning framework with containers, exploiting CPUs and GPUs

keypoints:
---

### Running a container for machine learning ###

Now we're going to run a container to perform a machine learning benchmark application.  We'll use the popular ML package [TensorFlow](https://www.tensorflow.org) to build a convolutional neural network (CNN) that classifies the [MNIST dataset](http://yann.lecun.com/exdb/mnist/), which is just a large collection of handwritten digits.  It's good dataset to get started with because the data has been formatted and cleaned, so you can focus on learning CNNs instead of dealing with data issues.

First let's get a sample program and the data that it needs, from the tensorflow/models Git repo:

```
> mkdir ml_example
> cd ml_example

> wget --no-check-certificate https://raw.githubusercontent.com/tensorflow/models/master/tutorials/image/mnist/BUILD 
> wget --no-check-certificate https://raw.githubusercontent.com/tensorflow/models/master/tutorials/image/mnist/__init__.py
> wget --no-check-certificate https://raw.githubusercontent.com/tensorflow/models/master/tutorials/image/mnist/convolutional.py
```

We now have some minor editing to complete. We need to edit `convolutional.py` to set `NUM_EPOCHS` to `1` (down from the current value of 10, to make the calculation run in a reasonably short amount of time). We can make the change from the shell using the tool `sed`, noting that on a Mac, note that `sed -i` needs to be changed to `sed -i ""`:

```
> sed -i 's/NUM_EPOCHS *=.*/NUM_EPOCHS = 1/' convolutional.py
```

In alternative, you can use your favourite text editor to edit the value assigned to `NUM_EPOCHS` from `10` to `1`.

Now we can get Docker involved. Eventually a single command is required to start Docker, get the container for Tensorflow and have it work on our data directory and it's this:

```
> docker run --rm -v `pwd`:/notebooks -w /notebooks tensorflow/tensorflow:1.13.1 python convolutional.py
Unable to find image 'tensorflow/tensorflow:1.13.1' locally
1.13.1: Pulling from tensorflow/tensorflow
7b722c1070cd: Pull complete 
5fbf74db61f1: Pull complete 
...
Digest: sha256:40844012558fe881ec58faf1627fd4bb3f64fe9d46a2fd8af70f139244cfb538
Status: Downloaded newer image for tensorflow/tensorflow:1.13.1
WARNING:tensorflow:From /usr/local/lib/python2.7/dist-packages/tensorflow/python/framework/op_def_library.py:263: colocate_with (from tensorflow.python.framework.ops) is deprecated and will be removed in a future version.
Instructions for updating:
Colocations handled automatically by placer.
WARNING:tensorflow:From convolutional.py:226: calling dropout (from tensorflow.python.ops.nn_ops) with keep_prob is deprecated and will be removed in a future version.
Instructions for updating:
Please use `rate` instead of `keep_prob`. Rate should be set to `rate = 1 - keep_prob`.
2019-04-24 06:32:26.488531: I tensorflow/core/platform/cpu_feature_guard.cc:141] Your CPU supports instructions that this TensorFlow binary was not compiled to use: FMA
2019-04-24 06:32:26.506593: I tensorflow/core/platform/profile_utils/cpu_utils.cc:94] CPU Frequency: 2299950000 Hz
2019-04-24 06:32:26.507967: I tensorflow/compiler/xla/service/service.cc:150] XLA service 0x44cad90 executing computations on platform Host. Devices:
2019-04-24 06:32:26.508337: I tensorflow/compiler/xla/service/service.cc:158]   StreamExecutor device (0): <undefined>, <undefined>
Successfully downloaded train-images-idx3-ubyte.gz 9912422 bytes.
Successfully downloaded train-labels-idx1-ubyte.gz 28881 bytes.
Successfully downloaded t10k-images-idx3-ubyte.gz 1648877 bytes.
Successfully downloaded t10k-labels-idx1-ubyte.gz 4542 bytes.
Extracting data/train-images-idx3-ubyte.gz
Extracting data/train-labels-idx1-ubyte.gz
Extracting data/t10k-images-idx3-ubyte.gz
Extracting data/t10k-labels-idx1-ubyte.gz
Initialized!
Step 0 (epoch 0.00), 6.9 ms
Minibatch loss: 8.334, learning rate: 0.010000
Minibatch error: 85.9%
Validation error: 84.6%
Step 100 (epoch 0.12), 268.1 ms
Minibatch loss: 3.237, learning rate: 0.010000
Minibatch error: 4.7%
Validation error: 7.3%
Step 200 (epoch 0.23), 263.1 ms
Minibatch loss: 3.357, learning rate: 0.010000
Minibatch error: 9.4%
Validation error: 4.4%
Step 300 (epoch 0.35), 292.6 ms
Minibatch loss: 3.143, learning rate: 0.010000
Minibatch error: 3.1%
Validation error: 3.2%
Step 400 (epoch 0.47), 264.3 ms
Minibatch loss: 3.224, learning rate: 0.010000
Minibatch error: 6.2%
Validation error: 2.7%
Step 500 (epoch 0.58), 264.5 ms
Minibatch loss: 3.184, learning rate: 0.010000
Minibatch error: 4.7%
Validation error: 2.4%
Step 600 (epoch 0.70), 268.9 ms
Minibatch loss: 3.136, learning rate: 0.010000
Minibatch error: 3.1%
Validation error: 2.1%
Step 700 (epoch 0.81), 268.4 ms
Minibatch loss: 2.958, learning rate: 0.010000
Minibatch error: 0.0%
Validation error: 2.1%
Step 800 (epoch 0.93), 267.0 ms
Minibatch loss: 3.076, learning rate: 0.010000
Minibatch error: 7.8%
Validation error: 2.1%
Test error: 2.0%
```

### Building our own ML container ###

Let's assume we need some additional Python packages for our ML code to run, but they are present in the stock PyTorch image.  We can simply build our image and add them.  To do this, we'll need to write a Dockerfile.

Create a new file named `Dockerfile` and add the following lines to it:

```
FROM tensorflow/tensorflow:1.13.1

RUN pip install astropy matplotlib pandas
```

To build our new image we'll use the `docker build` command:

```
> docker build -t tensorflow-ex .
```

We can now run the container just as before, but let's test it first, by running the Python interpreter inside our ML container and importing a few modules to test that it works:

```
> docker run tensorflow-ex python -c "import astropy; import pandas; import matplotlib"
```

Now let us re-run the same command as before, just with our new container:

```
> docker run --rm -v `pwd`:/notebooks -w /notebooks tensorflow-ex python convolutional.py
```

### Run a ML container on HPC ###


