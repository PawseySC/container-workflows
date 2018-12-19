---
title: "Introduction to Docker"
teaching: 20
exercises: 0
questions:
objectives:
- Learn how to run images, either one-off or as a running service

keypoints:
---

## Docker Containers ##

Docker is a tool that allows you to easily create, deploy, and run applications on any architecture.  It does this via something called **containers**, which is a way for you to package up an application and all of its dependecies, into a single object that's easy to track, manage, share, and deploy.

### Containers vs Virtual Machines ###

Many of you have probably used a VM, so you're actually already familiar with some of the concepts of a container.

![Containers vs. VMs]({{ page.root }}/fig/container_vs_vm.png)

The key difference here is that VMs virtualise ***hardware*** while containers virtualise ***operating systems***.  There are other differences (and benefits)

* Containers are lighter weight (less CPU and memory usage, faster start-up times)
* More portable
* Modular (can easily combine multiple containers that work together)

### Containers and your workflow ###

There are a number of reasons for using containers in your daily work:

* Provide a consistent testing environment
* Data reproducibility/provenance
* Cross-platform compatibility
* Simplified software dependencies and management
* Scalability

A few examples of how containers are being used at Pawsey

* Bioinformatics workflows
* RStudio & JupyterHub
* Webservers
* HPC workflows (via Shifter)
* Machine Learning 
* Python apps in radio astronomy

Here's an overview of what a workflow might look like:

![Docker Workflow]({{ page.root }}/fig/docker_workflow.png)

### Terminology ###

An **image** is a file (or set of files) that contains the application and all its dependencies, libraries, run-time systems, etc. required to run.  You can copy images around, upload them, download them etc.

A **container** is an instantiation of an image.  That is, it's a process that Docker creates and starts up, and an image is run inside a container.  You can run multiple containers from the same image, much like you might run the same application with different options or arguments.

In general, an image corresponds to a file, a container corresponds to a process.

### Running a simple command in a container ###

Let's run a simple command:

```
> docker run ubuntu cat /etc/lsb-release
Unable to find image 'ubuntu:latest' locally
latest: Pulling from library/ubuntu
124c757242f8: Pull complete
9d866f8bde2a: Pull complete
fa3f2f277e67: Pull complete
398d32b153e8: Pull complete
afde35469481: Pull complete
Digest: sha256:de774a3145f7ca4f0bd144c7d4ffb2931e06634f11529653b23eba85aef8e378
Status: Downloaded newer image for ubuntu:latest
DISTRIB_ID=Ubuntu
DISTRIB_RELEASE=18.04
DISTRIB_CODENAME=bionic
DISTRIB_DESCRIPTION="Ubuntu 18.04.1 LTS"

```
Here's what we've done:

* Downloaded an Ubuntu Docker image
* Created a container from our Ubuntu image
* The command we've run inside the Ubuntu container is `cat /etc/lsb-release`, which simply prints some info about the operating system

Docker images have a _name_ and a _tag_. The default for the tag is 'latest', and can be omitted (but be careful...more on this later). If you ask docker to run an image that is not present on your system, it will download it from [dockerhub.com](dockerhub.com) first, then run it.

Most Linux distributions have pre-built images available on dockerhub, so you can readily find something to get you started. Let's start with the official Ubuntu linux image, and run a simple 'hello world'. The **docker run** command takes options first, then the image name, then the command and arguments to run follow it on the command line:


Note in our example Docker uses the 'ubuntu:latest' tag, since we didn't specify what version we want.  We can specify a specific version of ubuntu like this:

```
> docker run ubuntu:17.04 cat /etc/lsb-release
DISTRIB_ID=Ubuntu
DISTRIB_RELEASE=17.04
DISTRIB_CODENAME=zesty
DISTRIB_DESCRIPTION="Ubuntu 17.04"
```

Docker caches images on your local disk, so the next time you need to run your container it will be faster:

```
> docker run ubuntu /bin/echo 'hello world'
hello world
```

You can list all Docker containers on your system with

```
> docker ps -a
```

