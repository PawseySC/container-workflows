---
title: "Running a container for machine learning"
teaching: 10
exercises: 0
questions:
objectives:
keypoints:
---
## Running a container for machine learning

Now we're going to run a container to perform a machine learning benchmark application.  We'll use the popular ML package [TensorFlow](tensorflow.org) to build a convolutional neural network that classifies the [MNIST dataset](http://yann.lecun.com/exdb/mnist/), which is just a large collection of handwritten digits.  It's good dataset to get started with because the data has been formatted and cleaned, so you can focus on learning CNNs instead of dealing with data issues.

First let's get a sample program and the data that it needs:

```
> git clone https://github.com/tensorflow/models.git

Cloning into 'models'...
remote: Counting objects: 17327, done.
remote: Compressing objects: 100% (2/2), done.
remote: Total 17327 (delta 2), reused 2 (delta 2), pack-reused 17323
Receiving objects: 100% (17327/17327), 469.82 MiB | 5.41 MiB/s, done.
Resolving deltas: 100% (10311/10311), done.
Checking connectivity... done.
Checking out files: 100% (2248/2248), done.
```

If this downloaded correctly you can have a look at the script we will use (called `convolutional.py`).  Do this and make sure that it's there:

```
> ls models/tutorials/image/mnist
BUILD  convolutional.py  __init__.py
```

If you can see it, then we have some minor editing to complete. We need to edit `convolutional.py` to set NUM_EPOCS to 1 (since we don’t have a GPU here, this will take forever at the current value of 10).  If you need assistance with this in a class, call a class assistance, otherwise have a look at (link).

Find the line with the variable `NUM_EPOCHS`, you should see that it's set to 10. Make the change to 1 and save the file. It now will look like the following:

```
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
```

Now we can get Docker involved, and there's only one command to start Docker, get the container (Tensorflow) and have it see our data directory and it's this:

```
> docker run --rm -it -v /home/ubuntu/models/tutorials/image/mnist/:/notebooks tensorflow/tensorflow python convolutional.py

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
Minibatch loss: 3.006,learning rate: 0.010000
Minibatch error: 3.1%
Validation error: 2.2%
Step 800 (epoch 0.93), 446.5 ms
Minibatch loss: 3.065, learning rate: 0.010000
Minibatch error: 6.2%
Validation error: 1.9%
Test error: 1.8%
```

## Building our own ML container ##

Let's assume we need some additional Python packages for our ML code to run, but they are present in the stock PyTorch image.  We can simply build our image and add them.  To do this, we'll need to write a Dockerfile.

Create a new file named `Dockerfile` and add the following lines to it:

```
FROM tensorflow/tensorflow:latest

RUN pip install astropy matplotlib pandas
```

To build our new image we'll use the `docker build` command:

```
> docker build -t bskjerven/tensorflow-ex .
```

Here were giving our image a title; the naming format is `<Docker Hub Account>/<Image Name>`, so you can substitute in your own details.  You'll need to set up your own DockerHub account if you want to push this image to the cloud later.

We can now run the container just as before, but let's test it first:

```
> docker run bskjerven/tensorflow-ex python -c "import astropy; import pandas; import matplotlib"
```

All we're doing in this example is running the Python interpreter inside our PyTorch container and importing a few modules to test that it works.  We can use the same command as before, just with our new container:

```
> docker run --rm -it -v /home/ubuntu/models/tutorials/image/mnist/:/notebooks bskjerven/tensorflow-ex python convolutional.py
```
