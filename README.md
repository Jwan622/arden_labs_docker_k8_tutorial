# Docker (from arden labs course)

## 1.1 The "Why" of Containers

- Since we moved to microservices and different environments (feature, prod, etc.) and not just a monolith anymore, containerization makes life easier. We have a deployment problem. Each environment has a different combination of settings and a different system to use. 
- the idea of containerization is that we put software in containers and then we ship these containers.
- Helps us escape dependency hell. Noticed best when we hire someone new to get their machine working. Other versions of mysql might conflict, different versions of java and ruby for each application. 
  - At first we write instruction files that tell the new hire how to install, then we turn it into a script, and then we turn that script into a Dockerfile. This Dockerfile will work everywhere. That's how we solve dependency hell problem.
- we can keep versions of containers so if bugs occur, we can deploy the previous version or to compare versions to figure out when the bug is introduced. We have the versions and use them easily
- we can decouple plumping with application logic. plumping = load balancing, service discovery. we can abstract that with containers
- Containers were not easy to use before Docker, that's what Docker brings to the table. APIs, shipping, Docker makes it easier.
- Images are broken down into layers that are cachable and we'll go into that later.

## 1.2 Setting up our Environment
- Let's do the Hello World of Docker.

run this in terminal:
```bash
docker run busybox echo hello world
Unable to find image 'busybox:latest' locally
latest: Pulling from library/busybox
3cb635b06aa2: Pull complete
Digest: sha256:b5cfd4befc119a590ca1a81d6bb0fa1fb19f1fbebd0397f25fae164abe1e8a6a
Status: Downloaded newer image for busybox:latest
hello world
```

the first time it fetches the busybox image and downloads it locally.

If you do it again, you already have busybox image locally.

```busybox
docker run busybox echo hello world
hello world
```

busybox is a tiny binary with commands like `echo`, `grep`, `ps`, `sed` and things on Linux systems. It's on routers, phones that run Linux

Let's see how containers are run:

```bash
root@09938340214f:/# dpkg
dpkg: error: need an action option

Type dpkg --help for help about installing and deinstalling packages [*];
Use 'apt' or 'aptitude' for user-friendly package management;
Type dpkg -Dhelp for a list of dpkg debug flag values;
Type dpkg --force-help for a list of forcing options;
Type dpkg-deb --help for help about manipulating *.deb files;

Options marked [*] produce a lot of output - pipe it through 'less' or 'more' !
root@09938340214f:/# apt-get update
Get:1 http://security.ubuntu.com/ubuntu focal-security InRelease [114 kB]
...
Get:18 http://archive.ubuntu.com/ubuntu focal-backports/main amd64 Packages [50.0 kB]
Fetched 20.1 MB in 2s (8141 kB/s)
Reading package lists... Done

root@09938340214f:/# apt-get install figlet
Reading package lists... Done
...
file /usr/share/man/man6/figlet-figlet.6.gz (of link group figlet) doesn't exist

root@09938340214f:/# figlet hello
 _          _ _
| |__   ___| | | ___
| '_ \ / _ \ | |/ _ \
| | | |  __/ | | (_) |
|_| |_|\___|_|_|\___/

root@09938340214f:/#
```

The above is how we install and run a program in the container.

to see how many packages are in the container:
```bash
dpkg -l | wc -l
98
```


not the same number of packages if you run this on host because they are 2 distinct environments. isolated environments. We ran an ubuntu environment on a Linux/windows/Macox Host and it worked. We can run any container on the host. But windows cannot run linux machines, not yet.

- When we exit a container, it's in a stopped state, but we can get back to that contianer later. Disk and compute are freed up. when the container is stopped.

If you run `docker run -ti ubuntu` and try to run figlet again after exiting the container, it's not found because the container is fresh.

because when you run `docker run -ti <image>` again the container is new. But you can spin up the old container again. We'll see later. Fresh containers are made for each invocation of `docker run`.

Virtual machines might differ over time since everyone has a different VM. Images are shared and so containers will be the same. The docker images SHOULD be shared.

Workflow with docker:
- create container image with our dev environment
- run container with that image
- work on project
- when done, shut down container
- next time we need to work on project, start new container
- if we need to tweak the environment, we create a new image.

## Running our first containers

When you run:
```bash
 docker run jpetazzo/clock
```

the clock program is in the foreground. To run it in the background:

```bash
docker run -d jpetazzo/clock
```

d stands for detached or daemonize. It returns an ID.

