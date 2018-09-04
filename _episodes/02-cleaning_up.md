## Objective ##

Learn how to remove containers and images from your machine when you no longer need them

### Cleaning Up ###
Eventually, you may want to clean out the cache of images and the history of containers, to reclaim space or just keep things tidy. Cleaning up involves two steps, removing the containers that you've run first, then removing the images themselves.

To remove the containers, including those that have exited and are still in the cache, use **docker ps --all** to get the ids, then  **docker rm** with the container ID or the container name:

```
> docker ps --all
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                        PORTS               NAMES
48a2dca14407        nginx               "nginx -g 'daemon off"   10 minutes ago      Exited (0) 4 minutes ago                          pensive_booth
1bf15ca4ba89        nginx               "nginx -g 'daemon off"   20 minutes ago      Exited (0) 16 minutes ago                         amazing_hugle
97b1e86df1f1        ubuntu              "/bin/bash"              37 minutes ago      Exited (0) 37 minutes ago                         backstabbing_brown
1688f55c3418        ubuntu              "/bin/bash"              37 minutes ago      Exited (127) 20 minutes ago                       ecstatic_hugle
c69d6f8d89bd        ubuntu              "/bin/bash"              41 minutes ago      Exited (0) 41 minutes ago                         sad_keller
960588723c36        ubuntu              "/bin/echo 'hello wor"   45 minutes ago      Exited (0) 45 minutes ago                         suspicious_mestorf
ffbb0f60bda6        ubuntu              "/bin/echo 'hello wor"   51 minutes ago      Exited (0) 51 minutes ago                         mad_booth

> docker rm ffbb0f60bda6 960588723c36 # cleaning by ID
ffbb0f60bda6
960588723c36

> docker rm sad_keller ecstatic_hugle # cleaning by name
sad_keller
ecstatic_hugle

> docker ps --all
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                      PORTS               NAMES
48a2dca14407        nginx               "nginx -g 'daemon off"   23 minutes ago      Exited (0) 17 minutes ago                       pensive_booth
1bf15ca4ba89        nginx               "nginx -g 'daemon off"   33 minutes ago      Exited (0) 29 minutes ago                       amazing_hugle
97b1e86df1f1        ubuntu              "/bin/bash"              50 minutes ago      Exited (0) 50 minutes ago                       backstabbing_brown
```

You can concoct your own one-liner to clean up everything, if you wish. Docker will refuse to delete something that's actively in use, so you won't screw things up too badly this way

```
> docker rm `docker ps --all -q`
48a2dca14407
1bf15ca4ba89
97b1e86df1f1
```

Now that we've removed the containers, let's clean up the images they came from. This uses the **docker images** and **docker rmi** commands, in a similar manner:

```
> docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
ubuntu              latest              4ca3a192ff2a        22 hours ago        128.2 MB
nginx               latest              abf312888d13        2 days ago          181.5 MB

> docker rmi ubuntu nginx
Untagged: ubuntu:latest
Untagged: ubuntu@sha256:3b64c309deae7ab0f7dbdd42b6b326261ccd6261da5d88396439353162703fb5
Deleted: sha256:4ca3a192ff2a5b7e225e81dc006b6379c10776ed3619757a65608cb72de0a7f6
Deleted: sha256:2c2e0ef08d4988122fddadfe7e84d7b0aae246e6fa805bb6380892a325bc5216
Deleted: sha256:918fe8ae141d41d7430eb887e57a21d76fb0181317ec4a68f7abbd17caab034a
Deleted: sha256:00f434fa2fa1a0558f8740af28aef3d6ee546a8758c0a37dddee3f65b5290e7a
Deleted: sha256:f9545ee77b4b95c887dbebc6d08325be354274423112b0b66f7288a2cf7905cb
Deleted: sha256:d29d52f94ad5aa750bd76d24effaf6aeec487d530e262597763e56065a06ee67
Untagged: nginx:latest
Untagged: nginx@sha256:3861a20a81e4ba699859fe0724dc6afb2ce82d21cd1ddc27fff6ec76e4c2824e
Deleted: sha256:abf312888d132e461c61484457ee9fd0125d666672e22f972f3b8c9a0ed3f0a1
Deleted: sha256:21976f5d6f037350c076d6974b2ac97777c962869d5511855170dc5c926d5aac
Deleted: sha256:f9719c3716279b47727a51595d1482506be469c581383cb529f4005ba9bf2aeb
Deleted: sha256:fe4c16cbf7a4c70a5462654cf2c8f9f69778db280f235229bd98cf8784e878e4
```

You need to be sure to stop a container before removing its image.  If not, you'll see an error about child images:

```
> docker ps

CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                  NAMES
bf0a1726909a        nginx               "nginx -g 'daemon of…"   18 seconds ago      Up 16 seconds       0.0.0.0:8080->80/tcp   eloquent_raman

> docker rmi nginx
Error response from daemon: conflict: unable to remove repository reference "nginx" (must force) - container a4dbdb7eb4b1 is using its referenced image 71c43202b8ac
```

### Conclusion ###
Cleaning up containers and images is a two-step process. Now you should be able to keep your system tidy.

### Best practices ###

- use the ```--rm``` flag when you know you won't want to re-start a container
- if you use containers heavily, clean up the images from time to time
