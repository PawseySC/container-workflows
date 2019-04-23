---
title: "Build your own container image with Docker"
teaching: 20
exercises: 0
questions:
objectives:
- Learn what is a Dockerfile and its basic syntax
- Learn how to build a container and push it to a web repository

keypoints:
---

### What is a Dockerfile? ###

A Dockerfile is a recipe to build a container image with Docker. It is basically a collection of the standard shell commands you would use to build your software through prompt; in addition, it contains Docker-specific instructions that handle the build process. We will see some examples below.


### Let's write a Dockerfile ###

We will build a very simple container image: a Ubuntu box featuring the `nano` text editor. It won't be particularly useful in itself, but its Dockerfile will contain all of the basic Docker instructions that can also be used to build more complicated images.

First let us create a directory where we'll store the Dockerfile. This directory will be the so called Docker _build context_. Files in this directory will be included in the build process and in the final image, making the build process longer and the image larger. Therefore, we want to include only those strictly required for the build, even none if possible.

```
> mkdir nano_dockerfile
> cd nano_dockerfile
```

Now use your favourite text editor to create a file named _Dockerfile_ and edit it. Here is its contents:

```
FROM ubuntu:18.04
  
MAINTAINER Marco De La Pierre <marco.delapierre@pawsey.org.au>

ARG NANO_VER="2.9.3-2"

RUN apt-get clean all && \
    apt-get update && \
    apt-get upgrade -y && \
    apt-get install -y \
        nano=${NANO_VER} \
    && apt-get clean all && \
    apt-get purge && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

ENV EDITOR=nano

VOLUME ["/data"]
WORKDIR /data

CMD ["/bin/bash"]
```

* `FROM`: compulsory, it provides the starting image we will use to build our customised one;
* `MAINTAINER`: details of the person who wrote the Dockerfile, optional;
* `ARG`: used to set temporary values that will be used during the build process, and that might need to be changed in future builds; a common use is to specify package versions, like in this sample case;
* `RUN`: this is the most used instruction, that alllows to run most shell commands during the build. Multiple `RUN` instructions are often found in a single Dockerfile. Each `RUN` instruction creates an additional _layer_ in the container image. More on this later;
* `ENV`: required to set environment variables that will persist at runtime in the container; in this specific case, the `EDITOR` variable is created as an example.  
  **DO NOT** use `RUN export <..>` to this end;
* `VOLUME`: creates a mount point ready to be used for mounting external (e.g. host) volumes; creates the corresponding directory if not existing;
* `WORKDIR`: changes directory to the specified path; the last current directory in the build will be the working directory in the running container.  
  **NOTE**: if you use instead `RUN cd <..>`, the changed directory will only persist within that `RUN` instruction, and then be lost in subsequent build steps/layers;
* `CMD`: specifies the default command to be executed with the container. `bash` is the default anyway for Ubuntu containers, but it's good to be aware of this syntax.

Note that uses for `ARG` and `ENV` in this example are not really required, they are included just to showcase these instructions.


### Layers in a container image ###

Note how the `RUN` instruction above is used to execute a sequence of commands to:

- update preexisting packages
- install `nano`
- clean build directories

We have concatenated all these commands in one using the `&&` linux operator, and then the `\` symbol to break them into multiple lines for readability.

We could have used one `RUN` instruction per command, so why concatenating instead?

Well, each `RUN` creates a distinct layer in the final image, increasing its size. It is a good practice to use as few layers, and thus `RUN` instructions, as possible, to keep the image size smaller.


### Building the image ###

Once the Dockerfile is ready, let us build the image:

```
> docker build -t marcodelapierre/nano:2.9.3-2 .
Sending build context to Docker daemon  2.048kB
Step 1/8 : FROM ubuntu:18.04
 ---> 93fd78260bd1

Step 8/8 : CMD ["/bin/bash"]
 ---> Running in c8e8e1f38299
Removing intermediate container c8e8e1f38299
 ---> 92b3a4b7a0f5
Successfully built 92b3a4b7a0f5
Successfully tagged nano:2.9.3-2
```

Here, `.` is the location of the build context (i.e. the directory for the Dockerfile).  
The `-t` flag is used to specify the image name (compulsory) and tag (optional).

Any lowercase alphanumeric string can be used as image name. Using the format `<Your Docker Hub account>/<Image name>` as in this example allows to push the built image to your Docker Hub repository.  
The image tag (following the colon) can be used to maintain a set of different image versions on Docker Hub, and is a key feature in enabling reproducibility of your computations through containers.

Finally, note that `docker build` has an option to change at build time the value of temporary `ARG` variables set in the Dockerfile: `--build-arg <variable>=<value>`.


### Pushing the image to Docker Hub ###

If you have a (free) Docker Hub account, `marcodelapierre` in this case, you are now ready to push your newly created image to the Docker Hub web repository:

```
> docker push marcodelapierre/nano:2.9.3-2 
The push refers to repository [docker.io/marcodelapierre/nano]
b7b5c20b2e8d: Pushed 
62c23d30c50a: Pushed 

2.9.3-2: digest: sha256:1efb81d9eabb81e0da9bda95fca11f9140c5934ca965f53be986463ae21218f6 size: 1568
```
Congratulations! Anyone can now pull your image in his computer using Docker.


### Base images for Python and Bioconda ###

[continuumio/miniconda](https://hub.docker.com/r/continuumio/miniconda/tags) and [continuumio/miniconda3](https://hub.docker.com/r/continuumio/miniconda3/tags) are Docker images provided by the maintainers of the [Anaconda](https://anaconda.org) project. They ship with Python 2 and 3, respectively, as well as `pip` and `conda` to install and manage packages. At the time of writing, the most recent version is **4.5.11**.

Among other use cases, these base images can be very useful for maintaining Python containers, as well as bioinformatics containers based on the [Bioconda](https://bioconda.github.io) project.


### Base images for R ###

The [Rocker Project](https://www.rocker-project.org) maintains a number of good R base images. Of particular relevance is [rocker/tidyverse](https://hub.docker.com/r/rocker/tidyverse/tags), which embeds the basic R distribution, an RStudio web-server installation and the [tydiverse](https://www.tidyverse.org) collection of packages for data science, that are also quite popular across the bioinformatics community of [Bioconductor](https://www.bioconductor.org). At the time of writing, the most recent version is **3.5.1**.

Other more basic images are [rocker/r-ver](https://hub.docker.com/r/rocker/r-ver/tags) (R only) and [rocker/rstudio](https://hub.docker.com/r/rocker/rstudio/tags) (R + RStudio).


### More Dockerfile instructions ###

Several other instructions are available, that we haven't covered in this introduction. You can find more information on them at [Dockerfile reference](https://docs.docker.com/engine/reference/builder/). Just to mention a few possibilities:

- `ADD`/`COPY` to embed files/directories from your computer into the container image;
- `EXPOSE` to make the container listen on specified network ports;
- `CMD`/`ENTRYPOINT` to tweak the default behaviour of the executing container;
- `USER` to switch user.


### Conclusion ###
Dockerfiles use specific instructions to direct the image building process. Once created, a container image can be pushed to a Web repository for release.


### Best practices ###

- for stand-alone packages, it is suggested to use the policy of one container per package
- for Python or R pipelines, it may be handier to use the policy of a single container for the entire pipeline
- [Best practices for writing Dockerfiles](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/) are found in the Docker website
