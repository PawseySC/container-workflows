---
title: "RStudio deployment for fun and profit"
teaching: 10
exercises: 0
questions:
objectives:
keypoints:
---
## RStudio Example  ##

R is a popular language in bioinformatics, particularly because of its statistical packages.  It often requires installing a large number of dependencies, and installing these on an HPC system can be tedious.

Instead we can use an R container to simplify the process.

## Rocker ##

The group [Rocker](https://hub.docker.com/r/rocker) has published a large number of R images we can use, including a Rstudio image.  To begin, we'll create a new directory to work in and start up an Rstudio container:

```
mkdir $HOME/r-example
mkdir $HOME/r-example/data
cd $HOME/r-example
docker pull rocker/tidyverse:3.5
```

We can now start this up:

```
docker run -d -p 8787:8787 -v `pwd`/data:/home/rstudio/data rocker/tidyverse
```

Here we're openning up a port so we can access the Rtudio server remotely.  You just need to open a web browser and point it to

```
<your-nimbus-ip>:8787
```

You should see a password prompt, and the default login is **rstudio** for **both** the username and password.

##Using RStudio images ##

The above example only provides a bare-bones RStudio image...we want to actually use some R packages.  The following example is based on a workshop at [OzSingleCell2018](https://github.com/IMB-Computational-Genomics-Lab/SingleCells2018Workshop).  We'll use their data for our Docker/Rstudio example.

To begin, let's clone the data (I've created a trimmed down repo with their data)

```
git clone https://github.com/skjerven/rstudio_ex.git
cd rstudio_ex
```

For this example, we'll use an RStudio image I've already built.  R images can take a while to build sometimes, depending on the number of packages and dependencies you're installing.  The Dockerfile I've used is included, and we'll go through it to explain how Docker builds images.
 
```
FROM rocker/tidyverse:3.5

RUN apt-get update -qq && apt-get -y --no-install-recommends install \
      autoconf \
      automake \
      g++ \
      gcc \
      gfortran \
      make \
      && apt-get clean all \
      && rm -rf /var/lib/apt/lists/*

RUN mkdir -p $HOME/.R
COPY Makevars /root/.R/Makevars

RUN Rscript -e "library('devtools')" \
      -e "install_github('Rdatatable/data.table', build_vignettes=FALSE)" \
      -e "install.packages('reshape2')" \
      -e "install.packages('fields')" \
      -e "install.packages('ggbeeswarm')" \
      -e "install.packages('gridExtra')" \
      -e "install.packages('dynamicTreeCut')" \
      -e "install.packages('DEoptimR')" \
      -e "install.packages('http://cran.r-project.org/src/contrib/Archive/robustbase/robustbase_0.90-2.tar.gz', repos=NULL, type='source')" \
      -e "install.packages('dendextend')" \
      -e "install.packages('RColorBrewer')" \
      -e "install.packages('locfit')" \
      -e "install.packages('KernSmooth')" \
      -e "install.packages('BiocManager')" \
      -e "source('http://bioconductor.org/biocLite.R')" \
      -e "biocLite('Biobase')" \
      -e "biocLite('BioGenerics')" \
      -e "biocLite('BiocParallel')" \
      -e "biocLite('SingleCellExperiment')" \
      -e "biocLite('GenomeInfoDb')" \
      -e "biocLite('GenomeInfgoDbData')" \
      -e "biocLite('DESeq')" \
      -e "biocLite('DESeq2')" \
      -e "BiocManager::install(c('scater', 'scran'))" \
      -e "library('devtools')" \
      -e "install_github('IMB-Computational-Genomics-Lab/ascend', ref = 'devel')" \
      && rm -rf /tmp/downloaded_packages
```

The first line, `FROM`, specifies a base image to use.  We could build up a full R image from scratch, but why waste the time.  We can use Rocker's pre-built image to start with and simplify our lives.

`RUN apt-get update` is installing some packages we'll need via Ubuntu's package manager.  Really all we're installing here are compilers.

The next section adds some flags and optionns we want to use when building R packages (`Makvevars`).

The last section is the main R package installation section.  Here we run several different installation methods:

* `install.packages()` is R's standard method for installing packages from CRAN.  We also install the `robustbase` package from source here.
* `biocLite()` is [Bioconductor's](https://bioconductor.org) method for installing packages
* `BiocManager` is a [CRAN package](https://cran.r-project.org/package=BiocManager) for installing bioinformatics software
* `install_github()` is method for installing R packages from GitHub.

We'll skip building this image for now, but we'll show how to do that later.  For now, we'll just use a prebuilt image.  We're also going to use something called `docker-compose` to help with setting up our container.  `docker-compose` is a tool for container orchestration (having multiple containers work together).  Here we'll use it for managing several options we want to use for our Rstudio image.

```
version: "2"

services:
  rstudio:
    restart: always
    image: bskjerven/oz_sc:latest
    container_name: rstudio
    volumes:
      - "$HOME/rstudio_ex/data:/home/rstudio/data"
    ports:
      - 8787:8787
    environment:
      - USER=rstudio
      - PASSWORD=rstudio
```
This yaml file simple tells Docker what image we want to run along with some options (like which volumes to mount, username/password, and what network ports to use).

To begin, make sure you're in the `rstudio_ex` directory (where we cloned the repo).  Simply type

```
docker-compose up
```
Docker will pull the `oz_sc` image first (as it's not present on your system); once that's complete you'll see output from the RStudio server:
```
skj002@turing ~/rstudio_ex ❯❯❯ docker-compose up
Recreating rstudio ... done
Attaching to rstudio
rstudio    | [fix-attrs.d] applying owners & permissions fixes...
rstudio    | [fix-attrs.d] 00-runscripts: applying...
rstudio    | [fix-attrs.d] 00-runscripts: exited 0.
rstudio    | [fix-attrs.d] done.
rstudio    | [cont-init.d] executing container initialization scripts...
rstudio    | [cont-init.d] add: executing...
rstudio    | Nothing additional to add
rstudio    | [cont-init.d] add: exited 0.
rstudio    | [cont-init.d] userconf: executing...
rstudio    | [cont-init.d] userconf: exited 0.
rstudio    | [cont-init.d] done.
rstudio    | [services.d] starting services
rstudio    | [services.d] done.
```

This is annoying, though...we need our terminal back.  Luckily, Docker lets you run processes in the background.  Kill the RStudio process with `Ctl-C`, and the rerun `docker-compose` with the `-d` flag:

```
docker-compose up -d
```
Shortly after that starts, open a webrowser and go to your **http://146.x.x.x:8787**, but with your Nimbus IP.  You should see an Rstudio login, and we've set the username and password to `rstudio`.

Once logged in, you type:

```
source(`data/SC_script.r`)
```

to run the tutorial (it may take a few minutes).  We can refer to the [OzSingleCell2018](https://github.com/IMB-Computational-Genomics-Lab/SingleCells2018Workshop) repo for details on each step.

To stop your Rstudio image, simply type
```
docker-compose down
```
### Conclusion ###
Containers are great way to manage R workflows.  You likely still want to have a local installation of R/Rstudio for some testing, but if you have set workflows, you can use containers to manage them.  You can also provide Rstudio servers for collaborators.

Also, docker-compose is a great way to manage complex Docker commands, as well as coordinating multiple containers.
