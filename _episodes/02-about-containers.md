---
title: "Let's talk about containers"
teaching: 0
exercises: 0
questions:
- "Key question"
objectives:
- "First objective."
keypoints:
- "First key point."
---

### What are containers ###

Before discussing the technical details of what a container is, let's discuss the ***purpose*** of a container.

A container allows us to:

* Isolate an application, and all of its dependencies (code, runtime, libraries, etc.) into a single unit
* Guarantee operation of an application between different users, operating systems, and hardware platforms
* Allows for application reproducability

For those familiar with virtual machines (VMs), this description will sound quite similar.  At the core, containers and VMs provide the same function; isolated, reproducable environments.
However, there are some important differences:

![Container vs. VM]({{ page.root }}/fig/container_vs_vm.png "Container vs. VM")

The key difference is in what each system virtualises; VMs virtualise ***hardware*** whereas containers virtualise ***operating systems***.

**Containers:**

* Uses host kernel (and some libraries & binaries)
* Kernel and other resources are shared amongst other containers (read-only)
* Generally lightweight and more portable

**Virutal Machines:**
* Each VM has its own kernel
* Hypervisor manages sharing hardware between VMs
* Larger overhead (Full OS, divers, etc.)

### Docker ###

![Docker Logo]({{ page.root }}/fig/docker_logo.png "Docker")

There are a number of options for creating, deploying, and using containers, but by far the most common and widely-used is Docker.

Docker provides a framework that allows users to create, share, and manage containers.  There are few key concepts to working with Docker containers:

* **Dockerfile**
  * A recipe that defines how a container is constructed
* **Image**
  * The end prodcut of building a Dockerfile
  * A template that defines how a container will be run
* **Container**
  * A specific instance of an image
* **Registry**
  * Repository for storing images
  * Public (e.g. Docker Hub) and private options

![Docker Workflow]({{ page.root }}/fig/docker_workflow.png "Docker Workflow")

### Docker Basics ###

The focus of this course is not to be a Docker tutorial; however, it is beneficial to cover somee of the more common Docker commands.
Docker provides a CLI that lets users interact with the Docker client, which sends the commands to the Docker engine where they are
interpreted and executed.

`docker help`

`docker COMMAND --help`
* These will provide a full listing of available options and specific `COMMAND` options

`docker run -i -t ubuntu`
* This is a basic command to run a container, which can be broken as follows:
  1. Docker first checks if you already an ubuntu image in your local repository
  2. If not, it is pulled from Docker Hub
  3. Once pulled to your local repository, the Docker engine creates a container based off the pulled image
  4. The `-i` and `-t` flags indicate that the container is to be run interactively, and that it is attaches to a pseudo-tty.
     This is the basic command to run your container in a standard, interactive shell

`docker run ubuntu cat /etc/lsb-release`
* Many Docker images allow for running in an interactive shell, but often we want to run a specific command inside a container.
We can run any valid command (i.e. the command exists in the container) just as we would from the command line.  Here, `cat` simply
dumps the contents of the file `/etc/lsb-release` inside the ubuntu image

`docker ps`
* This produces a list of all currently running containers

`docker images`
* Similar to the `ps` command, this will list all local images

`docker pull alpine`
* This will pull an image named alpine (A lightweight OS designed for running in containers)

`docker pull alpine:3.1`
* Docker images can also be tagged to specify different versions or configurations
* The format is `<name of image>:<tag>`, and `<tag>` can be any valid ASCII

`docker build my_image .`
* Assuming we have a valid Dockerfile, this will build an image based on the contents of the current directoyr (hence the `.`) and name it `my_image`

`docker push my_image`
* Pushes the newly created `my_image` image to Docker Hub (or a private repo if configured)

This is just a quick listing of the common Docker commands you'd use in everyday use.  There are a large number of excellent online tutorials and user guides
if you wish to know more about advanced features of Docker.


### Containers and HPC ###

![Containers & HPC]({{ page.root }}/fig/containers_hpc.png "Container and HPC")

Why can't we just run Docker on Magnus?  At present, Docker presents several challenges for running in a shared, HPC environment:

* **Security**
  * Docker assumes an "all or nothing" security model
  * Users run as root inside containers
