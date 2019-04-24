---
title: "Making Python not awful with containers"
teaching: 10
exercises: 0
questions:
objectives:
- embed a Python app in a container and run it

keypoints:
---

### Why can Python be Awful? ###

Python is a great language for doing all kinds of work.

![Python can get messy]({{ page.root }}/fig/python-complexity-cartoon.png)


### An example of Dockerfile for a Python container ###

This is a quick Python data science example.  We'll build a Python container, add a simple Python app, and run it.

To begin, let's clone another repo:

```
> git clone https://github.com/skjerven/python-demo.git
> cd python-demo
```

There a few files here:

* `Dockerfile`: outlines how we'll build our container

```
# Use an official Python runtime as a parent image
FROM python:3.6-slim

# Set the working directory to /app
WORKDIR /app

# Copy the current directory contents into the container at /app
ADD . /app

# Install any needed packages specified in requirements.txt
RUN pip install -r requirements.txt

# Define environment variable
ENV NAME World

# Run app.py when the container launches
CMD ["python", "my_app.py"]
```

* `requirements.txt`: what python packages we want to install with pip

```
numpy
scipy
scikit-learn
```

* `my_app.py`: a simple python app that builds a decision tree using Scikit-Learn

There are some aspects of this Dockerfile that are worth mentioning:

* we are using the base image `python:3.6-slim`, which comes from the official Python repository on Docker Hub; images in this repo whose tag has `slim` are characterised by a very small size, in this case about 100 MB as opposed to about 400 MB for the `miniconda3` image;
* the Docker instruction `ADD` permits to copy the content of the build context from the host machine inside the container image;
* the `CMD` instruction sets the default behaviour for this container: running the embedded Python app;
* finally, note from the requirement libraries that this is quite a standard scientific Python stack.


### Build and run a dockerised Python app ###

We can simply run:

```
> docker build -t python-demo .
```
to build our container.

After that, we can run it with:

```
> docker run python-demo
```


### Run a Python app on HPC with Shifter ###

We could push the container we created above to Docker Hub, and then pull it on HPC using Shifter; this requires a Docker account, though. Instead, let us run the same app using a publicly available container for scientific Python:

```
> module load shifter
> sg $PAWSEY_PROJECT -c 'shifter pull jupyter/scipy-notebook'
```

Let us write a SLURM script to execute our Python app using this container. Contrary to Docker example above, we'll need to explicitly run the app with the Python interpreter:

```
#!/bin/bash -l

#SBATCH --account=<your-pawsey-project>
#SBATCH --partition=workq
#SBATCH --ntasks=1
#SBATCH --time=00:05:00
#SBATCH --export=NONE
#SBATCH --job-name=python

module load shifter

# clone Git repo with the app
git clone https://github.com/skjerven/python-demo.git
cd python-demo

# run Python app
srun --export=all shifter run jupyter/scipy-notebook python my_app.py
```

Let's change directory to either `$MYSCRATCH` or `$MYGROUP`, e.g.

```
> cd $MYSCRATCH
```

Then, with your favourite text editor, create a `python.sh` script (remember to specify your Pawsey project ID in the script!),

and submit it with SLURM:

```
> sbatch python.sh
```

