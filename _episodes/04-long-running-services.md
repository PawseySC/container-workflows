---
title: "Long running services with Docker"
teaching: 20
exercises: 0
questions:
objectives:
- Learn how to run images, either one-off or as a running service

keypoints:
---

### Starting a long-running service in a container ###
Containers are useful for running services, like web-servers etc. Many come packaged from the developers, so you can start one easily, but first you need to find the one you want to run. You can either search on [Docker Hub](https://hub.docker.com), or you can use the **docker search** command. Nginx is a popular web-server, let's look for that:

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
> docker run -p 8080:80 --name=nginx nginx
Unable to find image 'nginx:latest' locally
latest: Pulling from library/nginx
386a066cd84a: Pull complete
386dc9762af9: Pull complete
d685e39ac8a4: Pull complete
Digest: sha256:3861a20a81e4ba699859fe0724dc6afb2ce82d21cd1ddc27fff6ec76e4c2824e
Status: Downloaded newer image for nginx:latest
```

Note the ```-p 8080:80``` option. That tells docker to map port 80 in the container to port 8080 on the host, so you can communicate with it.

We've also introduced a new option: `--name`
Docker will automatically name containers, but the names don't reflect the purpose of the container (e.g. pensive_booth was the name of the nginx container I ran for this example without the `--name` option).  You can name your container whatever you want, but it's helpful to give it a name similar to the container you're using, or the specific service or application you're running in the container.  In this example, we've called our container `nginx`.

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
> docker run -d -p 8080:80 --name=nginx nginx
48a2dca14407484ca4e7f564d6e8c226d8fdd8441e5196577b2942383b251106
```

Go back to your browser, reload **localhost:8080** (or IP address), and you should get the page loaded again.

We can view the logs of our nginx service with the `docker logs` command:

```
> docker logs --follow nginx
172.17.0.1 - - [30/Nov/2016:18:18:40 +0000] "GET / HTTP/1.1" 304 0 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10.11; rv:49.0) Gecko/20100101 Firefox/49.0" "-"
```

This gives us a live look at what is going on in our nginx container (try reloading your webpage and see what the logs look like).  Note that the `follow` option keeps the terminal open and will update the logs in real-time.  If you omit it, `docker logs` would simply display the last few lines of the log file.

If you hit CTRL-C now, your container is still running, in the background. You can see this with the **docker ps** command:

```
> docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                           NAMES
48a2dca14407        nginx               "nginx -g 'daemon off"   3 minutes ago       Up 3 minutes        443/tcp, 0.0.0.0:8080->80/tcp   pensive_booth
```

You can open a shell into the running container, if you wish. This can be handy for debugging. Let's take a look at the processes running in our nginx server:

```
> docker exec -t -i nginx /bin/bash
root@48a2dca14407:/# ps auxww | grep ngin                                                                                                                
root         1  0.0  0.0  31764  5164 ?        Ss   18:58   0:00 nginx: master process nginx -g daemon off;
nginx        7  0.0  0.0  32156  2900 ?        S    18:58   0:00 nginx: worker process
root        15  0.0  0.0  11128  1036 ?        S+   18:59   0:00 grep ngin
```

So now you've started it, how do you stop it? Use the **docker stop** command! 

```
> docker stop nginx
nginx
> docker ps # check that it's gone
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
```


### Docker-Compose ###

Sometimes we may need to run and manage multiple containers (e.g., running an Nginx webserver in front of some other application we are running in a container).

We can run all of these via `docker run` commands, but it may get cumbersome if you have lots of arguments and flags.  We can use a tool called `docker-compose` to orchestrate and manage containers for us.

All we need to do is define the various options and properties for our containers in a YAML file.  Let's create a simple, containerised MySQL/Nginx setup:

```
version: '3'
services:

  # Nginx
  webserver:
    image: nginx:alpine
    container_name: nginx-webserver
    restart: unless-stopped
    tty: true
    ports:
      - "80:80"
      - "443:443"
    networks:
      - appnet

  # MySQL
  db:
    image: mysql:5.7.22
    container_name: db
    restart: unless-stopped
    tty: true
    ports:
      - "3306:3306"
    environment:
      MYSQL_DATABASE: mydb
      MYSQL_ROOT_PASSWORD: password
    volumes:
      - dbdata:/var/lib/mysql
    networks:
      - appnet

  # Docker Volumes
  volumes:
    dbdata:
      driver:local
     
  # Docker Networks
  networks:
    appnet:
      driver: bridge
```

Here we can define different services and options.  There are a lot of options, so I'll touch on a few:

* image - this is the Docker image you want to pull
* restart - we can tell Docker-Compose to restart containers unders certain conditions
* ports - open up different ports to a container
* networks - we define a virtual network for containers to connect to
* volumes - Docker volumes are persistent data stores we can use to store app data after a container ends

To run this, you simply need to save the above file as `docker-compose.yml`, cd to that directory and run

```
docker-compose up -d
```


### Conclusion ###
In this lesson you've learned how to run Docker containers in the background, allowing you to have long-running services (like a web server).  You can also use options like `--name` and `docker logs` to manage and query your containers.
