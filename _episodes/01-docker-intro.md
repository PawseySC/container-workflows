## Objective ##

Learn how to run images, either one-off or as a running service

### Terminology ###

An **image** is a file (or set of files) that contains the application and all its dependencies, libraries, run-time systems, etc. required to run.  You can copy images around, upload them, download them etc.

A **container** is an instantiation of an image.  That is, it's a process started from an image.  You can run multiple containers from the same image, much like you might run the same application with different options or arguments.

So, an image corresponds to files, a container corresponds to processes.

### Running a simple command in a container ###

Docker images have a _name_ and a _tag_. The default for the tag is 'latest', and can be omitted. If you ask docker to run an image that is not present on your system, it will download it from [dockerhub.com](dockerhub.com) first, then run it.

Most Linux distributions have pre-built images available on dockerhub, so you can readily find something to get you started. Let's start with the official Ubuntu linux image, and run a simple 'hello world'. The **docker run** command takes options first, then the image name, then the command and arguments to run follow it on the command line:

```
> docker run ubuntu /bin/echo 'hello world'
Unable to find image 'ubuntu:latest' locally
latest: Pulling from library/ubuntu
af49a5ceb2a5: Pull complete 
8f9757b472e7: Pull complete 
e931b117db38: Pull complete 
47b5e16c0811: Pull complete 
9332eaf1a55b: Pull complete 
Digest: sha256:3b64c309deae7ab0f7dbdd42b6b326261ccd6261da5d88396439353162703fb5
Status: Downloaded newer image for ubuntu:latest
hello world
```

Note docker uses the 'ubuntu:latest' tag, since we didn't specify what version we want.  We can specify a specific version of ubuntu like this:

```
docker run ubuntu:17.04 cat /etc/lsb-release
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

### Running an interactive command in an image ###
Docker has the option to run containers interactively.  While this is convenient (and useful for debugging), in general you shouldn't use this model as your standard way of working with containers.  To run interactively, we just need to use the `-i` and `-t` flags:

```
> docker run -i -t ubuntu /bin/bash
root@c69d6f8d89bd:/# id
uid=0(root) gid=0(root) groups=0(root)
root@c69d6f8d89bd:/# ls
bin   dev  home  lib64  mnt  proc  run   srv  tmp  var
boot  etc  lib   media  opt  root  sbin  sys  usr
> exit # or hit control-D
```

The `-t` and `-i` options make sure we can attach a terminal to the container, and we tell it to run our favourite shell as the application.

As you can see, you have root access in your container, and you are in what looks like a normal linux system. Now you can do whatever you like, e.g. install software and develop applications, all within the container of your choice.

### Making changes to a container ###
In an interactive shell in a container, you can change the container contents. But the changes do not persist once you exit the container. If you re-run the image, you get a _new_ container, not a re-run of the one you modified. Let's create a file in /tmp, then exit and restart the image, and look for the file:

```
> docker run -t -i ubuntu /bin/bash
root@1688f55c3418:/# touch /tmp/a-file.txt
root@1688f55c3418:/# ls -l /tmp/a-file.txt 
-rw-r--r-- 1 root root 0 Nov 30 17:50 /tmp/a-file.txt
root@1688f55c3418:/# exit
exit
> docker run -t -i ubuntu /bin/bash
root@97b1e86df1f1:/# ls -l /tmp
total 0
root@97b1e86df1f1:/# exit
```

There are, of course, ways to make persisitent changes to a container. More on that later.

So, can you get back to a container you modified before? Yes, actually, you can! Once a container exits docker keeps it cached for a while, so you can recover it. The ```docker ps``` command will tell you which containers are running now, and if you give it the ```--all``` flag (`-a` also works), it tells you about containers that have exited, but still exist in the cache:

```
> docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
> docker ps --all
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                      PORTS               NAMES
97b1e86df1f1        ubuntu              "/bin/bash"              5 minutes ago       Exited (0) 5 minutes ago                        backstabbing_brown
1688f55c3418        ubuntu              "/bin/bash"              5 minutes ago       Exited (0) 5 minutes ago                        ecstatic_hugle
c69d6f8d89bd        ubuntu              "/bin/bash"              9 minutes ago       Exited (0) 9 minutes ago                        sad_keller
960588723c36        ubuntu              "/bin/echo 'hello wor"   13 minutes ago      Exited (0) 13 minutes ago                       suspicious_mestorf
ffbb0f60bda6        ubuntu              "/bin/echo 'hello wor"   19 minutes ago      Exited (0) 19 minutes ago                       mad_booth
```

(the container names are made up by docker at random. Use ```--name xyz``` to set the name explicitly yourself)

we know the container we modified was the one before last, with ID 1688f55c3418. We can **start** it again, then **attach** a terminal to it, and look for the file we created in /tmp:

```
> docker start 1688f55c3418
1688f55c3418
> docker attach 1688f55c3418
root@1688f55c3418:/# ls -l /tmp
total 0
-rw-r--r-- 1 root root 0 Nov 30 17:50 a-file.txt
```

In general, you don't want to do this much, it's messy if you run lots of containers. But it's useful to know, just in case you need it.

If you know you don't want to re-start a container afer you've run it, you can tell docker to clean it up automatically when it exits, with the ```--rm`` flag. E.g.:

```
> docker run --rm ubuntu echo 'hello world'
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

Now, go to your browser and enter **localhost:8080** in the address bar, you should see a page with a **Welcome to nginx!** message. On your terminal where you ran the docker command, you'll see some log information:

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

Go back to your browser, reload **localhost:8080**, and you should get the page loaded again.

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

- avoid making too many interactive changes to containers, see later exercises for how to modify containers
- prefer official images over those built by third-parties. Docker runs with privileges, so you have to be a bit careful what you run.