The `-a` flag prints all containers (those currently running and any stopped containers)

Similarly, you can list all docker images you have with

```
> docker images
```

### Running an interactive command in an image ###
Docker has the option to run containers interactively.  While this is convenient (and useful for debugging), in general you shouldn't use this model as your standard way of working with containers.  To run interactively, we just need to use the `-i` and `-t` flags, or `-it` for brevity:

```
> docker run -i -t ubuntu /bin/bash
root@c69d6f8d89bd:/# id
uid=0(root) gid=0(root) groups=0(root)
root@c69d6f8d89bd:/# ls
bin   dev  home  lib64  mnt  proc  run   srv  tmp  var
boot  etc  lib   media  opt  root  sbin  sys  usr
> exit # or hit control-D
```

The `-t` and `-i` options make sure we allocate a terminal to the container, and keep its STDIN (standard input) open.

As you can see, you have root access in your container, and you are in what looks like a normal linux system. Now you can do whatever you like, e.g. install software and develop applications, all within the container of your choice.

Note there is also a command to just download a container image without running it:

```
docker pull ubuntu
```

### Starting a long-running service in a container ###
Containers are useful for running services, like web-servers etc. Many come packaged from the developers, so you can start one easily, but first you need to find the one you want to run. You can either search on [dockerhub.com](dockerhub.com), or you can use the **docker search** command. Nginx is a popular web-server, let's look for that:

```
> docker search nginx
NAME                      DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
nginx                     Official build of Nginx.                        4719      [OK]       
jwilder/nginx-proxy       Automated Nginx reverse proxy for docker c...   877                  [OK]
richarvey/nginx-php-fpm   Container running Nginx + PHP-FPM capable ...   311                  [OK]
million12/nginx-php       Nginx + PHP-FPM 5.5, 5.6, 7.0 (NG), CentOS...   76                   [OK]
webdevops/php-nginx       Nginx with PHP-FPM                              63                   [OK]
maxexcloo/nginx-php       Framework container with nginx and PHP-FPM...   58                   [OK]
bitnami/nginx             Bitnami nginx Docker Image                      20                   [OK]
gists/nginx               Nginx on Alpine                                 8                    [OK]
evild/alpine-nginx        Minimalistic Docker image with Nginx            8                    [OK]
million12/nginx           Nginx: extensible, nicely tuned for better...   8                    [OK]
maxexcloo/nginx           Framework container with nginx installed.       7                    [OK]
webdevops/nginx           Nginx container                                 7                    [OK]
1science/nginx            Nginx Docker images based on Alpine Linux       4                    [OK]
ixbox/nginx               Nginx on Alpine Linux.                          3                    [OK]
drupaldocker/nginx        NGINX for Drupal                                3                    [OK]
yfix/nginx                Yfix own build of the nginx-extras package      2                    [OK]
frekele/nginx             docker run --rm --name nginx -p 80:80 -p 4...   2                    [OK]
servivum/nginx            Nginx Docker Image with Useful Tools            2                    [OK]
dock0/nginx               Arch container running nginx                    2                    [OK]
blacklabelops/nginx       Dockerized Nginx Reverse Proxy Server.          2                    [OK]
xataz/nginx               Light nginx image                               2                    [OK]
radial/nginx              Spoke container for Nginx, a high performa...   1                    [OK]
tozd/nginx                Dockerized nginx.                               1                    [OK]
c4tech/nginx              Several nginx images for web applications.      0                    [OK]
unblibraries/nginx        Baseline non-PHP nginx container                0                    [OK]
```

The official build of Nginx seems to be very popular, let's go with that:

```
> docker run -p 8080:80 nginx
Unable to find image 'nginx:latest' locally
latest: Pulling from library/nginx
386a066cd84a: Pull complete
386dc9762af9: Pull complete
d685e39ac8a4: Pull complete
Digest: sha256:3861a20a81e4ba699859fe0724dc6afb2ce82d21cd1ddc27fff6ec76e4c2824e
Status: Downloaded newer image for nginx:latest
```

