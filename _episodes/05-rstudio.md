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

## Building our own R images ##