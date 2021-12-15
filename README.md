# Docker

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

To see last container that was started
```bash
docker ps -l
```

To see quickly only show IDs
```bash
docker ps -q
```

To follow to see ends of logs use `-f`, and use `--tail` shows only last log lines instead of spanning all lines.
```bash
docker logs <container_id> -f --tail 1
```

to stop:
```bash
docker stop <image>
```
docker stop takes 10 seconds to stop. sends unix signal TERM to shutdown politely. Might handle it or it'll just kill it after a timeout. sends a KILL signal which on UNIX cannot be intercepted. So that's why after 10 seconds container gets killed.


to list all containers including killed ones:
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

To restart a stopped container:
```bash
15:21 $ docker attach 6182ad2faa27
You cannot attach to a stopped container, start it first
✘-1 ~/arden_labs_tutorial [main|✚ 1…7]
15:21 $ docker start 6182ad2faa27
6182ad2faa27
```

if you hit ctrl-c you stop a container