* **Batch Systems**
  * Docker (generally) doesn't integrate well with schedulers like SLURM or PBS
* **System Requirements**
  * Docker requires a recent kernel
  * Issues with underlying hardware (GPUs, Infiniband, Lustre)
* **Scalability**
  * Each Docker process fetches a new image

Several options have been developed to enable containers on HPC systems.

### Shifter ###

![Shifter]({{ page.root }}/fig/shifterlogo.png "Shifter Logo")

[Shifter](http://www.nersc.gov/users/software/using-shifter-and-docker/using-shifter-at-nersc) is a container technology developed by NERSC that enables the use of containers on HPC systems,
particularly Cray systems.  At its core it's meant to function just like Docker.

* Users build a custom image (locally or on host HPC system)
* Image pushed to Docker Hub or private reop
* Shifter pulls the image to the HPC filesystem
* Shifter is integrated with workload manager to allow users to run non-inteeractively

Shifter is composed of two parts:
* **Image Gateway**
  * A tool that user use to import images to the HPC system
  * Converts them to a format used by the Shifter runtime
  * Conversion essentially "flattens" image layers, and packs the image into a desired format (squashfs or ext4)
* **udiRoot**
  * User Defined Image
  * The Shifter runtime component
  * Instantiates and destroys Shifter containers
  * Handles the "chroot-ing" of user processes
  * Sets up volume mounts

Beyond allowing users to run Docker images on a Cray HPC system, Shifter also provides application performance in certain cases.  Python applications
in particular benefit from running inside a container.  Below are the results of running an application called Pynamic in several modes on different
Pawsey filesystems.  Pynamic is a synthetic Python code that creates a large number of shared object files, each with a random number of mathematical
functions.  The results show running Pynamic on several parallel filesytems, as well as copying the Pynamic libraries to a local ramdisk, and finally
running directly in a container with Shifter:

![Pyanmic Benchmark]({{ page.root }}/fig/pynamic_bench.png "Pyanmic Benchmark")

Using Shifter is outside the scope of this course, but if you wish to use it, see the following resources:

[Shifter at Pawsey](https://support.pawsey.org.au/documentation/display/US/Shifter)

[Using Shifter at NERSC](http://www.nersc.gov/users/software/using-shifter-and-docker/using-shifter-at-nersc/)

Pawsey currently has deployed Shifter on both of its Cray systems, Magnus and Galaxy.  For those who wish to use containers on Zeus, there is another option:

### Singularity ###

![Singularity]({{ page.root }}/fig/singularity_logo.png "Singularity Logo")

[Singularity](https://singularity.lbl.gov) is another container technology designed for HPC use.  It functions much the same as Shifter, in that it provides
a framework for safely and securely using containers in a shared, HPC environment.

Singularity works by virtualising namespaces, and it ***only*** does so with the namespaces it started with, meaning a user has the exact same permissions inside
a container as he or she would outside of the container.

There are number of important differences between Shifter and Singularity:

**Docker Interoperability**
* Shifter is primarily used to pull and convert Docker images, whereas Singularity is completely independent of Docker.  Singularity uses its own image format,
but is capable of importing Docker images and converting them

**MPI Support**
* Access to high-speed interconnects is a defining feature of most HPC applications.  Building containers to make use of these is not trivial, but both Shifter and
Singularity have support for MPI.  Shifter, being designed for Cray, relies on the MPICH implementation, while Singularity uses OpenMPI.  There is work being done
on both sides to support multiple MPI implementations.

**GPU Support**
Singularity supports GPUs by default, and a version of Shifter developed by CSCS has GPU support, as well.

Singularity has a similar CLI to Docker:

`singularity exec`
  * Spawn and execute a command within a container
  * `singularity exec centos7.img python hello.py`
  * NOTE: Not the same as Docker `exec`

`singularity build <target> <output container>`
  * Builds Singularity container
  * Can build from either Singularity Hub or Docker Hub
  * `singularity build my_img.simg shub://Pawsey/test_cont`

`singularity pull`
  * Pull a container from a remote repo
  * `singularity pull shub://vsoch/pokemon`

`singularity run`
  * Run a container as an executable
  * Under the hood, a runscript internal to the container is executed
  * `singularity exec centos7.img`