Note the ```-p 8080:80``` option. That tells docker to map port 80 in the container to port 8080 on the host, so you can communicate with it.

Note also that we didn't tell docker what program to run, that's baked into the container in this case. More on that later.

Now, go to your browser and enter **localhost:8080** in the address bar (or your VM's IP address), you should see a page with a **Welcome to nginx!** message. On your terminal where you ran the docker command, you'll see some log information:

```
172.17.0.1 - - [30/Nov/2016:18:07:59 +0000] "GET / HTTP/1.1" 200 612 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10.11; rv:49.0) Gecko/20100101 Firefox/49.0" "-"
2016/11/30 18:07:59 [error] 7#7: *1 open() "/usr/share/nginx/html/favicon.ico" failed (2: No such file or directory), client: 172.17.0.1, server: localhost, request: "GET /favicon.ico HTTP/1.1", host: "localhost:8080"
172.17.0.1 - - [30/Nov/2016:18:07:59 +0000] "GET /favicon.ico HTTP/1.1" 404 169 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10.11; rv:49.0) Gecko/20100101 Firefox/49.0" "-"
172.17.0.1 - - [30/Nov/2016:18:07:59 +0000] "GET /favicon.ico HTTP/1.1" 404 169 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10.11; rv:49.0) Gecko/20100101 Firefox/49.0" "-"
2016/11/30 18:07:59 [error] 7#7: *1 open() "/usr/share/nginx/html/favicon.ico" failed (2: No such file or directory), client: 172.17.0.1, server: localhost, request: "GET /favicon.ico HTTP/1.1", host: "localhost:8080"
```

That's a good start, but you now have a terminal tied up with nginx, and if you hit CTRL-C in that terminal, your web-server dies. We can run it in the background instead:

```
> docker run -d -p 8080:80 nginx
48a2dca14407484ca4e7f564d6e8c226d8fdd8441e5196577b2942383b251106
```

Go back to your browser, reload **localhost:8080** (or IP address), and you should get the page loaded again.

That nice string is the _container-ID_, which we can use to get access to its logfiles with the **docker logs** command:

```
> docker logs --follow 48a2dca14407484ca4e7f564d6e8c226d8fdd8441e5196577b2942383b251106
172.17.0.1 - - [30/Nov/2016:18:18:40 +0000] "GET / HTTP/1.1" 304 0 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10.11; rv:49.0) Gecko/20100101 Firefox/49.0" "-"
```

If you hit CTRL-C now, your container is still running, in the background. You can see this with the **docker ps** command:

```
> docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                           NAMES
48a2dca14407        nginx               "nginx -g 'daemon off"   3 minutes ago       Up 3 minutes        443/tcp, 0.0.0.0:8080->80/tcp   pensive_booth
```

You can open a shell into the running container, if you wish. This can be handy for debugging. Let's take a look at the processes running in our nginx server:

```
> docker exec -t -i pensive_booth /bin/bash
root@48a2dca14407:/# ps auxww | grep ngin                                                                                                                
root         1  0.0  0.0  31764  5164 ?        Ss   18:58   0:00 nginx: master process nginx -g daemon off;
nginx        7  0.0  0.0  32156  2900 ?        S    18:58   0:00 nginx: worker process
root        15  0.0  0.0  11128  1036 ?        S+   18:59   0:00 grep ngin
```

So now you've started it, how do you stop it? Use the **docker stop** command! You can give it either the container-ID or the name. You'll notice docker has made up a name for you (_pensive___booth_ in this case).

```
> docker stop pensive_booth
pensive_booth
> docker ps # check that it's gone
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
```

### Conclusion ###
You now know how to find a container that you want to run. You can start it, re-start it, run it as a detached service, attach a terminal to it for debugging, view the logs externally and even map ports between it and your host machine.

### Best practices ###

- prefer official images over those built by third-parties. Docker runs with privileges, so you have to be a bit careful what you run
- good online documentation on Docker commands can be found at [Docker run reference](https://docs.docker.com/engine/reference/run/) and related pages