while it's running in the background, we can see the running container. `docker ps` shows us the running containers.

```bash
docker run -d jpetazzo/clock
6182ad2faa2773a2a2496446682650ecb0258fcf3e91c8df5937395ee6d81ea5
✔ ~/arden_labs_tutorial [main|✚ 1…7]
13:48 $ sleep 10 &
[1] 7498
✔ ~/arden_labs_tutorial [main|✚ 1…7]
13:48 $ docker ps
CONTAINER ID   IMAGE                                                                 COMMAND                  CREATED          STATUS                         PORTS                                                           NAMES
6182ad2faa27   jpetazzo/clock                                                        "/bin/sh -c 'while d…"   19 minutes ago   Up 19 minutes
```

Like the above


#### Commands

- To see last container that was started
```bash
docker ps -l
```

- To see quickly only show IDs
```bash
docker ps -q
```

- To follow to see ends of logs use `-f`, and use `--tail` shows only last log lines instead of spanning all lines.
```bash
docker logs <container_id> -f --tail 1
```

- to stop a container:
```bash
docker stop <image>
```
docker stop takes 10 seconds to stop. sends unix signal TERM to shutdown politely. Might handle it or it'll just kill it after a timeout. sends a KILL signal which on UNIX cannot be intercepted. So that's why after 10 seconds container gets killed.


- to list all containers including killed ones:
```bash
docker ps -a
```


## 1.4 Background containers

Let's see how we restart containers

no real diff between background and foreground containers.

to reattach to a container, run:

```bash
docker attach <container_id>
```

To attach to last container:
```bash
docker attach $(docker ps -ql)
```

`$(docker ps -ql)` remember gives you the id of the last container

To restart a stopped container:
```bash
15:21 $ docker attach 6182ad2faa27
You cannot attach to a stopped container, start it first
✘-1 ~/arden_labs_tutorial [main|✚ 1…7]
15:21 $ docker start 6182ad2faa27
6182ad2faa27
```

if you hit ctrl-c you stop a container. If you hit ^p^q you exit a container but you don't stop it so you can run `docker attach <container_id>` again.

## 2.1 Images, Layers & Container Images

An **image** is a bunch of files and some metadata. It's the root file system which is at `/`

What is the metadata? Some of it for aesthetic purposes (who authored the image which has no use at runtime but for humans) and the command that is run when we execute the container, env variables.

Images are made of layers, each layer can add files and metadata. We use layers to save time and bandwidth. Layers can be reused between images. A layer could be a centOS base layer. Another layer could be the code. Another layer could be the config.

Most layers are read-only, but there is a read-write layer on top. We can't change most layers, but if we need to write anything, has to be done in read-write layer, leaves other layers unchanged.

An image is a read-only file system, cannot change an image. What's a container? It's a set of processes running in  read-write copy of filesystem (namespaces and groups, part of linux.) They give processes illusions that they are running in own environment.

Even though images are read only, we make changes to the container which we then transform into a new layer and a new image is made by stacking the new layer on top of the old image.

An image can be namespaced by user like `jwan622/<image>` which is my own version of the image.
If tehre's a different host where the image is located you can do `quay.io/<user or organization>/<image>`. This is done if you want to host the image outside of Docker Hub.


Let's talk about image tags

When we run `docker run <image>` we default use the latest tag. We can specify tag by running:

```bash
docker run -ti ubuntu:12.04
```

We use tags when:
- going to production
- when recording a procedure into a script
- to ensure same version will be used everwhere

don't specify tags when you want the latest version, it's that by default.

#### Commands
This will show all images on machine:
```bash
docker images
```

To find images:
```bash
docker search zookeeper
NAME                               DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
zookeeper                          Apache ZooKeeper is an open-source server wh…   1182      [OK]
jplock/zookeeper                   Builds a docker image for Zookeeper version …   165                  [OK]
wurstmeister/zookeeper                                                             159                  [OK]
```


## 2.2 Building Images Interactively

Let's build our own images interactively
We will:
1. create a container
2. run commands to install software in the container
3. review changes with `docker diff`
4. turn the container into an image using `docker commit`
5. add tags to the image with `docker tag`

to create an image from a container, run `docker commit <container_id>`


But this is cumbersome and long and you have a build a new container each time. Let's use an automated process next.

#### Commands

- to turn container into image:
```bash
docker commit <container_id>
```
The output will be the id of the new image

- To find out what ends up in an image from a container: 
This shows us content of read-write layer and which files were changed in the container.
```bash
docker diff <container_id>
```
