# Docker: From Hello World to Swarm

You can follow this tutorial to learn the basics of Docker, from running a hello world container to
running a multi-machine swarm. It's a condensed version of some of Docker's own documentation and
other articles (see the [references](#references) at the bottom).

Note: some names and IDs shown in sample command outputs are randomly generated or
context-dependent, and will differ on your machine. Pay attention to the instructions and do not
blindly copy-paste commands.

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->


- [Requirements](#requirements)
- [What is Docker?](#what-is-docker)
  - [What is a container?](#what-is-a-container)
- [Containers & images](#containers--images)
  - [Make sure Docker is working](#make-sure-docker-is-working)
  - [Run a container from an image](#run-a-container-from-an-image)
  - [Container isolation](#container-isolation)
  - [Run multiple commands in a container](#run-multiple-commands-in-a-container)
  - [Commit a container's state to an image manually](#commit-a-containers-state-to-an-image-manually)
  - [Run containers in the background](#run-containers-in-the-background)
  - [Access container logs](#access-container-logs)
  - [Stop and restart containers](#stop-and-restart-containers)
  - [Run multiple containers](#run-multiple-containers)
  - [Image layers](#image-layers)
    - [The top writable layer of containers](#the-top-writable-layer-of-containers)
    - [Total image size](#total-image-size)
- [Dockerfile](#dockerfile)
  - [The `docker build` command](#the-docker-build-command)
  - [Format](#format)
  - [Build an image from a Dockerfile](#build-an-image-from-a-dockerfile)
  - [Build cache](#build-cache)
  - [A Dockerfile for a Node.js application](#a-dockerfile-for-a-nodejs-application)
- [Connecting containers](#connecting-containers)
  - [Exposing container ports on the host machine](#exposing-container-ports-on-the-host-machine)
  - [Docker networks](#docker-networks)
    - [Running a container in a network](#running-a-container-in-a-network)
- [Persistent storage](#persistent-storage)
  - [Bind mounts](#bind-mounts)
  - [Volumes](#volumes)
- [Debugging containers](#debugging-containers)
  - [Ephemeral containers](#ephemeral-containers)
- [Docker Compose](#docker-compose)
  - [The `docker-compose.yml` file](#the-docker-composeyml-file)
  - [Running Docker Compose services](#running-docker-compose-services)
  - [Rebuilding Docker Compose services](#rebuilding-docker-compose-services)
  - [Starting containers automatically](#starting-containers-automatically)
  - [Horizontal scaling](#horizontal-scaling)
- [Docker Swarm](#docker-swarm)
  - [Setup](#setup)
  - [Create a swarm](#create-a-swarm)
  - [Run a private registry](#run-a-private-registry)
    - [Push an image to a private registry](#push-an-image-to-a-private-registry)
  - [Service stacks](#service-stacks)
  - [Service replication & placement](#service-replication--placement)
  - [Overlay network](#overlay-network)
  - [Deploy a service stack to a swarm](#deploy-a-service-stack-to-a-swarm)
  - [Swarm mode routing mesh](#swarm-mode-routing-mesh)
- [Appendices](#appendices)
  - [Squashing image layers](#squashing-image-layers)
    - [Using the `--squash` option](#using-the---squash-option)
  - [Dockerfile tips](#dockerfile-tips)
    - [Using smaller base images](#using-smaller-base-images)
    - [Labeling images](#labeling-images)
    - [Environment variables](#environment-variables)
    - [Non-root users](#non-root-users)
    - [Speeding up builds](#speeding-up-builds)
    - [Documenting exposed ports](#documenting-exposed-ports)
    - [Using an entrypoint script](#using-an-entrypoint-script)
      - [Waiting for other containers](#waiting-for-other-containers)
  - [Multi-stage builds](#multi-stage-builds)
- [TODO](#todo)
- [References](#references)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

<br><br><br><br><br>





## Requirements

* [Docker Community Edition (CE)][docker-ce] (the latest version is `18.03.0-ce` at the time of
  writing).
* [Docker Compose][docker-compose-install] (installed with Docker by default if you're using Docker
  for Mac or Docker for Windows; the latest version is `1.21.0` at the time of writing).
* A UNIX command line (on Windows, use [Git Bash][git-bash] or the [Windows Subsystem for
  Linux][wsl]).

Optionally, to quickly set up the infrastructure for the [Docker Swarm](#docker-swarm) chapter, you
can use:

* [Vagrant][vagrant] 2+
* [VirtualBox][virtualbox] 5+

[Back to top](#readme)

---

<br><br><br><br><br>





## What is Docker?

[Docker][what-is-docker] is the company driving the **container movement**.  Today's businesses are
under pressure to digitally transform but are constrained by existing applications and
infrastructure while rationalizing an **increasingly diverse portfolio of clouds, datacenters and
application architectures**.  Docker enables true **independence between applications and
infrastructure** and developers and IT ops to unlock their potential and creates a model for better
collaboration and innovation.

[Back to top](#readme)

<br>

### What is a container?

A [container][what-is-container] image is a **lightweight, stand-alone, executable package of a
piece of software** that includes everything needed to run it: code, runtime, system tools, system
libraries, settings.  Available for both Linux and Windows based apps, containerized software will
always run the same, regardless of the environment. **Containers isolate software from its
surroundings**, for example differences between development and staging environments and help reduce
conflicts between teams running different software on the same infrastructure.

![Docker containers](images/containers.png)

**Containers and virtual machines are similar** when it comes to resource isolation and allocation
benefits, **but function differently** because containers virtualize the operating system instead of
hardware.  Containers are more portable and efficient.

Virtual machines (VMs) are an abstraction of physical hardware turning one server into many servers.
The hypervisor allows multiple VMs to run on a single machine. Each VM includes a full copy of an
operating system, one or more apps, necessary binaries and libraries - taking up tens of GBs. VMs
can also be slow to boot.

![Docker virtual machine versus container](images/vm-vs-container.png)

Containers are an abstraction at the app layer that packages code and dependencies together.
Multiple containers can run on the same machine and share the OS kernel with other containers, each
running as isolated processes in user space. Containers take up less space than VMs (container
images are typically tens of MBs in size), and start almost instantly.

![Docker container versus virtual machine](images/container-vs-vm.png)

In a nutshell, containers are:

* **Lightweight** - Docker containers running on a single machine **share that machine's operating
  system kernel**; they start instantly and use less CPU and RAM. Images are constructed from file
  system layers and share common files. This minimizes disk usage and image downloads are much
  faster.
* **Standard** - Docker containers are based on [**open standards**][open-container-initiative] and
  run on all major Linux distributions, Microsoft Windows, and on any infrastructure including VMs,
  bare-metal and in the cloud.
* **Secure** - Docker containers **isolate applications** from one another and from the underlying
  infrastructure. Docker provides the strongest default isolation to limit app issues to a single
  container instead of the entire machine.

[Back to top](#readme)

---

<br><br><br><br><br>





## Containers & images

<br>

### Make sure Docker is working

Run a `hello-world` container to make sure everything is installed correctly:

```bash
$> docker run hello-world

Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
9bb5a5d4561a: Pull complete
Digest: sha256:f5233545e43561214ca4891fd1157e1c3c563316ed8e237750d59bde73361e77
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/engine/userguide/
```

If your output is similar, you can move on to the next section.

[Back to top](#readme)

<br>

### Run a container from an image

There are many official and community images available on the [Docker Hub][hub-explore]. For this
tutorial, we'll start by pulling the [official `ubuntu` image][hub-ubuntu] from the hub:

```bash
$> docker pull ubuntu
Using default tag: latest
latest: Pulling from library/ubuntu
d3938036b19c: Pull complete
a9b30c108bda: Pull complete
67de21feec18: Pull complete
817da545be2b: Pull complete
d967c497ce23: Pull complete
Digest: sha256:9ee3b83bcaa383e5e3b657f042f4034c92cdd50c03f73166c145c9ceaea9ba7c
Status: Downloaded newer image for ubuntu:latest
```

An **image** is a **blueprint** which forms the basis of containers. It's basically a snapshot of a
file system in a given state. This `ubuntu` image contains a headless [Ubuntu operating
system][ubuntu] with only minimal packages installed.

You can list available images with `docker images`:

```bash
$> docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
ubuntu              latest              c9d990395902        7 days ago          113MB
hello-world         latest              e38bc07ac18e        8 days ago          1.85kB
```

Run a **container** based on that image with `docker run <image> [command...]`.  The following
command runs an Ubuntu container:

```bash
$> docker run ubuntu echo "hello from ubuntu"
hello from ubuntu
```

Running a container means **executing the specified command**, in this case `echo "hello from
ubuntu"`, **in an isolated container started from an image**, in this case the Ubuntu image.  The
`echo` binary that is executed is the one provided by the Ubuntu OS in the image, not your machine.

If you list running containers with `docker ps`, you will see that the container we just ran is
**not running**.  A container **stops as soon as the process started by its command is done**.
Since `echo` is not a long-running command, the container stopped right away:

```bash
$> docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
```

You can see the stopped container with `docker ps -a`, which lists all containers regardless of their status:

```bash
$> docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                      PORTS               NAMES
d042ed58ef63        ubuntu              "echo 'hello from ub…"   10 seconds ago      Exited (0) 11 seconds ago                       relaxed_galileo
02bbe5e66c15        hello-world         "/hello"                 42 seconds ago      Exited (0) 42 seconds ago                       jovial_jones
```

You can remove a stopped container or containers with `docker rm`, using either its ID or its name:

```bash
$> docker rm relaxed_galileo 02bbe5e66c15
relaxed_galileo
02bbe5e66c15

$> docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
```

You can also add the `--rm` option to `docker run` to run a container and automatically remove it
when it stops:

```bash
$> docker run --rm ubuntu echo "hello from ubuntu"
hello from ubuntu
```

No new container should appear in the list:

```bash
$> docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                      PORTS               NAMES
```

[Back to top](#readme)

<br>

### Container isolation

Docker containers are very similar to [LXC containers][lxc] which provide many
[security][docker-security] features.  When you start a container with `docker run`, you get:

* **Process isolation:** processes running within a container cannot see, and even less
  affect, processes running in another container, or in the host system.

  See the difference between running `ps -e` and `docker run --rm ubuntu ps -e`, which will show you
  all running processes on your machine, and the same as seen from within a container, respectively.
* **File system isolation:** a container has its own file system separate from your machine's. See
  the difference between running the following commands:

  * `ls -la /` and `docker run --rm ubuntu ls -la /`, which will show you all files at the root of
    your file system, and all files at the root of the container's file system, respectively.
  * `bash --version` and `docker run --rm ubuntu bash --version`, which will show you that the Bash
    shell on your machine is (probably) not the exact same version as the one in the image.
  * `uname -a` and `docker run --rm ubuntu uname -a`, which will show you your machine's operating
    system and the container's, respectively.
* **Network isolation:** a container doesn't get privileged access to the sockets or interfaces of
  another container. Of course, containers can interact with each other through their respective
  network interface, just like they can interact with external hosts. We will see examples of this
  later.

[Back to top](#readme)

<br>

### Run multiple commands in a container

You can run commands more complicated than `echo`.
For example, let's run a [Bash shell][bash].

Since this is an interactive command, add the `-i` (interactive) and `-t` (pseudo-TTY) options to `docker run`:

```bash
$> docker run -it ubuntu bash
root@e07f81d7941d:/#
```

This time you are running a Bash shell, which is a **long running comman**d. The process running the
shell will not stop until you manually type `exit` in the shell, so the **container is not
stopping** either.

You should have a new command line prompt (`root@e07f81d7941d:/#` in this example), indicating that you are
within the container:

```bash
root@e07f81d7941d:/#
```

You can now run any command you want within the running container:

```bash
root@e07f81d7941d:/# date
Fri Apr 20 13:20:32 UTC 2018
```

You can make changes to the container. Since this is an Ubuntu container, you can install packages.
Update of the package lists first with `apt-get update`:

```bash
root@e07f81d7941d:/# apt-get update
Get:1 http://archive.ubuntu.com/ubuntu xenial InRelease [247 kB]
...
Fetched 25.1 MB in 1s (12.9 MB/s)
Reading package lists... Done
```

Install the `fortune` package:

```bash
root@e07f81d7941d:/# apt-get install -y fortune
Reading package lists... Done
Building dependency tree
Reading state information... Done
Note, selecting 'fortune-mod' instead of 'fortune'
The following additional packages will be installed:
  fortunes-min librecode0
Suggested packages:
  fortunes x11-utils bsdmainutils
The following NEW packages will be installed:
  fortune-mod fortunes-min librecode0
0 upgraded, 3 newly installed, 0 to remove and 5 not upgraded.
Need to get 625 kB of archives.
After this operation, 2184 kB of additional disk space will be used.
Get:1 http://archive.ubuntu.com/ubuntu xenial/main amd64 librecode0 amd64 3.6-22 [523 kB]
...
Fetched 625 kB in 0s (3250 kB/s)
...
```

The [`fortune`][fortune] command prints a quotation/joke such as the ones found in fortune cookies
(hence the name):

```bash
root@e07f81d7941d:/# /usr/games/fortune
Your motives for doing whatever good deed you may have in mind will be
misinterpreted by somebody.
```

Let's create a **fortune clock script** that tells the time and a fortune every 5 seconds.

The `ubuntu` container image is very minimal, as most images are, and doesn't provide any editor
such as `nano` or `vim`. You could install it as well, but for the purposes of this tutorial, we'll
create the script with the basic commands available to us in the container.

Run the following multiline command to save a bash script to `/usr/local/bin/clock.sh` (copy-paste
the entire 10 lines of the command, starting with `cat` and ending with the second `EOF` 9 lines
below, then press enter to execute the whole thing):

```bash
root@e07f81d7941d:/# cat << EOF > /usr/local/bin/clock.sh
#!/bin/bash
trap "exit" SIGKILL SIGTERM SIGHUP SIGINT EXIT
while true; do
  echo It is \$(date)
  /usr/games/fortune
  echo
  sleep 5
done
EOF
```

Make the script executable:

```bash
root@e07f81d7941d:/# chmod +x /usr/local/bin/clock.sh
```

Make sure it works. Since the `/usr/local/bin` directory is in the PATH by default on Linux, you can
simply execute `clock.sh` without using its absolute path:

```bash
root@e07f81d7941d:/# clock.sh
It is Mon Apr 23 08:47:37 UTC 2018
You have no real enemies.

It is Mon Apr 23 08:47:42 UTC 2018
Beware of a dark-haired man with a loud tie.

It is Mon Apr 23 08:47:47 UTC 2018
If you sow your wild oats, hope for a crop failure.
```

Use Ctrl-C to stop the clock script. Then use `exit` to stop the Bash shell:

```bash
root@e07f81d7941d:/# exit
```

Since the Bash process has exited, the container has stopped as well:

```bash
$> docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                       PORTS               NAMES
f6b9fa680789        ubuntu              "bash"              13 minutes ago      Exited (130) 4 seconds ago                       goofy_shirley
```

[Back to top](#readme)

<br>

### Commit a container's state to an image manually

Retrieve the name or ID of the previous container, in this case `goofy_shirley`.  You can **create a
new image based on that container's state** with the `docker commit <container> <repository:tag>`
command:

```bash
$> docker commit goofy_shirley fortune-clock:1.0
sha256:407daed1a864b14a4ab071f274d3058591d2b94f061006e88b7fc821baf8232e
```

You can see the new image in the list of images:

```bash
$> docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
fortune-clock       1.0                 407daed1a864        9 seconds ago       156MB
ubuntu              latest              c9d990395902        10 days ago         113MB
hello-world         latest              e38bc07ac18e        11 days ago         1.85kB
```

That image contains the `/usr/local/bin/clock.sh` script we created, so we can run it directly with
`docker run <image> [command...]`:

```bash
$> docker run --rm fortune-clock:1.0 clock.sh
It is Mon Apr 23 08:55:54 UTC 2018
You will have good luck and overcome many hardships.

It is Mon Apr 23 08:55:59 UTC 2018
While you recently had your problems on the run, they've regrouped and
are making another attack.
```

Again, our `clock.sh` script is a long-running command (due to the `while` loop). The container will
keep running until the script is stopped.  Use Ctrl-C to stop it (the container will stop and be
removed automatically thanks to the `--rm` option).

That's nice, but let's create a fancier version of our clock. Run a new Bash shell based on our
`fortune-clock:1.0` image:

```bash
$> docker run -it fortune-clock:1.0 bash
root@4b38e523336c:/#
```

This new container is **based off of our `fortune-clock:1.0` image**, so it already contains what
we've done done so far, i.e. the `fortune` command is already installed and the `clock.sh` script is
where we put it:

```bash
root@4b38e523336c:/# /usr/games/fortune
You're working under a slight handicap.  You happen to be human.

root@4b38e523336c:/# cat /usr/local/bin/clock.sh
#!/bin/bash
trap "exit" SIGKILL SIGTERM SIGHUP SIGINT EXIT
while true; do
  echo It is $(date)
  /usr/games/fortune
  echo
  sleep 5
done
```

Install the `cowsay` package:

```bash
root@4b38e523336c:/# apt-get install -y cowsay
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following additional packages will be installed:
  cowsay-off ifupdown iproute2 isc-dhcp-client isc-dhcp-common libatm1 libdns-export162 libgdbm3 libisc-export160 libmnl0 libperl5.22 libtext-charwidth-perl libxtables11 netbase perl perl-base
  perl-modules-5.22 rename
Suggested packages:
  filters ppp rdnssd iproute2-doc resolvconf avahi-autoipd isc-dhcp-client-ddns apparmor perl-doc libterm-readline-gnu-perl | libterm-readline-perl-perl make
The following NEW packages will be installed:
  cowsay cowsay-off ifupdown iproute2 isc-dhcp-client isc-dhcp-common libatm1 libdns-export162 libgdbm3 libisc-export160 libmnl0 libperl5.22 libtext-charwidth-perl libxtables11 netbase perl perl-modules-5.22
  rename
The following packages will be upgraded:
  perl-base
1 upgraded, 18 newly installed, 0 to remove and 4 not upgraded.
Need to get 9432 kB of archives.
After this operation, 44.8 MB of additional disk space will be used.
Get:1 http://archive.ubuntu.com/ubuntu xenial-updates/main amd64 perl-base amd64 5.22.1-9ubuntu0.3 [1286 kB]
...
Fetched 9432 kB in 0s (17.5 MB/s)
...
```

Overwrite the clock script with this multiline command. The output of the `fortune` command is now
piped into the `cowsay` command:

```bash
root@4b38e523336c:/# cat << EOF > /usr/local/bin/clock.sh
#!/bin/bash
trap "exit" SIGKILL SIGTERM SIGHUP SIGINT EXIT
while true; do
  echo It is \$(date)
  /usr/games/fortune | /usr/games/cowsay
  echo
  sleep 5
done
EOF
```

Test our improved clock script:

```bash
root@4b38e523336c:/# clock.sh
It is Mon Apr 23 09:02:21 UTC 2018
 ____________________________________
/ Look afar and see the end from the \
\ beginning.                         /
 ------------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||

It is Mon Apr 23 09:02:26 UTC 2018
 _______________________________________
/ One of the most striking differences  \
| between a cat and a lie is that a cat |
| has only nine lives.                  |
|                                       |
| -- Mark Twain, "Pudd'nhead Wilson's   |
\ Calendar"                             /
 ---------------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||

It is Tue Jun 14 21:58:18 UTC 2018
 _______________________________________
/ I broke a chair ?! Did I just broke a \
| chair ?!                              |
|                                       |
\ -- Anonymous iranian girl in Sweden   /                                      
 ---------------------------------------
        \   ^__^
         \  (^^)\_______
            (__)\       )\/\
                ||----w |
                ||     ||  
```

Much better. Exit Bash to stop the container:

```bash
root@4b38e523336c:/# exit
```

You should now have two stopped containers. The one in which we created the original clock script, and
the newest one we just stopped:

```bash
$> docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                        PORTS               NAMES
4b38e523336c        fortune-clock:1.0   "bash"              4 minutes ago       Exited (130) 21 seconds ago                       peaceful_turing
f6b9fa680789        ubuntu              "bash"              27 minutes ago      Exited (130) 13 minutes ago                       goofy_shirley
```

Let's create an image from that latest container, in this case `peaceful_turing`:

```bash
$> docker commit peaceful_turing fortune-clock:2.0
sha256:92bfbc9e4c4c68a8427a9c00f26aadb6f7112b41db19a53d4b29d1d6f68de25f
```

As before, the image is available in the list of images:

```bash
$> docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
fortune-clock       2.0                 92bfbc9e4c4c        24 seconds ago      205MB
fortune-clock       1.0                 407daed1a864        11 minutes ago      156MB
ubuntu              latest              c9d990395902        10 days ago         113MB
hello-world         latest              e38bc07ac18e        11 days ago         1.85kB
```

You can run it:

```bash
$> docker run --rm fortune-clock:2.0 clock.sh
It is Mon Apr 23 09:06:21 UTC 2018
 ________________________________________
/ Living your life is a task so          \
| difficult, it has never been attempted |
\ before.                                /
 ----------------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```

Use Ctrl-C to stop the script (and the container).

Your **previous image is still available** under the `1.0` tag.  You can run it again:

```bash
$> docker run --rm fortune-clock:1.0 clock.sh
It is Mon Apr 23 09:08:04 UTC 2018
You attempt things that you do not even plan because of your extreme stupidity.
```

Use Ctrl-C to stop the script (and the container).

[Back to top](#readme)

<br>

### Run containers in the background

Until now we've only run a container command **in the foreground**, meaning that Docker takes
control of our console and forwards the script's output to it.

You can run a container command **in the background** by adding the `-d` or `--detach` option.
Let's also use the `--name` option to give it a specific name instead of using the default randomly
generated one:

```bash
$> docker run -d --name clock fortune-clock:2.0 clock.sh
06eb72c218051c77148a95268a2be45a57379c330ac75a7260c16f89040279e6
```

This time, the `docker run` command simply prints the ID of the container it has launched, and exits
immediately.  But you can see that the container is indeed running with `docker ps`, and that it has
the correct name:

```bash
$> docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
06eb72c21805        fortune-clock:2.0   "clock.sh"          6 seconds ago       Up 9 seconds                            clock
```

[Back to top](#readme)

<br>

### Access container logs

You can use the `docker logs <container>` command to see the **output of a container running in the
background**:

```bash
$> docker logs clock
It is Mon Apr 23 09:12:06 UTC 2018
 _____________________________________
< Excellent day to have a rotten day. >
 -------------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
...
```

Add the `-f` option to keep following the log **output in real time**:

```bash
$> docker logs -f clock
It is Mon Apr 23 09:13:36 UTC 2018
 _________________________________________
/ I have never let my schooling interfere \
| with my education.                      |
|                                         |
\ -- Mark Twain                           /
 -----------------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```

Use Ctrl-C to stop following the logs.

[Back to top](#readme)

<br>

### Stop and restart containers

You may **stop a container running in the background** with the `docker stop <container>` command:

```bash
$> docker stop clock
clock
```

You can check that is has indeed stopped:

```bash
$> docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                        PORTS               NAMES
06eb72c21805        fortune-clock:2.0   "clock.sh"          2 minutes ago       Exited (0) 1 second ago                           clock
4b38e523336c        fortune-clock:1.0   "bash"              17 minutes ago      Exited (130) 13 minutes ago                       peaceful_turing
f6b9fa680789        ubuntu              "bash"              40 minutes ago      Exited (130) 27 minutes ago                       goofy_shirley
```

You can **restart** it with the `docker start <container>` command. This will **re-execute the
command** that was originally given to `docker run <container> [command...]`, in this case
`clock.sh`:

```bash
$> docker start clock
clock
```

It's running again:

```bash
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
06eb72c21805        fortune-clock:2.0   "clock.sh"          3 minutes ago       Up 1 second                             clock
```

You can follow its logs again:

```bash
$> docker logs -f clock
It is Mon Apr 23 09:14:50 UTC 2018
 _________________________________
< So you're back... about time... >
 ---------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```

Stop following the logs with Ctrl-C.

You can **stop and remove** a container in one command by adding the `-f` or `--force` option to
`docker rm`. Beware that it will *not ask for confirmation*:

```bash
$> docker rm -f clock
clock
```

No containers should be running any more:

```bash
$> docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
```

[Back to top](#readme)

<br>

### Run multiple containers

Since containers have isolated processes, networks and file systems, you can of course run more than
one at the same time:

```bash
$> docker run -d --name old-clock fortune-clock:1.0 clock.sh
25c9016ce01f93c3e073b568e256ae7f70223f6abd47bb6f4b31606e16a9c11e

$> docker run -d --name new-clock fortune-clock:2.0 clock.sh
4e367ffdda9829482734038d3eb71136d38320b6171dda31a5b287a66ee4b023
```

You can see that both are indeed running:

```bash
$> docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
4e367ffdda98        fortune-clock:2.0   "clock.sh"          20 seconds ago      Up 24 seconds                           new-clock
25c9016ce01f        fortune-clock:1.0   "clock.sh"          23 seconds ago      Up 27 seconds                           old-clock
```

Each container is running based on the correct image, as you can see by their output:

```bash
$> docker logs old-clock
It is Mon Apr 23 09:39:18 UTC 2018
Too much is just enough.
                -- Mark Twain, on whiskey
...

$> docker logs new-clock
It is Mon Apr 23 09:40:36 UTC 2018
 ____________________________________
/ You have many friends and very few \
\ living enemies.                    /
 ------------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
...
```

[Back to top](#readme)

<br>

### Image layers

A Docker **image** is built up from a series of **layers**. Each layer contains a set of differences
from the layer before it:

![Docker: Image Layers](images/layers.jpg)

You can list those layers by using the `docker inspect` command with an image name or ID. Let's see
what layers the `ubuntu` image has:

```bash
$> docker inspect ubuntu
...
        "RootFS": {
            "Type": "layers",
            "Layers": [
                "sha256:fccbfa2912f0cd6b9d13f91f288f112a2b825f3f758a4443aacb45bfc108cc74",
                "sha256:e1a9a6284d0d24d8194ac84b372619e75cd35a46866b74925b7274c7056561e4",
                "sha256:ac7299292f8b2f710d3b911c6a4e02ae8f06792e39822e097f9c4e9c2672b32d",
                "sha256:a5e66470b2812e91798db36eb103c1f1e135bbe167e4b2ad5ba425b8db98ee8d",
                "sha256:a8de0e025d94b33db3542e1e8ce58829144b30c6cd1fff057eec55b1491933c3"
            ]
        }
...
```

Each layer is identified by a hash based on previous layer's hash and the state when the layer was
created. This is similar to commit hashes in a Git repository.

Let's check the layers of our first `fortune-clock:1.0` image:

```bash
$> docker inspect fortune-clock:1.0
...
        "RootFS": {
            "Type": "layers",
            "Layers": [
                "sha256:fccbfa2912f0cd6b9d13f91f288f112a2b825f3f758a4443aacb45bfc108cc74",
                "sha256:e1a9a6284d0d24d8194ac84b372619e75cd35a46866b74925b7274c7056561e4",
                "sha256:ac7299292f8b2f710d3b911c6a4e02ae8f06792e39822e097f9c4e9c2672b32d",
                "sha256:a5e66470b2812e91798db36eb103c1f1e135bbe167e4b2ad5ba425b8db98ee8d",
                "sha256:a8de0e025d94b33db3542e1e8ce58829144b30c6cd1fff057eec55b1491933c3",
                "sha256:3539c048e45e973cf477148af3c9c91885950e77d10e77e1db1097bd20b16129"
            ]
        },
...
```

Note that the layers are the same as the `ubuntu` image, with an additional one at the end (starting
with `3539c048e`). This additional layer contains the changes we made compared to the original
`ubuntu` image, i.e.:

* Update the package lists with `apt-get update`
* Install the `fortune` package with `apt-get install`
* Create the `/usr/local/bin/clock.sh` script

The new hash (starting with `3539c048e`) is based both on these changes and the previous hash
(starting with `a8de0e025`), and it **uniquely identifies this layer**.

Take a look at the layers of our second `fortune-clock:2.0` image:

```bash
$> docker inspect fortune-clock:2.0
...
        "RootFS": {
            "Type": "layers",
            "Layers": [
                "sha256:fccbfa2912f0cd6b9d13f91f288f112a2b825f3f758a4443aacb45bfc108cc74",
                "sha256:e1a9a6284d0d24d8194ac84b372619e75cd35a46866b74925b7274c7056561e4",
                "sha256:ac7299292f8b2f710d3b911c6a4e02ae8f06792e39822e097f9c4e9c2672b32d",
                "sha256:a5e66470b2812e91798db36eb103c1f1e135bbe167e4b2ad5ba425b8db98ee8d",
                "sha256:a8de0e025d94b33db3542e1e8ce58829144b30c6cd1fff057eec55b1491933c3",
                "sha256:3539c048e45e973cf477148af3c9c91885950e77d10e77e1db1097bd20b16129",
                "sha256:5dbae8ae43cba9e34980fd6076156503814c916453a5582646a6a34c02c68546"
            ]
        },
...
```

Again, we see the same layers, including the `3539c048e` layer from the `fortune-clock:1.0` image,
and an additional layer (starting with `5dbae8ae4`). This layer contains the following changes we
made based on the `fortune-clock:1.0` image:

* Install the `cowsay` package with `apt-get install`
* Overwrite the `/usr/local/bin/clock.sh` script

[Back to top](#readme)

<br>

#### The top writable layer of containers

When you create a new container, you add a new **writable layer** on top of the image's underlying
layers. All changes made to the running container (i.e. creating, modifying, deleting files) are
written to this thin writable container layer.  When the container is deleted, the writable layer is
also deleted, unless it was committed to an image.

The layers belonging to the image used as a base for your container are never modified–they are
**read-only**. Docker uses a [union file system][union-fs] and a [copy-on-write strategy][cow] to
make it work:

* When you read a file, the union file system will look in all layers, from newest to oldest, and
  return the first version it finds.
* When you write to a file, the union file system will look for an older version, copy it to the top
  writable layer, and modify that copied version. Previous version(s) of the file in older layers
  still exist, but are "hidden" by the file system; only the most recent version is seen.

Multiple containers can therefore use the same read-only image layers, as they only modify their own
writable top layer:

![Docker: Sharing Layers](images/sharing-layers.jpg)

[Back to top](#readme)

<br>

#### Total image size

What we've just learned about layers has several implications:

* You **cannot delete files from previous layers to reduce total image size**. Assume that an
  image's last layer contains a 1GB file. Creating a new container from that image, deleting that
  file, and saving that state as a new image will not reclaim that gigabyte. The total image size
  will still be the same, as the file is still present in the previous layer.

  This is also similar to a Git repository, where committing a file deletion does not reclaim its
  space from the repository's object database (as the file is still referenced by previous commits
  in the history).
* Since layers are read-only and incrementally built based on previous layers, **the size of common
  layers shared by several images is only taken up once**.

  If you take a look at the output of `docker images`, naively adding the displayed sizes adds up to
  447MB, but that is **not** the size that is actually occupied by these images on your file system.

  ```bash
  $> docker images
  REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
  fortune-clock       2.0                 92bfbc9e4c4c        2 hours ago         205MB
  fortune-clock       1.0                 407daed1a864        2 hours ago         156MB
  ubuntu              latest              c9d990395902        10 days ago         113MB
  hello-world         latest              e38bc07ac18e        11 days ago         1.85kB
  ```

  Let's add the `-s` or `--size` option to `docker ps` to display the size of our containers' file
  systems:

  ```bash
  $> docker ps -as
  CONTAINER ID   IMAGE               COMMAND      CREATED             STATUS                     PORTS   NAMES                    SIZE
  4e367ffdda98   fortune-clock:2.0   "clock.sh"   About an hour ago   Up About an hour                   new-clock                0B (virtual 205MB)
  25c9016ce01f   fortune-clock:1.0   "clock.sh"   About an hour ago   Up About an hour                   old-clock                0B (virtual 156MB)
  4b38e523336c   fortune-clock:1.0   "bash"       2 hours ago         Exited (130) 2 hours ago           peaceful_turing          48.7MB (virtual 205MB)
  f6b9fa680789   ubuntu              "bash"       2 hours ago         Exited (130) 2 hours ago           goofy_shirley            43MB (virtual 156MB)
  ```

  The `SIZE` column shows the size of the top writable container layer, and the total virtual size
  of all the layers (including the top one) in parentheses.  If you look at the virual sizes, you
  can see that:

  * The virtual size of the `goofy_shirley` container is 156MB, which corresponds to the size of the
    `fortune-clock:1.0` image, since we committed that image based on that container's state.
  * Similarly, the virtual size of the `peaceful_turing` container is 205MB, which corresponds to
    the size of the `fortune-clock:2.0` image.
  * The `old-clock` and `new-clock` containers also have the same virtual sizes since they are based
    on the same images.

  Taking a look at the sizes of the top writable container layers, we can see that:

  * The size of the `goofy_shirley` container's top layer is 43MB. This corresponds to the space
    taken up by the package lists, the `fortune` package and its dependencies, and the `clock.sh`
    script.

    The virtual size of 156MB corresponds to the 113MB of the `ubuntu` base image, plus the 43MB of
    the top layer. As we've seen above, this is also the size of the `fortune-clock:1.0` image.
  * The size of the `peaceful_turing` container's top layer is 48.7MB. This corresponds to the space
    taken up by the `cowsay` package and its dependencies, and the new version of the `clock.sh`
    script.

    The virtual size of 205MB corresponds to the 156MB of the `fortune-clock:1.0` base image, plus
    the 48.7MB of the top layer. As we've seen above, this is also the size of the
    `fortune-clock:2.0` image.
  * The size of the `old-clock` and `new-clock` containers' top layers is 0 bytes, since no file was
    modified in these containers. Their virtual size correspond to their base images' size.

  Using all that we've learned, we can determine the total size taken up on your machine's file
  system:

  * The 113MB of the `ubuntu` image's layers, even though they are used by 3 images (the `ubuntu`
    image itself and the `fortune-clock:1.0` and `fortune-clock:2.0` images), are taken up only
    once.
  * Similarly, the 43MB of the `fortune-clock:1.0` image's additional layer are taken up only once,
    even though the layer is used by 2 images (the `fortune-clock:1.0` image itself and the
    `fortune-clock:2.0` image).
  * Finally, the 48.7MB of the `fortune-clock:2.0` image's additional layer are also taken up once.

  Therefore, the `ubuntu`, `fortune-clock:1.0` and `fortune-clock:2.0` images take up only 205MB of
  space on your file system, not 447MB. Basically, it's the same size as the `fortune-clock:2.0`
  image, since it re-uses the `fortune-clock:1.0` and `ubuntu` images' layers, and the
  `fortune-clock:1.0` image also re-uses the `ubuntu` image's layers.

  The `hello-world` image takes up an additional 1.85kB on your file system, since it has no layers
  in common with any of the other images.

[Back to top](#readme)

---

<br><br><br><br><br>





## Dockerfile

Manually starting containers, making changes and committing images is all well and good, but is
**prone to errors and not reproducible**.

Docker can build images automatically by reading the instructions from a [Dockerfile][dockerfile]. A
Dockerfile is a text document that contains all the commands a user could call on the command line
to assemble an image. Using the `docker build` command, users can create an **automated build** that
executes several command line instructions in succession.

[Back to top](#readme)

<br><br><br>

### The `docker build` command

This `docker build <context>` command builds an image from a **Dockerfile** and a **context**. The
build's context is the set of files at a specified path on your file system (or Git repository URL).
For example, running `docker build /foo` would expect to find a Dockerfile at the path
`/foo/Dockerfile`, and would use the entire contents of the `/foo` directory as the build context.

The build is run by the Docker daemon, not by the CLI. The first thing a build process does is
**send the entire context (recursively) to the daemon**. In most cases, it's best to start with an
empty directory as context and keep your Dockerfile in that directory. Add only the files needed for
building the Dockerfile.

**Warning:** do not use your root directory, `/`, as the build context as it causes the build to
transfer the entire contents of your hard drive to the Docker daemon.

To ignore some files in the build context, use a [`.dockerignore` file][docker-ignore] (similar to a
`.gitignore` file).

[Back to top](#readme)

<br>

### Format

The format of a Dockerfile is:

```
# Comment
INSTRUCTION arguments...
INSTRUCTION arguments...
```

You can find all available instructions, such as `FROM` and `RUN`, in the [Dockerfile
reference][dockerfile]. Many correspond to arguments or options of the Docker commands that we've
used. For example, the `FROM` instruction corresponds to the `<image>` argument of the `docker run
<image> [command...]` command, and specifies what base image to use.

[Back to top](#readme)

<br>

### Build an image from a Dockerfile

Clone or download this repository, then move to its `fortune-clock` directory in your console. You
will see that it contains the same clock script we used in the `fortune-clock:2.0` image, and a
Dockerfile:

```bash
$> cd fortune-clock

$> ls
Dockerfile clock.sh
```

The Dockerfile looks like this:

```
FROM ubuntu

RUN apt-get update
RUN apt-get install -y fortune
RUN apt-get install -y cowsay
COPY clock.sh /usr/local/bin/clock.sh
RUN chmod +x /usr/local/bin/clock.sh
```

It basically **replicates what we have done manually**:

* The `FROM ubuntu` instruction starts the build process from the `ubuntu` base image.
* The `RUN apt-get update` instruction executes the `apt-get update` command like we did before.
* The next two `RUN` instructions install the `fortune` and `cowsay` packages, also like we did
  before.
* The `COPY <src> <dest>` instruction copies a file from the build context into the file system of
  the container.  In this case, we copy the `clock.sh` file in the build context to the
  `/usr/local/bin/clock.sh` path in the container. When we run the build command, we will specify
  the `fortune-clock` directory of this repository as the build context, so that its `clock.sh` file
  is copied to the container.
* The final `RUN` instruction makes the script executable.

Remain in the `fortune-clock` directory and run the following build command.  The `-t` or `--tag
<repo:tag>` option indicates that we want to tag the image like we did when we were using the
`docker commit <container> <repo:tag>` command. The last argument, `.`, indicates that the build
context is the current directory:

```bash
$> docker build -t fortune-clock:3.0 .
Sending build context to Docker daemon  3.072kB
Step 1/6 : FROM ubuntu
 ---> c9d990395902
Step 2/6 : RUN apt-get update
 ---> Running in 6763a261b156
...
Removing intermediate container 6763a261b156
 ---> e131accf3a09
Step 3/6 : RUN apt-get install -y fortune
 ---> Running in a77bf3a72ded
...
Removing intermediate container a77bf3a72ded
 ---> 56969308655b
Step 4/6 : RUN apt-get install -y cowsay
 ---> Running in b7ab94de15e3
...
Removing intermediate container b7ab94de15e3
 ---> 844fbb19235c
Step 5/6 : COPY clock.sh /usr/local/bin/clock.sh
 ---> 9ff5a0b9ba2b
Step 6/6 : RUN chmod +x /usr/local/bin/clock.sh
 ---> Running in 6066a5e6a121
Removing intermediate container 6066a5e6a121
 ---> bcf88ef22f4c
Successfully built bcf88ef22f4c
Successfully tagged fortune-clock:3.0
```

As you can see, Docker:

* **Uploaded to build context** (i.e. the contents of the `fortune-clock` directory) to the Docker
  deamon.
* **Ran each instruction** in the Dockerfile **one by one**, creating an intermediate container each
  time, based on the previous state.
* **Created an image** with the final state, and the specified tag (i.e. `fortune-clock:3.0`).

You can see that new image in the list of images:

```bash
$> docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
fortune-clock       3.0                 bcf88ef22f4c        6 minutes ago       205MB
fortune-clock       2.0                 92bfbc9e4c4c        3 hours ago         205MB
fortune-clock       1.0                 407daed1a864        3 hours ago         156MB
ubuntu              latest              c9d990395902        10 days ago         113MB
hello-world         latest              e38bc07ac18e        11 days ago         1.85kB
```

You can also run a container based on it like we did before:

```bash
$> docker run --rm fortune-clock:3.0 clock.sh
It is Mon Apr 23 12:10:16 UTC 2018
 _____________________________________
/ Today is National Existential Ennui \
\ Awareness Day.                      /
 -------------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```

Use Ctrl-C to stop it.

Let's take a look at that new image's layers:

```bash
$> docker inspect fortune-clock:3.0
...
        "RootFS": {
            "Type": "layers",
            "Layers": [
                "sha256:fccbfa2912f0cd6b9d13f91f288f112a2b825f3f758a4443aacb45bfc108cc74",
                "sha256:e1a9a6284d0d24d8194ac84b372619e75cd35a46866b74925b7274c7056561e4",
                "sha256:ac7299292f8b2f710d3b911c6a4e02ae8f06792e39822e097f9c4e9c2672b32d",
                "sha256:a5e66470b2812e91798db36eb103c1f1e135bbe167e4b2ad5ba425b8db98ee8d",
                "sha256:a8de0e025d94b33db3542e1e8ce58829144b30c6cd1fff057eec55b1491933c3",
                "sha256:89d694b3a7a4bd236c161894c5aae7d6f86ec3e42c6b4a71032774e60d2d8e4a",
                "sha256:a96e0ea28068686e2bafd181958407d722e179ea5cc3138be5b130424f4925f1",
                "sha256:d57cb59abf79bbe2e4150450574110b862379ecf44d3a23956a965477a0a1848",
                "sha256:0b34384d1bf850620090e95b07cfcf2d552ec869b8f0ea574d8ae81acc2334d2",
                "sha256:a9b1e603de7dd53e2d6adcf3538d454a350e1384a4311b6d686151fabfa450fb"
            ]
        },
...
```

The first few layers (up to the one starting with `a8de0e025`) are the same as before, since they
are the `ubuntu` image's base layers. The last 5 layers, however, are new.

Basically, Docker created **one layer for each instruction in the Dockerfile**. Since we have 4
`RUN` instructions and 1 `COPY` instruction in the Dockerfile we used, there are 5 additional
layers.

[Back to top](#readme)

<br>

### Build cache

Re-run the same build command:

```bash
$> docker build -t fortune-clock:3.0 .
Sending build context to Docker daemon  3.072kB
Step 1/6 : FROM ubuntu
 ---> c9d990395902
Step 2/6 : RUN apt-get update
 ---> Using cache
 ---> e131accf3a09
Step 3/6 : RUN apt-get install -y fortune
 ---> Using cache
 ---> 56969308655b
Step 4/6 : RUN apt-get install -y cowsay
 ---> Using cache
 ---> 844fbb19235c
Step 5/6 : COPY clock.sh /usr/local/bin/clock.sh
 ---> Using cache
 ---> 9ff5a0b9ba2b
Step 6/6 : RUN chmod +x /usr/local/bin/clock.sh
 ---> Using cache
 ---> bcf88ef22f4c
Successfully built bcf88ef22f4c
Successfully tagged fortune-clock:3.0
```

It was much faster this time. As you can see by the `Using cache` indications in the output, Docker
is keeping a **cache of the previously built layers**. Since you have not changed the instructions
in the Dockerfile or the build context, it assumes that the result will be the same and reuses the
same layer.

Make a change to the `clock.sh` script in the `fortune-clock` directory. For example, add a new line
or a comment:

```bash
#!/bin/bash
trap "exit" SIGKILL SIGTERM SIGHUP SIGINT EXIT

# TODO: add new lines or a comment here!

# Print the date and a fortune every 5 seconds.
while true; do
  echo It is $(date)
  /usr/games/fortune | /usr/games/cowsay
  echo
  sleep 5
done
```

Re-run the same build command:

```bash
$> docker build -t fortune-clock:3.0 .
Sending build context to Docker daemon  3.072kB
Step 1/6 : FROM ubuntu
 ---> c9d990395902
Step 2/6 : RUN apt-get update
 ---> Using cache
 ---> e131accf3a09
Step 3/6 : RUN apt-get install -y fortune
 ---> Using cache
 ---> 56969308655b
Step 4/6 : RUN apt-get install -y cowsay
 ---> Using cache
 ---> 844fbb19235c
Step 5/6 : COPY clock.sh /usr/local/bin/clock.sh
 ---> 8575c9e05b3c
Step 6/6 : RUN chmod +x /usr/local/bin/clock.sh
 ---> Running in 037dc123faaa
Removing intermediate container 037dc123faaa
 ---> 99c24c7e3c1c
Successfully built 99c24c7e3c1c
Successfully tagged fortune-clock:3.0
```

Docker is still using its cache for the first 3 commands (the `apt-get update` and the installation
of the `fortune` and `cowsay` packages), since they are executed before the `clock.sh` script is
copied, and are therefore not affected by the change.

The `COPY` instruction is executed **without cache**, however, since Docker detects that **the
`clock.sh` script has changed**.

Consequently, **all further instructions** after that `COPY` **cannot use the cache**, since the
state upon which they are based has changed. Therefore, the last `RUN` instruction also does not use
the cache.

See [Squashing Image Layers][squashing-layers] for tips on how to minimize build time and the number
of layers.

[Back to top](#readme)

<br>

### A Dockerfile for a Node.js application

The `todo` directory contains a sample Node.js application to manage to-do notes.

If you wanted to run this application on your machine or on a server, you would need to:

* Install and run a [MongoDB][mongo] database (version 3).
* Install [Node.js][node] (version 8).
* Run `npm install` in the application's directory to install its dependencies.
* Run `npm start` in the application's directory to start it.
* (Additionally in a production environment: set up a process manager to keep it running if it
  crashes.)

That takes **some effort** if you don't already have the database or Node.js, or if you're not used
to installing and managing them. It could also be problematic if you already have **different,
non-compatible versions** of Node.js or MongoDB installed.

Running the application in a container can solve these problems. With what we've learned so far, you
could write a Dockerfile that installs Node.js, installs the application's dependencies and starts
it.

However, don't forget that images can be shared on the [Docker hub][hub] and reused. Popular
frameworks and languages already have official (or non-official) images you can use. For example,
the [`node` image][hub-node] already has Node.js installed: you only need to base your own image off
of it and add your application's code.

You will find a minimal Dockerfile to do that in the `Dockerfile` file in the `todo` directory of
this repository. This is what it contains:

```
FROM node:8

WORKDIR /usr/src/app
COPY . /usr/src/app/
RUN npm install

CMD [ "npm", "start" ]
```

Here's what the instructions are for:

* `FROM node:8` instructs Docker to build from the official `node:8` image (which contains the
  latest Node.js 8 version).
* `WORKDIR /usr/src/app` indicates the working directory in which commands are run (e.g. when using
  a `RUN` instruction). Why use the `/usr/src/app` directory? It's simply a popular convention for
  the directory of the application in Docker containers.
* `COPY . /usr/src/app` copies the entire contents of the build context to the `/usr/src/app`
  directory in the container.
* `RUN npm install` executes an `npm install` command to install the application's dependencies. Due
  to the previous `WORKDIR` instruction, this is executed in the `/usr/src/app` directory of the
  container.
* `CMD [ "npm", "start" ]` indicates the default command that will be executed when running this
  container. This is equivalent to the arguments we passed to `docker run <image> [command...]`. If
  a default command is specified in the image with `CMD`, you can simply use `docker run <image>` to
  run that command.

Move to the `todo` directory and build a new image based on that Dockerfile:

```bash
$> cd todo

$> docker build -t todo .
Sending build context to Docker daemon  60.93kB
Step 1/5 : FROM node:8
 ---> 4635bc7d130c
Step 2/5 : WORKDIR /usr/src/app
 ---> Using cache
 ---> 9e5af697d233
Step 3/5 : COPY . /usr/src/app/
 ---> 0c7a80e4fedc
Step 4/5 : RUN npm install
 ---> Running in 0122811036db
added 142 packages in 2.822s
Removing intermediate container 0122811036db
 ---> 13546c3ce198
Step 5/5 : CMD [ "npm", "start" ]
 ---> Running in 8becf1bd710d
Removing intermediate container 8becf1bd710d
 ---> a8dc25bf2972
Successfully built a8dc25bf2972
Successfully tagged todo:latest
```

You can now attempt to run the application:

```bash
$> docker run --rm todo

> todo@0.0.0 start /usr/src/app
> node ./bin/www

{ MongoNetworkError: failed to connect to server [localhost:27017] on first connect [MongoNetworkError: connect ECONNREFUSED 127.0.0.1:27017]
    ... }
npm ERR! code ELIFECYCLE
npm ERR! errno 1
npm ERR! todo@0.0.0 start: `node ./bin/www`
npm ERR! Exit status 1
npm ERR!
npm ERR! Failed at the todo@0.0.0 start script.
npm ERR! This is probably not a problem with npm. There is likely additional logging output above.

npm ERR! A complete log of this run can be found in:
npm ERR!     /root/.npm/_logs/2018-04-23T15_27_29_784Z-debug.log
```

It doesn't work because it attempts to connect to a MongoDB database on `localhost:27017` and there
is no such thing. Even if you do actually have a MongoDB database running on your machine on that
port for development, remember that each container has its own **isolated network stack**, so it
can't reach services listening on your host machine's ports.

We will run a database in the next section.

See [Dockerfile Tips][dockerfile-tips] for more information and good practices concerning
Dockerfiles.

[Back to top](#readme)

---

<br><br><br><br><br>





## Connecting containers

As we've seen, containers are **isolated by default**. Our Node.js application cannot run because it
cannot reach a database. If your application's environment was a virtual machine, your next step may
be to install and run a MongoDB server on that same virtual machine. You could add instructions to
the Dockerfile to do the same in the container we've built so far, but that would not be "the Docker
way".

Docker best practices suggest that **each container should have only one concern**.  Decoupling
applications into multiple containers makes it much easier to **scale horizontally** and **reuse
containers**. For instance, a web application stack might consist of three separate containers, each
with its own unique image, to manage the web application, database, and an in-memory cache in a
decoupled manner.

(You may have heard that there should be "one process per container". While this mantra has good
intentions, it is not necessarily true that there should be only one operating system process per
container. In addition to the fact that containers can now be spawned with an init process, some
programs might spawn additional processes of their own accord. For instance, Celery can spawn
multiple worker processes, or Apache might create a process per request. While "one process per
container" is frequently a good rule of thumb, it is not a hard and fast rule. Use your best
judgment to keep containers as clean and modular as possible.)

For our Node.js application, we will therefore run **2 containers**:

* 1 container to run a MongoDB server.
* 1 container to run the Node.js application.

This will make it easy to, for example, horizontally scale our application by running more than 1
Node.js application container, while keeping only 1 MongoDB server container.

(It would also be possible to run more than 1 MongoDB server container, but that would be more
complicated as it would require the configuration of [MongoDB replication][mongo-replication]. That
is outside the scope of this tutorial. Multiple application containers, however, can easily talk to
the same database with no additional configuration.)

[Back to top](#readme)

<br>

### Exposing container ports on the host machine

We will use the [official `mongo` image][hub-mongo] on Docker hub to **run a MongoDB database
container** for our application. [Its Dockerfile][hub-mongo-dockerfile] is more complex than the one
we've used as an example, but you should now easily understand what it does on principle (i.e.
install and configure what is needed to run a MongoDB server and run the appropriate command to
start it).

Run a MongoDB 3+ container named `db` with the following command:

```bash
$> docker run --name db --rm mongo:3
Unable to find image 'mongo:3' locally
3: Pulling from library/mongo
...
Digest: sha256:670f9ea4f85f7e188cb0f572261feb1f2e170ee593ff3981474395e145a0c062
Status: Downloaded newer image for mongo:3
2018-04-25T08:41:21.614+0000 I CONTROL  [initandlisten] MongoDB starting : pid=1 port=27017 dbpath=/data/db 64-bit host=608a8a274da2
...
2018-04-25T08:41:22.096+0000 I NETWORK  [initandlisten] waiting for connections on port 27017
```

If you open another command line console, you can inspect that container and find its IP address:

```bash
$> docker inspect db
...
    "Networks": {
        "bridge": {
            "IPAMConfig": null,
            "Links": null,
            "Aliases": null,
            "NetworkID": "4156a5216eed6ee7715a18fed1586620fc6ec87fb7ddc6c7f15032f640816850",
            "EndpointID": "ffd645200b204475441262772f4b4cbb2d7a2209aaa39df19e7a966931e915e2",
            "Gateway": "172.17.0.1",
            "IPAddress": "172.17.0.2",
            "IPPrefixLen": 16,
            "IPv6Gateway": "",
            "GlobalIPv6Address": "",
            "GlobalIPv6PrefixLen": 0,
            "MacAddress": "02:42:ac:11:00:02",
            "DriverOpts": null
        }
    }
...
```

On a Linux host machine, you could connect to that IP address directly on port 27017 to reach the
database. (It might not work on macOS & Windows machines because some Docker installations use an
intermediate Linux virtual machine to run the containers, so your machine might not actually be the
host Docker machine.)

However, **that IP address is not predictable** so that's not a good solution. You can **expose a
container's port on your host machine** by adding the `-p` or `--publish <hostPort:containerPort>`
option to the `docker run` command.

Stop the MongoDB container with Ctrl-C and start another one with this command:

```bash
$> docker run --name db -p 5000:27017 --rm mongo:3
...
2018-04-25T08:41:22.096+0000 I NETWORK  [initandlisten] waiting for connections on port 27017
```

The above command publishes the container's 27017 port to your host machine's 5000 port. If you have
a MongoDB client on your host machine, you can now connect to the database with the following
command:

```bash
$> mongo localhost:5000
MongoDB shell version v3.6.3
connecting to: mongodb://localhost:5000/test
MongoDB server version: 3.6.4
...
>
```

Exit the MongoDB shell with `exit`. Stop the MongoDB container with Ctrl-C.

This works, but we can't use this method to connect our Node.js application container to our MongoDB
server container. You can reach any container from the host machine, but **containers cannot
reach your host machine's ports**.

[Back to top](#readme)

<br>

### Docker networks

You can create **networks** to break the isolation between containers and connect them together.
List available networks with the `docker network ls` command:

```bash
$> docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
4156a5216eed        bridge              bridge              local
c7777321c9e3        host                host                local
cd79f5b6d678        none                null                local
```

Read the [Docker Networking Overview][docker-networking] to learn about the different network
drivers such as `bridge`, `host` and `none`. The links in that article will also give more
information on how Docker networks work at the OS level.

For now, know that a [**bridge network**][docker-bridge-networks] uses a software bridge which
**allows containers connected to the same network to communicate**, while providing isolation from
containers which are not connected to that network. The Docker bridge driver automatically installs
rules in the host machine so that **containers on different bridge networks cannot communicate
directly with each other**.

A bridge network named `bridge` exists by default. New containers connect to it unless otherwise
specified. However, we will not use this default network, as **user-defined bridge networks are
superior** to the default bridge network. We will see why shortly.

Let's create a network for our application with the `docker network create <name>` command:

```bash
$> docker network create todo
15109ea2a697b8be45b02511fdc217f3707c3489cbcfdf4cebf49e968d2bc1e3
```

We can see it in the list now:

```bash
$> docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
4156a5216eed        bridge              bridge              local
c7777321c9e3        host                host                local
cd79f5b6d678        none                null                local
15109ea2a697        todo                bridge              local
```

[Back to top](#readme)

<br>

#### Running a container in a network

To run the MongoDB server container in our user-defined `todo` network, add the `--network <name>`
option to the `docker run` command. This time, we'll also run the container in the background with
the `-d` option:

```bash
$> docker run -d --name db --network todo mongo:3
6f6066b97321a4f333f7dc8cb4364b719bba795e6deea79e9fd796946de623cd
```

Make sure the container is running:

```bash
$> docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED                  STATUS              PORTS               NAMES
6f6066b97321        mongo:3             "docker-entrypoint.s…"   Less than a second ago   Up 4 seconds        27017/tcp           db
```

If you inspect it, you will see that it is indeed connected to the `todo` network instead of the
`bridge` network:

```bash
$> docker inspect db
...
    "Networks": {
        "todo": {
            "IPAMConfig": null,
            "Links": null,
            "Aliases": [
                "6f6066b97321"
            ],
            "NetworkID": "15109ea2a697b8be45b02511fdc217f3707c3489cbcfdf4cebf49e968d2bc1e3",
            "EndpointID": "c4428534c06e83751a3fe10d530481888b3101645b666f75aafad21634cedc10",
            "Gateway": "172.18.0.1",
            "IPAddress": "172.18.0.2",
            "IPPrefixLen": 16,
            "IPv6Gateway": "",
            "GlobalIPv6Address": "",
            "GlobalIPv6PrefixLen": 0,
            "MacAddress": "02:42:ac:12:00:02",
            "DriverOpts": null
        }
    }
...
```

Now that our MongoDB server container is running, we can attempt to connect our Node.js application
to it. Attempt to run a container based on our `todo` image as before, but this time add the
`--network` option like you did for the other container:

```bash
$> docker run --name app --network todo --rm todo

> todo@0.0.0 start /usr/src/app
> node ./bin/www

{ MongoNetworkError: failed to connect to server [localhost:27017] on first connect [MongoNetworkError: connect ECONNREFUSED 127.0.0.1:27017]
    ... }
npm ERR! code ELIFECYCLE
npm ERR! errno 1
npm ERR! todo@0.0.0 start: `node ./bin/www`
npm ERR! Exit status 1
...
```

It still does not work because the Node.js application still attempts to connect to its default
database address of `localhost:27017`. Fortunately, a custom database URL can be provided to the
application by setting the `$DATABASE_URL` environment variable. But **what address should we use?**

This is where Docker's networking magic comes into play. We mentioned earlier that user-defined
bridge networks are superior to Docker's default bridge network. **User-defined bridges provide
automatic DNS resolution between containers**, i.e. containers can resolve each other by name or
alias.

Since we added the `--name db` option when running our MongoDB container, any container on the same
network can reach it at the `db` address (which is simply a host like `localhost` or `example.com`).

Simply add the `-e` or `--env <VARIABLE=value>` option to the `docker run` command to define a new
environment variable. We'll also add the `-d` option to run it in the background, and a `-p` option
to publish its 3000 port on our host machine:

```bash
$> docker run -d -e "DATABASE_URL=mongodb://db:27017" --name app --network todo -p 3000:3000 todo
8e7b4d08691fa46c93afa80c6ec76a9be7bb768b699e0bdd2353df682568fe18
```

Both our containers are now running:

```bash
$> docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
8e7b4d08691f        todo                "npm start"              3 minutes ago       Up 3 minutes        0.0.0.0:3000->3000/tcp   app
6f6066b97321        mongo:3             "docker-entrypoint.s…"   16 minutes ago      Up 16 minutes       27017/tcp                db
```

The application should be working and accessible at
[`http://localhost:3000`](http://localhost:3000)!

![To-do application](images/todo.png)

Play with it and use `docker logs app` to see that the Node.js application is indeed processing your
requests.

[Back to top](#readme)

---

<br><br><br><br><br>





## Persistent storage

Create a few to-do notes in the running application.

Stop both containers and restart them:

```bash
$> docker stop app db
app
db

$> docker start db
db

$> docker start app
app
```

(Wait a few seconds after starting the `db` container before starting the `app` container. The
Node.js application cannot start unless it successfully connects to the database. If the `app`
container doesn't start the first time, re-run the `docker start app` command. See [Dockerfile
Tips][dockerfile-tips] and [Waiting for other containers][dockerfile-tips-waiting] for a solution to
this problem.)

The application should again be running on [`http://localhost:3000`](http://localhost:3000), and
your to-do notes should still be there.

Now let's see what happens if you stop *and remove* both containers:

```bash
$> docker rm -f app db
app
db
```

Now re-run both containers with the same commands as before:

```bash
$> docker run -d --name db --network todo mongo:3
f8afad7282fad05fb16230ef4c56a96bef969b7f3dad9c312b4a6b8c4024cd43

$> docker run -d -e "DATABASE_URL=mongodb://db:27017" --name app --network todo -p 3000:3000 todo
b4dfaf7d0d9f4669b13dac759c4b7858b0c23537a31183c7c757b8c9d48825b8
```

New container instances should be running:

```bash
$> docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
b4dfaf7d0d9f        todo                "npm start"              7 seconds ago       Up 14 seconds       0.0.0.0:3000->3000/tcp   app
f8afad7282fa        mongo:3             "docker-entrypoint.s…"   20 seconds ago      Up 28 seconds       27017/tcp                db
```

However, your to-do notes are gone!

This is because, as we've seen, container data is written to a **top writable layer** in the union
file system, and **that layer is deleted when the container is removed**.

Let's see how to [manage data in Docker][docker-storage]. There are 3 solutions:

* [Volumes][docker-storage-volumes] - Store data in a volume created and managed by Docker.
* [Bind mounts][docker-storage-bind] - Mount a file/directory of the host machine into a container.
* [tmpfs mounts][docker-storage-tmpfs] - Mount an in-memory directory into a container to store
  non-persistent state or sensitive information.

![Docker storage types](images/docker-storage.png)

We'll talk about the first 2.

[Back to top](#readme)

<br>

### Bind mounts

[Bind mounts][docker-storage-bind] have been around since the early days of Docker. When you use a
bind mount, a **file or directory on the host machine is mounted into a container**. The file or
directory is referenced by its full or relative path on the host machine.

Bind mounts are very performant, but they rely on the host machine's filesystem having a specific
directory structure available, making the container **less portable**. If you are developing new
Docker applications, consider using [named volumes][docker-storage-volumes] instead (which we'll see
in the next section). You can't use Docker commands to directly manage bind mounts.

Let's stop the `app` and `db` containers and remove the `db` container. As before, this will clear
all our data:

```bash
$> docker stop app db
app
db

$> docker rm db
```

If you read the [official `mongo` image][hub-mongo]'s documentation, under its *Where to Store Data*
heading, you will see that it stores the MongoDB server's data under the `/data/db` path in the
container's file system. Let's bind mount a directory on your host machine to that directory, so
that the data persists even after the container is removed.

Run the database again, but add the `-v` or `--volume <hostDir:containerDir>` option to the `docker
run` command:

```bash
$> docker run -d --name db --network todo -v ~/docker-brownbag-db:/data/db mongo:3
0678bd8cafde87a6d0a6c29022d567d316d3a735229c08ab5387737644143283
```

This instructs Docker to create the `~/docker-brownbag-db` on your host machine, and to **mount it
into the running container** at the `/data/db` path (overwriting anything that was already there).

Give the MongoDB server a second or two to initialize, then list the contents of the
`~/docker-brownbag-db` on your machine:

```bash
$> ls -1 ~/docker-brownbag-db
WiredTiger
WiredTiger.lock
WiredTiger.turtle
WiredTiger.wt
WiredTigerLAS.wt
_mdb_catalog.wt
collection-0--8103351089522900052.wt
collection-2--8103351089522900052.wt
diagnostic.data
index-1--8103351089522900052.wt
index-3--8103351089522900052.wt
journal
mongod.lock
sizeStorer.wt
storage.bson
```

As you can see, the MongoDB server's data is indeed being stored into that directory. Note that the
MongoDB server itself doesn't know about your `~/docker-brownbag-db` directory. From its point of
view, it's simply writing files under `/data/db` in the container's file system. The Docker bind
mount makes these writes go to your host machine instead of the container's top writable layer.

Start the `app` container again:

```bash
$> docker start app
app
```

Now play with the application–create a few todo-notes.

Stop both containers again and remove the `db` container. As before, the top writable layer of the
`db` container is deleted:

```bash
$> docker stop app db
app
db

$> docker rm db
db
```

Now re-run the `db` container, giving the same volume option as before, and restart the `app`
container as well:

```bash
$> docker run -d --name db --network todo -v ~/docker-brownbag-db:/data/db mongo:3
12b985e23b4bea3b25595e0a2c52cf85693cdc1e2851a7197e93929da3aeecf7

$> docker start app
app
```

This time, the persisted MongoDB server's data was mounted into the new container. Instead of
initializing from scratch, **the MongoDB server loaded the existing data** (it's as if it was simply
restarted). Your to-do notes are still here!

[Back to top](#readme)

<br>

### Volumes

As mentioned, bind mounts are not ideal for container portability.

[Volumes][docker-storage-volumes] are the preferred mechanism for persisting data generated by and
used by Docker containers.  While bind mounts are dependent on the directory structure of the host
machine, volumes are completely managed by Docker. Volumes have several advantages over bind mounts:

* Volumes are easier to back up or migrate than bind mounts.
* You can manage volumes using Docker CLI commands or the Docker API.
* Volumes work on both Linux and Windows containers.
* Volumes can be more safely shared among multiple containers.
* Volume drivers allow you to store volumes on remote hosts or cloud providers, to encrypt the
  contents of volumes, or to add other functionality.
* A new volume's contents can be pre-populated by a container.


In addition, volumes are often a better choice than persisting data in a container's writable layer,
because using a volume does not increase the size of containers using it, and the volume's contents
exist outside the lifecycle of a given container.

Stop both containers again, remove the `db` container and the `~/docker-brownbag-db` directory on
your machine:

```bash
$> docker stop app db
app
db

$> docker rm db
db

$> rm -fr ~/docker-brownbag-db
```

To run a MongoDB server container with a volume instead of a bind mount, use the same volume option,
but use an arbitrary name instead of a host machine directory, i.e. `todo_data` instead of
`~/docker-brownbag-db`. Because it's not an absolute path (it doesn't start with `~` or `/`), Docker
interprets it as the name of a Docker volume, which it will automatically create if it doesnt exist.

```bash
$> docker run -d --name db --network todo -v todo_data:/data/db mongo:3
b53c96764916b3531909171aeade9be2d86acfeccbf23a4128e55a04709d25d7
```

You can list volumes with the `docker volume ls` command:

```bash
$> docker volume ls
DRIVER              VOLUME NAME
...
local               todo_data
```

Start the `app` container again, and you should be good to go:

```bash
$> docker start app
app
```

Create a few todo-notes, then stop both containers and remove the `db` container again, and finally
re-run the `db` container with the same command:

```bash
$> docker stop app db
app
db

$> docker rm db
db

$> docker run -d --name db --network todo -v todo_data:/data/db mongo:3
3ac1a6846a805d7821f8b80c230fe318fcb626b956b5b447e756cf1da1e221ae
```

Again, the MongoDB server's data persisted in the Docker volume, and **a volume is not deleted when
containers using it are removed**, so your to-do notes are still here!

If you inspect the volume, you can see where its data is actually stored:

```bash
$> docker inspect todo_data
[
    {
        "CreatedAt": "2018-04-25T11:31:09Z",
        "Driver": "local",
        "Labels": null,
        "Mountpoint": "/var/lib/docker/volumes/todo_data/_data",
        "Name": "todo_data",
        "Options": {},
        "Scope": "local"
    }
]
```

(Note that this represents the path of the volume on the Docker host. When using Docker on macOS or
Windows, this directory might not exist on your machine due to the intermediate virtual machine that
is sometimes used. In that case, it exists on the virtual machine's file system.)

[Volume drivers][docker-storage-volume-drivers] allow very flexible management of your data, such as
storing it on external services (e.g. cloud providers), transparently encrypting content, etc.

[Back to top](#readme)

---

<br><br><br><br><br>





## Debugging containers

A very useful command to debug containers is `docker exec <command...>`. It executes a command **in
a running container**.

For example, let's say you want to check what's in the `/usr/src/app` directory in the `app`
container:

```bash
$> docker exec app ls -la /usr/src/app
total 104
drwxr-xr-x    1 todo     todo          4096 Apr 25 13:55 .
drwxr-xr-x    1 root     root          4096 Apr 23 15:22 ..
-rw-r--r--    1 todo     todo            13 Apr 23 14:41 .dockerignore
-rw-r--r--    1 todo     todo            95 Apr 23 15:05 Dockerfile
-rw-r--r--    1 todo     todo           518 Apr 25 07:15 Dockerfile.full
-rw-r--r--    1 todo     todo          1242 Apr 25 13:37 app.js
drwxr-xr-x    2 todo     todo          4096 Apr 23 13:21 bin
-rw-r--r--    1 todo     todo           458 Apr 25 13:55 docker-compose-full.yml
-rw-r--r--    1 todo     todo           305 Apr 25 13:53 docker-compose.yml
-rwxr-xr-x    1 todo     todo           550 Apr 25 06:58 entrypoint.sh
drwxr-xr-x    2 todo     todo          4096 Apr 23 13:31 models
-rw-r--r--    1 todo     todo           746 Apr 25 13:36 nginx.conf
drwxr-xr-x  131 todo     todo          4096 Apr 25 12:42 node_modules
-rw-r--r--    1 todo     todo         36450 Apr 25 12:30 package-lock.json
-rw-r--r--    1 todo     todo           293 Apr 25 12:30 package.json
drwxr-xr-x    3 todo     todo          4096 Apr 23 14:33 public
drwxr-xr-x    2 todo     todo          4096 Apr 23 13:28 routes
drwxr-xr-x    2 todo     todo          4096 Apr 23 13:21 views
```

Or what's in the `/data/db` directory in the `db` container:

```bash
$> docker exec db ls -1 /data/db
WiredTiger
WiredTiger.lock
WiredTiger.turtle
WiredTiger.wt
WiredTigerLAS.wt
_mdb_catalog.wt
collection-0--7247392160342363554.wt
collection-2--7247392160342363554.wt
collection-4--7247392160342363554.wt
collection-7--7247392160342363554.wt
diagnostic.data
index-1--7247392160342363554.wt
index-3--7247392160342363554.wt
index-5--7247392160342363554.wt
index-6--7247392160342363554.wt
index-8--7247392160342363554.wt
journal
mongod.lock
sizeStorer.wt
storage.bson
```

You can execute any available command, including a full shell (if there is one in your container's
file system). For example, let's run a shell in the `app` container:

```bash
$> docker exec -it app sh
/usr/src/app $
```

You're now in the container! You can run any command you want:

```bash
/usr/src/app $ echo hello from $(hostname)
hello from 5914ad5835e4

/usr/src/app $ ls
Dockerfile               docker-compose.yml       package-lock.json
Dockerfile.full          entrypoint.sh            package.json
app.js                   models                   public
bin                      nginx.conf               routes
docker-compose-full.yml  node_modules             views
```

Run `exit` once you're done:

```bash
/usr/src/app $ exit
```

[Back to top](#readme)

<br>

### Ephemeral containers

You could make changes to a running container using `docker exec`, but that's considered a bad
practice.

Containers produced by your Dockerfiles should be as **ephemeral** as possible. By "ephemeral", we
mean that they can be **stopped and destroyed** and a **new one built and put in place** with an
**absolute minimum of setup and configuration**. You shouldn't have to perform additional manual
changes in a container once it's started.

You may want to take a look at the [Processes][12factor-processes] section of the [12 Factor app
methodology][12factor] to get a feel for the motivations of running containers in such a stateless
fashion.

[Back to top](#readme)

---

<br><br><br><br><br>





## Docker Compose

[Docker Compose][docker-compose] is a tool for defining and running multi-container Docker
applications. With Compose, you use a **[YAML][yaml] file to configure your application's
services**.  Then, with a single command, you **create and start all the services from your
configuration**. It enables:

* Multiple isolated environments on a single host.
* Preserve volume data when containers are created.
* Only recreate containers that have changed.
* Variables and moving a composition between environments.

Using Compose is basically a three-step process:

* Define your app's environment with a Dockerfile so it can be reproduced anywhere (we've already
  done that in the previous examples).
* Define the services that make up your app in `docker-compose.yml` so they can be run together in
  an isolated environment.
* Run `docker-compose up` and Compose starts and runs your entire app.

Compose has commands for managing the whole lifecycle of your application:

* Start, stop, and rebuild services.
* View the status of running services.
* Stream the log output of running services.
* Run a one-off command on a service.

[Back to top](#readme)

<br>

### The `docker-compose.yml` file

If the containers from the previous steps are still running, remove them and also remove the Docker
image, network and data volume we created for the to-do application, so that we start from scratch:

```bash
$> docker rm -f app db
app
db

$> docker network rm todo
todo

$> docker volume rm todo_data
todo_data

$> docker rmi todo
...
```

It's a pain to remember the full commands we used before with all the correct options to build and
start our 2-container infrastructure in a bridge network with a persistent data volume. If only
there was a more reproducible solution.

Enter the `docker-compose.yml` file:

```
version: '3'

services:

  app:
    image: docker-brownbag/todo
    build:
      context: .
      dockerfile: Dockerfile.full
    depends_on:
      - db
    environment:
      DATABASE_URL: "mongodb://db:27017"
    ports:
      - "3000:3000"

  db:
    image: mongo:3
    volumes:
      - data:/data/db

volumes:
  data:
```

This file defines 2 `app` and `db` **services** that can be managed with Docker Compose.

Let's go into more details, looking at the `app` service first:

```
app:
  image: docker-brownbag/todo
  build:
    context: .
    dockerfile: Dockerfile.full
  depends_on:
    - db
  environment:
    DATABASE_URL: "mongodb://db:27017"
  ports:
    - "3000:3000"
  restart: always
```

It's basically another way to indicate all the options and arguments we've passed to Docker commands
so far::

* The `context: .` option under `build:` indicates that the `app` service should be built with
  `docker build` with the current directory as the build context (where we will run the
  `docker-compose` command).
* The `dockerfile: Dockerfile.full` option indicates that the Dockerfile that should be used is the
  file named `Dockerfile.full` instead of the default `Dockerfile`. (For the purposes of this
  example, `Dockerfile.full` implements all suggested tips under [Dockerfile
  Tips][dockerfile-tips].)
* The `image: docker-brownbag/todo` option indicates that the resulting image should be tagged as
  `todo`, exactly like `-t todo` option of the `docker build` command.
* The `depends_on: [ db ]` option indicates that the `app` service depends on the `db` service, and
  therefore the `db` service should be started first.
* The `environment: ...` option is a map of environment variables to add to the service's
  containers, exactly like the `--env` option of the `docker run` command.
* The `ports: [ "3000:3000" ]` option indicates that the container's 3000 port should be published
  to the host machine's 3000 port, exactly like the `--publish` option of the `docker run` command.

Let's look at the `db` service:

```
db:
  image: mongo:3
  volumes:
    - data:/data/db
```

* The `image: mongo:3` option indicates that a container should be launched from the `mongo:3` image
  from the Docker hub, exactly like the first argument to the `docker run <image>` command.
* The `volumes: [ data:/data/db ]` option indicates that a named Docker data volume
  should be mounted into the container's file system at the `/data/db` path.

Additionally, because we will run Docker Compose in the `todo` directory of the repository, Compose
will assume that the project is named `todo`, and will use that to do a few things:

* Container, network and volume names will be prefixed with the project name. For example, the data
  volume is defined as `data` in the `db` service, so its full name will be `todo_data`.
* Compose will automatically create a bridge network for our 2 services, which it will name
  `default` with the project name as a prefix, so `todo_default` in our case.

(You may specify another project name with the `-p` or `--project-name <name>` option of the
`docker-compose` command.)

Finally, named data volumes must be declared, which is what the last section is for:

```
volumes:
  data:
```

(There is nothing under `data:` because the default options are appropriate for this example.)

[Back to top](#readme)

<br>

### Running Docker Compose services

To redo everything we have done so far manually, this time with Docker Compose, all you have to do
is go into the `todo` directory, and run the `docker-compose up` command:

```bash
$> docker-compose up --build -d
Creating network "todo_default" with the default driver
Creating volume "todo_data" with default driver
Building app
...
Successfully built ab624a535392
Successfully tagged docker-brownbag/todo:latest
Creating todo_db_1 ... done
Creating todo_app_1 ... done
```

As you can see, it has:

* Created a network.
* Created a data volume.
* Built the `todo` image.
* Run 2 containers named `todo_app_1` and `todo_db_1` for the Node.js application and MongoDB
  server, respectively.

List networks, running containers and volumes to make sure:

```bash
$> docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
...
fc5505683b61        todo_default        bridge              local

$> docker ps
CONTAINER ID        IMAGE     COMMAND                  CREATED                  STATUS              PORTS                    NAMES
a45ab4f341ba        todo      "npm start"              Less than a second ago   Up 2 seconds        0.0.0.0:3000->3000/tcp   todo_app_1
4ea3b100947d        mongo:3   "docker-entrypoint.s…"   Less than a second ago   Up 2 seconds        27017/tcp                todo_db_1

$> docker volume ls
DRIVER              VOLUME NAME
...
local               todo_data
```

You can test the to-do application at [`http://localhost:3000`](http://localhost:3000) to make sure
it runs as before.

To test data persistence, stop and remove all services defined in the `docker-compose.yml` file with
the `docker-compose stop` and `docker-compose rm` commands:

```bash
$> docker-compose stop
Stopping todo_app_1 ... done
Stopping todo_db_1  ... done

$> docker-compose rm
Going to remove todo_app_1, todo_db_1
Are you sure? [yN] y
Removing todo_app_1 ... done
Removing todo_db_1  ... done
```

The network and data volume remain.  (You could also use `docker-compose down` which would do the
same thing and also remove the network.)

To restart both containers, simply use `docker-compose up` again:

```bash
$> docker-compose up -d
Creating todo_db_1 ... done
Creating todo_app_1 ... done
```

Docker Compose has many utility commands which **simplify working with a multi-container
application**.  For example, the `docker-compose ps` command **lists services' containers**:

```bash
$> docker-compose ps
   Name                Command             State           Ports
-------------------------------------------------------------------------
todo_app_1   npm start                     Up      0.0.0.0:3000->3000/tcp
todo_db_1    docker-entrypoint.sh mongod   Up      27017/tcp
```

The `docker-compose logs <service>` command allows you to **check a service's logs** without knowing
the exact name or ID of the containers:

```bash
$> docker-compose logs app
Attaching to todo_app_1
app_1  |
app_1  | > todo@0.0.0 start /usr/src/app
app_1  | > node ./bin/www
app_1  |
app_1  | GET / 304 277.099 ms - -
app_1  | GET /javascripts/client.js 200 4.445 ms - 656
app_1  | Mongoose: todos.find({}, { sort: { createdAt: 1 }, fields: {} })
app_1  | GET /todos 304 16.767 ms - -
```

And [much more][docker-compose-cli].

[Back to top](#readme)

<br>

### Rebuilding Docker Compose services

If you attempt to run `docker-compose up` again with the `--build` option, note that it does
nothing:

```bash
$> docker-compose up --build -d
Building app
...
 ---> Using cache
...
Successfully built 8215f1cda8b6
Successfully tagged docker-brownbag/todo:latest
todo_db_1 is up-to-date
todo_app_1 is up-to-date
```

It indicates that both containers are "up-to-date", meaning that since the images have not changed,
it left the existing containers running and did not attempt to restart them.

Make a change to the application. For example, open `todo/public/javascripts/client.js` and change
the `title` property of the application. Then rebuild it:

```bash
$> docker-compose up --build -d
Building app
...
 ---> Using cache
 ---> 1ad095898f72
Step 9/13 : COPY --chown=todo:todo . /usr/src/app/
 ---> 4d6d681d4444
...
Successfully built e67c21b4e1ab
Successfully tagged docker-brownbag/todo:latest
todo_db_1 is up-to-date
Recreating todo_app_1 ... done
```

As expected, the build cache was invalidated since the application changed. Docker Compose therefore
**recreated the `todo_app_1` container**. But it still **left `todo_db_1` intact** since the change
did not affect the database, so no recreation was necessary.

[Back to top](#readme)

<br>

### Starting containers automatically

There are several reasons you might want to **start your containers automatically** rather than
manually:

* You want them to **restart automatically if the host machine reboots**.
* You want them to **restart automatically if the process in the container crashes** due to a bug,
  causing the container to stop.

Docker can be instructed to automatically start containers by using a **restart policy** either when
running containers manually or when using Docker Compose:

* Add a `--restart <policy>` option to your `docker run` command.
* Add a `restart: <policy>` option to your `docker-compose.yml` file.

For example, here's what the services in the previous `docker-compose.yml` example look like with an
additional restart policy:

```
app:
  image: docker-brownbag/todo
  build:
    context: .
    dockerfile: Dockerfile.full
  depends_on:
    - db
  environment:
    DATABASE_URL: "mongodb://db:27017"
  restart: always
  ports:
    - "3000:3000"

db:
  image: mongo:3
  restart: always
  volumes:
    - data:/data/db
```

That way, Docker can also fulfill the role of process manager.

The available restart policies are:

* `no` - Do not automatically restart the container. (the default).
* `on-failure` - Restart the container if it exits due to an error, which manifests as a non-zero
  exit code.
* `unless-stopped` - Restart the container unless it is explicitly stopped or Docker itself is
  stopped or restarted.
* `always` - Always restart the container if it stops.

[Back to top](#readme)

<br>

### Horizontal scaling

Let's say we expect many users to use our to-do app, and we're worried that our new infrastructure
won't be able to handle the load. We're going to [**scale horizontally**][horizontal-scaling] by
deploying **multiple containers for our Node.js application**.

(As mentioned before, we'll keep only 1 MongoDB server container running, because setting up MongoDB
replication takes a bit more work.)

If you've followed the previous examples, run the following command to bring down the entire
infrastructure (except persistent data volumes):

```bash
$> docker-compose down
Stopping todo_app_1 ... done
Stopping todo_db_1  ... done
Removing todo_app_1 ... done
Removing todo_db_1  ... done
Removing network todo_default
```

So far, our `app` container has its 3000 port published on your machine's 3000 port. That won't work
if you deploy more than 1 container, as they **can't all listen on the same port**. You'll have to
put a **load balancer** in front of your `app` containers, so that it can listen on the port, and
redirect requests to the other containers.

Make the following changes to the `docker-compose.yml` in the `todo` directory:

* Remove the `ports: [ "3000:3000" ]` option from the `app` service.
* Add a new `lb` (load balancer) service with the following configuration:

  ```
  lb:
    depends_on:
      - app
    image: nginx:1.13-alpine
    ports:
      - "3000:80"
    restart: always
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
  ```

(You will find the expected result in the `docker-compose-full.yml` file in the `todo` directory.)

We're using the [official `nginx` image][hub-nginx] from the Docker hub, which will launch an
[nginx][nginx] web server that will act as our [load balancer][nginx-lb]. Its 80 port will be
published to your machine's 3000 port so that you'll be able to access `http://localhost:3000` like
before.

The `volumes` option defines a bind mount that will mount the `nginx.conf` file in the `todo`
directory to `/etc/nginx/nginx.conf` in the container's file system. That is the default location
for nginx's configuration file. The final `:ro` option defines the mount as read-only (nginx only
needs to read from that file, not write to it).

Look at the interesting parts of the `nginx.conf` file, the `upstream` and `server` directives:

```
upstream todo_cluster {
  server todo_app_1:3000;
  server todo_app_2:3000;
  server todo_app_3:3000;
}

server {
  listen 80;
  server_name localhost;

  location / {
    proxy_pass http://todo_cluster;
  }
}
```

The `upstream` directive defines a group of 3 addresses that nginx can redirect traffic to. By
default, nginx will use a round-robin strategy, meaning that each request will be redirected to the
next address. The `proxy_pass` directive instructs nginx to redirect all requests to the upstream
group.

You can **run this entire infrastructure** like before with the `docker-compose up` command. Note
the use of the `--scale <service=num>` option to define the number of containers to create for the
`app` service:

```bash
$> docker-compose up --build -d --scale app=3
Creating network "todo_default" with the default driver
Building app
...
Successfully built 416a49a3a498
Successfully tagged todo:latest
Creating todo_lb_1 ... done
Creating todo_db_1 ... done
Creating todo_app_1 ... done
Creating todo_app_2 ... done
Creating todo_app_3 ... done
```

You now have 1 database container, 3 application containers and 1 load balancer containers running:

```bash
$> docker-compose ps
   Name                 Command               State          Ports
--------------------------------------------------------------------------
todo_app_1   /usr/local/bin/entrypoint. ...   Up      3000/tcp
todo_app_2   /usr/local/bin/entrypoint. ...   Up      3000/tcp
todo_app_3   /usr/local/bin/entrypoint. ...   Up      3000/tcp
todo_db_1    docker-entrypoint.sh mongod      Up      27017/tcp
todo_lb_1    nginx -g daemon off;             Up      0.0.0.0:3000->80/tcp
```

The to-do application should run as before on [`http://localhost:3000`](http://localhost:3000),
except that the "Running on host ..." message should change every time you reload the page, proving
that you are in fact communicating with each container of the `app` service in turn.

This is nice, but the `upstream` block in the `nginx.conf` configuration file is **hardcoded** to
load balance over exactly 3 containers. Incidentally, nginx won't start if any of the containers is
unreachable. This is not as flexible as we might want.

There are several solutions to this problem. Here's two:

* Use a [service discovery][service-discovery] tool like [Consul][consul], [Serf][serf] or
  [ZooKeeper][zookeeper]. Setting up such a solution can enable the `lb` container to be aware of
  what other containers are deployed at a given time, and to automatically update its configuration
  when containers are started or stopped.
* Use [Docker in swarm mode][docker-swarm], because it can manage services and load balancing for
  you. You'll see in action if you [read on](#docker-swarm).

[Back to top](#readme)

---

<br><br><br><br><br>





## Docker Swarm

Docker comes with [SwarmKit][swarmkit], a cluster management and orchestration tool.

![Docker Swarm](images/swarm.png)

A **swarm** consists of **multiple Docker nodes**, i.e. separate computers or cloud servers, which
run in swarm mode and act as **managers** (to manage membership and delegation) and/or **workers**
(which run swarm services).

A **service** is a definition of the tasks to execute on the manager or worker nodes. It is the
central structure of the swarm system and the primary root of user interaction with the swarm.  When
you create a service, you specify **which container image to use** and **which commands** to execute
inside running containers.

When you define a service, you describe its optimal state (number of replicas, network and storage
resources available to it, ports the service exposes to the outside world, and more). Docker works
to maintain that desired state. For instance, if a worker node becomes unavailable, Docker schedules
that node's tasks on other nodes.

A **task** carries a Docker **container** and the **commands** to run inside the container. It is
the **atomic scheduling unit** of swarm. Manager nodes assign tasks to worker nodes according to the
number of **replicas** set in the service scale.

The advantages of Docker Swarm over standalone containers and Docker Compose are:

* SwarmKit uses the [Raft Consensus Algorithm][raft] in order to coordinate and **does not rely on a
  single point of failure** to perform decisions.
* Node communication and membership within a Swarm are **secure out of the box**. SwarmKit uses
  mutual TLS for node authentication, role authorization and transport encryption, automating both
  certificate issuance and rotation.
* SwarmKit is **operationally simple** and minimizes infrastructure dependencies. It does not need
  an external database to operate.
* You can modify a service's configuration, including the networks and volumes it is connected to,
  **without the need to manually restart the service**. Docker will update the configuration, stop
  the service tasks with the out of date configuration, and create new ones matching the desired
  configuration.
* The swarm manager uses **ingress load balancing** to expose the services you want to make
  available externally to the swarm. Once you configure a published port for the service, external
  components, such as cloud load balancers, can access the service on the published port of any node
  in the cluster whether or not the node is currently running the task for the service. All nodes in
  the swarm route ingress connections to a running task instance.
* Swarm mode has an **internal DNS component** that automatically assigns each service in the swarm
  a DNS entry. The swarm manager uses internal load balancing to distribute requests among services
  within the cluster based upon the DNS name of the service.

[Back to top](#readme)

<br>

### Setup

If you have [Vagrant][vagrant] and [VirtualBox][virtualbox] installed, you can easily set up the
infrastructure required for the next steps by running the following command in this repository:

```bash
$> vagrant up
```

Grab a coffee, it will take a while. Once it's done, you should be able to connect to each virtual
machine with the `vagrant ssh <name>` command, for example `vagrant ssh vm1`. You can then skip the
rest of this section and move on to [creating a swarm](#create-a-swarm).

**If you want to set up the infrastructure yourself instead of using Vagrant**, you need 3 virtual
machines running the Ubuntu Xenial operating system:

| Hostname | IP address     |
| :---     | :---           |
| `vm1`    | `192.168.50.4` |
| `vm2`    | `192.168.50.5` |
| `vm3`    | `192.168.50.6` |

You must generate a self-signed certificate for `192.168.50.4` to set up TLS for a Docker registry.
You may do so from one of the virtual machines with the following command:

```bash
openssl req -newkey rsa:4096 -nodes -sha256 -keyout domain.key -x509 -days 365 -out domain.crt -subj "/C=US/ST=New York/L=New York City/O=Acme/OU=IT/CN=example.com" -reqexts SAN -extensions SAN -config <(cat /etc/ssl/openssl.cnf <(printf "\n[SAN]\nsubjectAltName=IP:192.168.50.4"))
```

This command will create a `domain.key` and `domain.crt` file in the current directory.

These are the requirements for all machines:

* Each machine must be able to reach the other two on the following ports: 2376, 2377, 7946 and
  4789.
* [Docker][docker-ce] must be installed and running.
* A copy of the self-signed certificate (the `domain.crt` file) must be stored under
  `/etc/docker/certs.d/192.168.50.4:443/ca.crt`.

Additionally, these are the requirements for the first machine, `vm1`, which will be the Swarm
manager:

* It must be reachable from your test machine (where you are running your browser).
* [Docker Compose][docker-compose-install] must be installed.
* It must have a clone of this repository. In all further examples, it is stored under `/vagrant`,
  as that is where it is automatically mounted when doing `vagrant up`. Adapt the instructions if
  you choose another path.
* The `domain.key` and `domain.crt` files must be under the `tmp` directory in the repository (e.g.
  `/vagrant/tmp`).

[Back to top](#readme)

<br>

### Create a swarm

Connect to `vm1` as the root user. Run the following `docker swarm init` command to enable swarm
mode on the Docker engine and create a swarm:

```bash
root@vm1:~# docker swarm init --advertise-addr 192.168.50.4
Swarm initialized: current node (s9klmcfzhn0j0qww2qjv2hkrs) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-4m7la1jfwvsiycxoeky84ljag8gn3z1f6j15j1tyxgc7t34gfx-5h00wpxkbj6jc6dd4pt9m7mny 192.168.50.4:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
```

As you can see from the output, this new swarm node has been designated a **manager node**, meaning
that it can be used to control the swarm.

You can list the nodes in the swarm with the `docker node ls` command:

```bash
root@vm1:~# docker node ls
ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
s9klmcfzhn0j0qww2qjv2hkrs *   vm1                 Ready               Active              Leader              18.03.0-ce
```

Connect to `vm2` and have it join the swarm with the `docker swarm join` command that was printed
when you created the swarm on `vm1`:

```bash
root@vm2:~# docker swarm join --token SWMTKN-1-4m7la1jfwvsiycxoeky84ljag8gn3z1f6j15j1tyxgc7t34gfx-5h00wpxkbj6jc6dd4pt9m7mny 192.168.50.4:2377
This node joined a swarm as a worker.
```

Connect to `vm3` and do the same:

```bash
root@vm3:~# docker swarm join --token SWMTKN-1-4m7la1jfwvsiycxoeky84ljag8gn3z1f6j15j1tyxgc7t34gfx-5h00wpxkbj6jc6dd4pt9m7mny 192.168.50.4:2377
This node joined a swarm as a worker.
```

Run the `docker node ls` command on `vm1` again (it must be run on a manager node), and you should
see that all 3 nodes are part of the swarm:

```bash
root@vm1:~# docker node ls
ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
s9klmcfzhn0j0qww2qjv2hkrs *   vm1                 Ready               Active              Leader              18.03.0-ce
tfeytcntwewcbk5wcywalxjzd     vm2                 Ready               Active                                  18.03.0-ce
8a3ovnodft8h87na2ov9nophd     vm3                 Ready               Active                                  18.03.0-ce
```

**Labels** can be applied to swarm nodes to help differentiate them when deploying containers.
Let's add some labels to the nodes we created. Imagine the following infrastructure:

* `vm1` is part of the organization's network DMZ and can be reached from the outside, so we'll add
  a `zone` label to it with the value `dmz`. We'll want to deploy the load balancer since it's the
  entrypoint to our infrastructure.
* `vm2` is only accessible from the organization's internal network, so we'll add a `zone` label to
  it with the value `lan`. Additionally, it has high-capacity hard drives optimized for high I/O
  workloads, so we'll also add a `type` label with the `storage` value. We'll want to deploy the
  database there since it's optimized for storage.
* `vm3` is only accessible from the organization's internal network, so we'll also add a `zone`
  label to it with the value `lan`. It has no other special properties.

```bash
root@vm1:~# docker node update --label-add zone=dmz vm1
vm1

root@vm1:~# docker node update --label-add type=storage --label-add zone=lan vm2
vm2

root@vm1:~# docker node update --label-add zone=lan vm3
vm3
```

[Back to top](#readme)

<br>

### Run a private registry

A swarm service can be run from a Docker image, like a container or Docker Compose service, but it
has the additional restriction that **the image must be available in a Docker registry**, such as
the Docker hub.

This is because, until now in this tutorial, we've run containers based on images that were
available locally, either because they'd been pulled from the Docker hub, or because they'd been
built on the machine where we were going to deploy them, and were therefore available locally.

There are multiple nodes in a swarm, i.e. Docker Engines running on separate physical (or virtual)
machines. This means that the images you build on one node is not available to the other nodes.
Therefore, it must be pushed to a registry which is available to all the nodes before it can be
deployed to the swarm.

To run our to-do application, we have 2 options:

* Publish its image to the Docker hub.
* Run a private registry and publish the image there.

We'll use the second solution in this tutorial. Thankfully, it's trivial to run a private registry,
since it's available as an image on the Docker hub. Run the following command on `vm1`:

```bash
root@vm1:~# docker run -d \
  --restart=always \
  --name registry \
  -v /vagrant/tmp:/certs \
  -e REGISTRY_HTTP_ADDR=0.0.0.0:443 \
  -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/domain.crt \
  -e REGISTRY_HTTP_TLS_KEY=/certs/domain.key \
  -p 443:443 \
  registry:2

Unable to find image 'registry:2' locally
2: Pulling from library/registry
81033e7c1d6a: Pull complete
b235084c2315: Pull complete
c692f3a6894b: Pull complete
ba2177f3a70e: Pull complete
a8d793620947: Pull complete
Digest: sha256:672d519d7fd7bbc7a448d17956ebeefe225d5eb27509d8dc5ce67ecb4a0bce54
Status: Downloaded newer image for registry:2
1d8f7aa1b7795d3bdcce623ec174918a132ce9abf659d5199f737453ccd466a8
```

(Note that the mounted volume, `REGISTRY_HTTP_TLS_CERTIFICATE` and `REGISTRY_HTTP_TLS_KEY` options
are used to configure TLS for the registry. Adapt the paths as required if you set up the virtual
machines and generated the certificate yourself instead of using Vagrant.)

Run `docker ps` on `vm1` to check that the registry is running:

```bash
root@vm1:~# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                            NAMES
1d8f7aa1b779        registry:2          "/entrypoint.sh /etc…"   45 seconds ago      Up 44 seconds       0.0.0.0:443->443/tcp, 5000/tcp   registry
```

You now have a private Docker registry accessible at `https://192.168.50.4`!

[Back to top](#readme)

<br>

#### Push an image to a private registry

Which registry a Docker image belongs to is actually defined in the tag:

* `ubuntu` - The official `ubuntu` image on the Docker hub.
* `localhost:5000/ubuntu` - The `ubuntu` image on the Docker registry at `http://localhost:5000`.

By including a host/port combination in the tag, you are specifying the registry where the image can
be pulled from (or pushed to).

Move to the `todo` directory of the repository on `vm1`, and run the following `docker build`
command to build the image. Prepend `192.168.50.4:443` to the tag to indicate that it's part of our
private registry:

```bash
root@vm1:~# cd /vagrant/todo

root@vm1:/vagrant/todo# docker build -f Dockerfile.full -t 192.168.50.4:443/todo .
Sending build context to Docker daemon  69.12kB
Step 1/13 : FROM node:8-alpine
8-alpine: Pulling from library/node
605ce1bd3f31: Pull complete
0511902e1bcd: Pull complete
343e34c41f87: Pull complete
Digest: sha256:d0febbb04c15f58a28888618cf5c3f1d475261e25702741622f375d0a82e050d
Status: Downloaded newer image for node:8-alpine
...
Successfully built 073b4de87319
Successfully tagged 192.168.50.4:443/todo:latest
```

Once that's done, you simply have to run `docker push` with the correct image tag. The image will
automatically be pushed to the correct registry; in this case, the private registry we just
deployed:

```bash
root@vm1:/vagrant/todo# docker push 192.168.50.4:443/todo
The push refers to repository [192.168.50.4:443/todo]
e1b160971701: Pushed
2df32d5d8e82: Pushed
e7238aaba1f3: Pushed
f276f6863123: Pushed
47c6cfbfe051: Pushed
2832e3e1d3a9: Pushed
0f0aa102cc1f: Pushed
9dfa40a0da3b: Pushed
latest: digest: sha256:23996894f90e4b144161e3dc8de07d1025e494691979bb3940ef1fc421ce2c3a size: 1994
```

(Note that the repository is contacted with TLS over the 443 port. If the push fails with "insecure"
errors, make sure the self-signed certificate was correctly generated and configured. See [Test an
Insecure Registry][docker-registry-insecure].)

[Back to top](#readme)

<br>

### Service stacks

This tutorial does not explain how to deploy a single service in a swarm. Read [Getting Started with
Docker Swarm][docker-swarm-getting-started] for that.

Docker Swarm allows you to deploy a **complete application stack** to a swarm. This is very similar
to what we did earlier with Docker Compose, but across multiple swarm nodes. The `docker stack
deploy` command, used to deploy services to a swarm, even accepts a `docker-compose.yml` file
argument to define the stack you want to deploy.

Let's look at the `docker-compose-swarm.yml` file in the `todo` directory. It is similar to
`docker-compose-full.yml` which is the final version we used with Docker Compose, but with a few
changes.

The `build` option of the `app` service has been removed. It is not compatible with `docker stack
deploy` because every image used for services in a swarm must come from a Docker registry.

The `restart` options of all services have been replaced by **`restart_policy` options** under the
`deploy` option. Restart policies for swarm services are more fine-grained than for local Docker
Compose services because they need to be deployed across multiple swarm nodes.

For example, let's take a look at the restart policy of the `lb` service:

```
restart_policy:
  condition: on-failure
  delay: 5s
  window: 15s
```

* `condition: on-failure` indicates that the swarm should only attempt to restart a service's task
  if it stopped due to failure (as opposed to, for example, stopping it manually).
* `delay: 5s` indicates that the swarm should wait 5 second between restart attempts.
* `window: 15s` indicates that after attempting to restart a task, the swarm should wait 15
  seconds to check the tasks and decide whether the restart has succeeded. This option should be
  adjusted based on your service's startup time on your swarm's nodes.

Read the documentation of the [`restart_policy` option][docker-compose-restart-policy] to learn
more.

[Back to top](#readme)

<br>

### Service replication & placement

You can control how many tasks are run on swarm nodes for a service. A service is either:

* **Replicated** (the default): you specify the number of replica tasks the swarm manager(s) should
  schedule on available nodes. 1 replica is deployed by default.
* **Global**: one task should run on every available node.

The **`placement` option** under the `deploy` option has been added to the `docker-compose.yml` file
to specify placement constraints, i.e. rules that limit which swarm nodes tasks will be deployed to.
You will see that the constraints match the requirements we defined when we applied labels to the
swarm nodes:

* We want to deploy the load balancer to a node within the organization's network DMZ
  (`node.labels.zone == dmz`).
* We want to deploy the database to a node optimized for storage (`node.labels.type == storage`).
* We don't care where application containers are deployed.

The **`replicas` option** under the `deploy` option of the `app` service specifies that we want 6
replicas of our application to run in the swarm. The swarm manager(s) will therefore schedule 6
tasks on nodes which match the placement constraints (all of them in this case, since there are no
constraints for the `app` service). The tasks will be spread evenly across available nodes.

[Back to top](#readme)

<br>

### Overlay network

When deploying a stack based on a `docker-compose.yml` file, an [**overlay
network**][docker-network-overlay] will automatically be created for the stack.

It is a network that is distributed among the swarm nodes. This network sits on top of (overlays)
the host-specific networks, allowing containers connected to it (including swarm service containers)
to communicate securely. Docker transparently handles routing of each packet to and from the correct
Docker host and the correct destination container.

![Docker overlay networking](images/overlay.jpg)

Much like the user-created bridge network we used earlier, Docker Swarm provides **automatic DNS
resolution** on the overlay network so that services can communicate with each other simply by using
their name.

Additionally, the overlay network will perform **automatic load balancing** based on the service
name. For example, a request on the `app` host (i.e. the `app` service defined in the stack) will
automatically be routed by the overlay network to one of the replicated tasks of the service.

If you look at the `nginx-swarm.conf` configuration file we'll be using to deploy nginx in the
swarm, it no longer has a hardcoded reference to multiple applications like before. It only refers
to the DNS name of the `app` service, and lets the overlay network do the load balancing.

[Back to top](#readme)

<br>

### Deploy a service stack to a swarm

Simply run the following `docker stack deploy` command on `vm1` to deploy the entire stack for the
to-do application:

```bash
root@vm1:/vagrant/todo# docker stack deploy -c docker-compose-swarm.yml todo
Creating network todo_default
Creating service todo_lb
Creating service todo_app
Creating service todo_db
```

After a few seconds (the swarm manager(s) need a little time to instruct the nodes to run the
correct tasks), you can use the `docker service ls` command to check the status of the various
services deployed as part of the stack:

```bash
root@vm1:/vagrant/todo# docker service ls
ID                  NAME                MODE                REPLICAS            IMAGE                          PORTS
j48resqwpsli        todo_app            replicated          6/6                 192.168.50.4:443/todo:latest
ed8rwb6k3co4        todo_db             replicated          1/1                 mongo:3
l6axuc05ctpb        todo_lb             replicated          1/1                 nginx:1.13-alpine              *:80->80/tcp
```

As instructed, the swarm has deployed:

* 1 replica of our load balancer (since only 1 node matches the constraints).
* 1 replica of our database (since only 1 node matches the constraints).
* 6 replicas of our application.

Check out [`http://192.168.50.4`](http://192.168.50.4) to see the to-do application running.

Reload the page a few times. Note that the hostname cycles over the 6 application container IDs,
showing that the overlay network is indeed load balancing nginx's requests to the various
application containers.

Run `docker ps` on `vm1`:

```bash
root@vm1:/vagrant/todo# docker ps
CONTAINER ID   IMAGE                          COMMAND                  CREATED              STATUS              PORTS                            NAMES
6e5555644b82   nginx:1.13-alpine              "nginx -g 'daemon of…"   About a minute ago   Up About a minute   80/tcp                           todo_lb.1.bsfr4sa8dytgzb55e3gvm315u
6d4243ec2e0f   192.168.50.4:443/todo:latest   "/usr/local/bin/entr…"   About a minute ago   Up About a minute   3000/tcp                         todo_app.2.9685t8c7p9jvjw4nmg0svlb7u
0579018b22c5   192.168.50.4:443/todo:latest   "/usr/local/bin/entr…"   About a minute ago   Up About a minute   3000/tcp                         todo_app.4.br7402xmhw8ysfhjyvlofbhs5
1d8f7aa1b779   registry:2                     "/entrypoint.sh /etc…"   5 minutes ago        Up 5 minutes        0.0.0.0:443->443/tcp, 5000/tcp   registry
```

As expected, we find the load balancer deployed on `vm1` because it is the only node matching the
constraint (the `zone` label must be `dmz`).

Additionally, we can see 2 application containers running on this node. There are only 2 because
since we have not defined any placement constraints for that service, the swarm has used its default
behavior of spreading the service's tasks evenly across the available nodes (2 replicas on each of
the 3 nodes adds up to the 6 replicas we requested).

Run `docker ps` on `vm2` this time:

```bash
root@vm2:~# docker ps
CONTAINER ID   IMAGE                          COMMAND                  CREATED              STATUS              PORTS               NAMES
d385f706d17f   mongo:3                        "docker-entrypoint.s…"   About a minute ago   Up About a minute   27017/tcp           todo_db.1.s2adepvkeu0zvgvr4zv8looid
6baeaf8b3228   192.168.50.4:443/todo:latest   "/usr/local/bin/entr…"   2 minutes ago        Up 2 minutes        3000/tcp            todo_app.1.uczmthswg2xgnlccjpn3cutn0
8b1ab4c0b593   192.168.50.4:443/todo:latest   "/usr/local/bin/entr…"   2 minutes ago        Up 2 minutes        3000/tcp            todo_app.3.adpprnou75oov3e1lbewe7so6
```

You'll find the database deployed on `vm2` since it is the only node matching the constraint (the
`type` label must be `storage`). And you'll find another 2 application containers.

Finally, run `docker ps` on `vm3` this time:

```bash
root@vm3:~# docker ps
CONTAINER ID        IMAGE                          COMMAND                  CREATED             STATUS              PORTS               NAMES
d5eec77a1fa5        192.168.50.4:443/todo:latest   "/usr/local/bin/entr…"   13 seconds ago      Up 12 seconds       3000/tcp            todo_app.6.v2omr5jcm5rieeb7biek03n37
0d417f7b4e6e        192.168.50.4:443/todo:latest   "/usr/local/bin/entr…"   13 seconds ago      Up 11 seconds       3000/tcp            todo_app.5.yonmr2h4dv3g8eaigxww2xy6y
```

You'll find the last 2 remaining application containers. Nothing else has been deployed to this node
due to the requested placement constraints.

Use the `docker service ps <service>` command to more easily list a service's tasks and the nodes on
which they are running:

```bash
root@vm1:/vagrant/todo# docker service ps todo_app
ID                  NAME                IMAGE                          NODE                DESIRED STATE       CURRENT STATE            ERROR               PORTS
uczmthswg2xg        todo_app.1          192.168.50.4:443/todo:latest   vm2                 Running             Running 41 minutes ago
9685t8c7p9jv        todo_app.2          192.168.50.4:443/todo:latest   vm1                 Running             Running 41 minutes ago
adpprnou75oo        todo_app.3          192.168.50.4:443/todo:latest   vm2                 Running             Running 41 minutes ago
br7402xmhw8y        todo_app.4          192.168.50.4:443/todo:latest   vm1                 Running             Running 41 minutes ago
yonmr2h4dv3g        todo_app.5          192.168.50.4:443/todo:latest   vm3                 Running             Running 2 minutes ago
v2omr5jcm5ri        todo_app.6          192.168.50.4:443/todo:latest   vm3                 Running             Running 2 minutes ago
```

[Back to top](#readme)

<br>

### Swarm mode routing mesh

Docker Swarm makes it easy to publish ports for services to make them available to resources outside
the swarm. All nodes participate in an **ingress routing mesh**. The routing mesh enables each node
in the swarm to accept connections on published ports for any service running in the swarm, even if
there's no task running on the node. The routing mesh routes all incoming requests to published
ports on available nodes to an active container.

In our infrastructure, port 80 is published for the `lb` service. With the routing mesh, you can
actually reach the to-do application at any of these adresses:

* [`http://192.168.50.4`](http://192.168.50.4)
* [`http://192.168.50.5`](http://192.168.50.5)
* [`http://192.168.50.6`](http://192.168.50.6)

Even though no nginx container is running on `vm2` or `vm3`, the routing mesh automatically routes
incoming port 80 requests to the `todo_lb.1...` container on `vm1`.

[Back to top](#readme)

---

<br><br><br><br><br>





## Appendices

<br><br><br>



### Squashing image layers

It is considered good practice to minimize the number of layers in a Docker image:

* There will be fewer layers to download (when pulling an image from the Docker hub).
* The build will be faster as larger chunks of the build process will be cached.

There are two things we can do to simply our current Dockerfile.  The first is to make the
`clock.sh` script executable so that we don't need the last `RUN` instruction of the Dockerfile any
more. The `COPY` instruction conserves file permissions:

```bash
$> chmod +x clock.sh
```

The second is to use only 1 `RUN` instruction instead of 3.
Writing a `RUN` instruction like this will only create 1 layer:

```
RUN apt-get update && apt-get install -y cowsay fortune
```

You may use slashes to split the command into multiple lines for readability:

```
RUN apt-get update && \
    apt-get install -y cowsay fortune
```

Your final Dockerfile should look like this:

```
FROM ubuntu

RUN apt-get update && \
    apt-get install -y cowsay fortune

COPY clock.sh /usr/local/bin/clock.sh
```

Run the same build command as before:

```bash
$> docker build -t fortune-clock:3.0 .
Sending build context to Docker daemon  3.072kB
Step 1/3 : FROM ubuntu
 ---> c9d990395902
Step 2/3 : RUN apt-get update &&     apt-get install -y fortune &&     apt-get install -y cowsay
...
 ---> Running in 7a652ff4ab0d
Removing intermediate container 7a652ff4ab0d
 ---> 3ad4e786e5eb
Step 3/3 : COPY clock.sh /usr/local/bin/clock.sh
 ---> 4fa0c71e952c
Successfully built 4fa0c71e952c
Successfully tagged fortune-clock:3.0
```

If you inspect the resulting image, you will see that there are only 2 new layers, one for the `RUN`
instruction, and one for the `COPY` instruction:

```
$> docker inspect fortune-clock:3.0
...
        "RootFS": {
            "Type": "layers",
            "Layers": [
                "sha256:fccbfa2912f0cd6b9d13f91f288f112a2b825f3f758a4443aacb45bfc108cc74",
                "sha256:e1a9a6284d0d24d8194ac84b372619e75cd35a46866b74925b7274c7056561e4",
                "sha256:ac7299292f8b2f710d3b911c6a4e02ae8f06792e39822e097f9c4e9c2672b32d",
                "sha256:a5e66470b2812e91798db36eb103c1f1e135bbe167e4b2ad5ba425b8db98ee8d",
                "sha256:a8de0e025d94b33db3542e1e8ce58829144b30c6cd1fff057eec55b1491933c3",
                "sha256:3395f7dea253615f40e90014d99f91636ac4da015653e93c82c8f3296839d7d9",
                "sha256:fc263405dc2cbb4ae56ba7eabad0b0fdc5b39a64adf7239e907486f460f6a085"
            ]
        },
...
```

Modify the `clock.sh` script again (e.g. add a comment or a new line) and re-run the same build
command:

```bash
$> docker build -t fortune-clock:3.0 .
Sending build context to Docker daemon  3.072kB
Step 1/3 : FROM ubuntu
 ---> c9d990395902
Step 2/3 : RUN apt-get update &&     apt-get install -y fortune &&     apt-get install -y cowsay
 ---> Using cache
 ---> 3ad4e786e5eb
Step 3/3 : COPY clock.sh /usr/local/bin/clock.sh
 ---> 29ac649d902f
Successfully built 29ac649d902f
Successfully tagged fortune-clock:3.0
```

As you can see, the entire `RUN` instruction is cached again, but only 1 layer had to be retrieved
from the cache. Then the `COPY` instruction is executed without the cache, as expected.

[Back to top](#readme)

<br>

#### Using the `--squash` option

Another way to squash layers is to add the `--squash` option to the build command:

```bash
$> docker build -t fortune-clock:3.0 --squash .
Sending build context to Docker daemon  3.072kB
Step 1/3 : FROM ubuntu
 ---> c9d990395902
Step 2/3 : RUN apt-get update &&     apt-get install -y fortune &&     apt-get install -y cowsay
 ---> Using cache
 ---> 3ad4e786e5eb
Step 3/3 : COPY clock.sh /usr/local/bin/clock.sh
 ---> Using cache
 ---> 29ac649d902f
Successfully built e036c40ed4c1
Successfully tagged fortune-clock:3.0
```

If you inspect the resulting image, you will see that there is only 1 additional layer compared to
the base `ubuntu` image (in this case, the one starting with `56286e292`):

```bash
$> docker build -t fortune-clock:3.0 --squash .
...
        "RootFS": {
            "Type": "layers",
            "Layers": [
                "sha256:fccbfa2912f0cd6b9d13f91f288f112a2b825f3f758a4443aacb45bfc108cc74",
                "sha256:e1a9a6284d0d24d8194ac84b372619e75cd35a46866b74925b7274c7056561e4",
                "sha256:ac7299292f8b2f710d3b911c6a4e02ae8f06792e39822e097f9c4e9c2672b32d",
                "sha256:a5e66470b2812e91798db36eb103c1f1e135bbe167e4b2ad5ba425b8db98ee8d",
                "sha256:a8de0e025d94b33db3542e1e8ce58829144b30c6cd1fff057eec55b1491933c3",
                "sha256:56286e2923cafe6b814ae99b8ad2a104be49c870accffa468c17add09f8e2a35"
            ]
        },
...
```

When using the `--squash` option, the new layers are combined into a new image with a single new
layer once the build process is complete. Squashing does not destroy any existing image, rather it
creates a new image with the content of the squashed layers. This effectively makes it look like all
Dockerfile commands were created with a single layer.

Squashing layers can be beneficial if your Dockerfile produces multiple layers modifying the same
files, for example, files that are created in one step, and removed in another step. For other
use cases, squashing images may actually have a negative impact on performance; squashed layers
cannot be reused between images, as they always have a new hash regardless of whether they have the
same content.

Note that the `--squash` option is an experimental feature, and should not be considered stable.

[Back to top](#readme)

---

<br><br><br>



### Dockerfile tips

In the `todo` directory, you will find a `Dockerfile.full` file which is a more complete Dockerfile
than the one used as a first example. Its contents are:

```
FROM node:8-alpine

LABEL maintainer="mei-admin@heig-vd.ch"

ENV NODE_ENV=production \
    PORT=3000

RUN addgroup -S todo && \
    adduser -S -G todo todo && \
    mkdir -p /usr/src/app && \
    chown todo:todo /usr/src/app

USER todo:todo

COPY --chown=todo:todo package.json package-lock.json /usr/src/app/

WORKDIR /usr/src/app
RUN npm install

COPY --chown=todo:todo . /usr/src/app/

EXPOSE 3000

CMD [ "npm", "start" ]
```

Let's see what all of this means.

[Back to top](#readme)

<br>

#### Using smaller base images

Many popular Docker images these days have an Alpine variant. This means that the image is based on
the [official `alpine` image][hub-alpine] on Docker hub, based on the [Alpine Linux][alpine]
distribution. Alpine Linux is much smaller than most distribution base images (~5MB), and thus leads
to much slimmer images in general.

```
FROM node:8-alpine
```

Here we use the `node:8-alpine` tag instead of simply `node:8`.

These variants are highly recommended when final image size being as small as possible is desired.
The main caveat to note is that it does use [musl libc][musl-libc] instead of [glibc and
friends][glibc-etc], so certain software might run into issues depending on the depth of their libc
requirements. However, most software doesn't have an issue with this, so this variant is usually a
very safe choice. See this [Hacker News comment thread][alpine-size] for more discussion of the
issues that might arise and some pro/con comparisons of using Alpine-based images.

To minimize image size, it's uncommon for additional related tools (such as Git or Bash) to be
included in Alpine-based images. Using this image as a base, add the things you need in your own
Dockerfile (see the [alpine image description][hub-alpine] for examples of how to install packages
if you are unfamiliar).

[Back to top](#readme)

<br>

#### Labeling images

Labels are metadata attached to images and containers. They can be used to influence the behavior of
some commands, such as `docker ps`. You can add labels from a Dockerfile with the `LABEL`
instruction.

A popular convention is to add a `maintainer` label to provide a maintenance contact e-mail:

```
LABEL maintainer="mei-admin@heig-vd.ch"
```

You may see the labels of an image or container with `docker inspect`.

You may also filter containers by label. For example, to see all running containers that have the
`foo` label set to the value `bar`, you can use the following command:

```bash
docker ps -f label=foo=bar
```

[Back to top](#readme)

<br>

#### Environment variables

The `ENV` instruction allows you to set environment variables. Many applications change their
behavior in response to some variables. The to-do application example is an [Express][express]
application, so it runs in production mode if the `$NODE_ENV` variable is set to `production`.
Additionally, it listens on the port specified by the `$PORT` variable.

The application should be run in production if you intend to deploy it, and it's good practice to
explicitly set the port rather than relying on the default value, so we set both variables:

```
ENV NODE_ENV=production \
    PORT=3000
```

[Back to top](#readme)

<br>

#### Non-root users

All commands run by a Dockerfile (`RUN` and `CMD` instructions) are **run by the `root` user** of
the container by default. This is not a good idea as any security flaw in your application may give
root access to the entire container to an attacker.

The security impact of this would be mitigated since the container is isolated from the host
machine, but it could still be a **severe security issue** depending on your container's
configuration.

Therefore, it is good practice to create an **unprivileged user** to run your application even in
the container. Here we use Alpine Linux's `addgroup` and `adduser` commands to create a user, and
make sure that the `/usr/src/app` directory where we will copy the application is owned by that
user:

```
RUN addgroup -S todo && \
    adduser -S -G todo todo && \
    mkdir -p /usr/src/app && \
    chown todo:todo /usr/src/app
```

(Note that these commands are specific to Alpine Linux. You would use `groupadd` and `useradd` on
Ubuntu, for example, which use different options.)

Finally, we use the `USER` instruction to make sure that all further commands run in this Dockerfile
(by `RUN` or `CMD` instructions) are executed as the new user instead of the root user:

```
USER todo:todo
```

As you will see, some of the next `COPY` commands in the Dockerfile use the `--chown=todo:todo`
flag. This is because files copied with a `COPY` instruction are always owned by the root user,
regardless of the `USER` instruction, unless the `--chown` (change ownership) flag is used.

[Back to top](#readme)

<br>

#### Speeding up builds

The following pattern is popular to speed up builds of applications that use a package manager (e.g.
npm, RubyGems, Composer).

Installing packages is often one of the slowest command to run for small applications, so we want to
take advantage of Docker's build cache as much as possible to avoid running it every time. Suppose
you did this like in the example Dockerfile of the todo-application:

```
COPY . /usr/src/app/
WORKDIR /usr/src/app
RUN npm install
```

Every time you make the slightest change in any of the application's files, the `COPY` instruction's
cache, and all further commands' caches will be invalidated, including the cache for the `RUN npm
install` instruction. Therefore, any change will trigger a full installation of all dependencies
from scratch.

To improve this behavior, you can split the installation of your application in the container into
two parts. The first part is to copy only the package manager's files (in this case `package.json`
and `package-lock.json`) into the application's directory, and to run an `npm install` command like
before:

```
COPY --chown=todo:todo package.json package-lock.json /usr/src/app/
WORKDIR /usr/src/app
RUN npm install
```

Now, if a change is made to the `package.json` or `package-lock.json` files, the cache of the `RUN
npm install` instruction will be invalidated like before, and the dependencies will be re-installed,
which we want since the change was probably a dependency update. However, changes in any other file
of the application will not invalidate the cache for those 3 instructions, so the result of the `RUN
npm install` instruction will remain cached.

The second part of the installation process is to copy the rest of your application into the
directory:

```
COPY --chown=todo:todo . /usr/src/app/
```

Now, if any file in your application changes, the cache of further instructions will be invalidated,
but since the `RUN npm install` instruction comes before, it will remain in the cache and be skipped
at build time (unless you modify the `package.json` or `package-lock.json` files).

[Back to top](#readme)

<br>

#### Documenting exposed ports

The `EXPOSE` instruction informs Docker that the container listens on the specified network ports at
runtime.

```
EXPOSE 3000
```

The `EXPOSE` instruction does not actually publish the port. It functions as a type of documentation
between the person who builds the image and the person who runs the container, about which ports are
intended to be published. To actually publish the port when running the container, use the `-p`
option on `docker run` to publish and map one or more ports, or the `-P` option to publish all
exposed ports and map them to high-order ports.

[Back to top](#readme)

<br>

#### Using an entrypoint script

(It's recommended that you finish reading the [Docker Compose](#docker-compose) section to better
understand this tip, especially the [Waiting for other containers][dockerfile-tips-waiting]
subsection.)

We've seen that the `CMD` instruction defines the default command to run in the container, but
Docker actually uses 2 instructions to determine the full command to run: `ENTRYPOINT` and `CMD`.

Basically the `ENTRYPOINT` and `CMD` instructions together form the full command. For example, if
your `ENTRYPOINT` is `[ "npm" ]` and your `CMD` is `[ "start" ]`, the full command run in the
container will be `npm start`. The `ENTRYPOINT` is not set by default.

Another characteristic of the `ENTRYPOINT` instruction is that it is **not overriden** when you
specify a different command to `docker run <image> [command...]`. Only the `CMD` instruction is
overriden. To override the entrypoint, you need to add the `--entrypoint` option to the `docker run`
command.

For example, assume that your Dockerfile contains the following instructions:

```
ENTRYPOINT [ "npm" ]
CMD [ "start" ]
```

Here's a few examples of the behavior of `ENTRYPOINT` and `CMD` with various `docker run` options:

* `docker run my-image` - The command run will be `npm start`.
* `docker run my-image test` - The command run will be `npm test`.
* `docker run --entrypoint foo my-image` - The command run will be `foo` (note that the default
  `CMD` is not used when you override the entrypoint).
* `docker run --entrypoint foo my-image bar` - The command run will be `foo bar`

In our case, we use a new `entrypoint.sh` shell script as the entrypoint, because we want to perform
some work before running the actual application (read on to find out why):

```
COPY entrypoint.sh /usr/local/bin/entrypoint.sh
ENTRYPOINT [ "/usr/local/bin/entrypoint.sh" ]
CMD [ "npm", "start" ]
```

The full command executed by our container will therefore be `/usr/local/bin/entrypoint.sh npm start`.

[Back to top](#readme)

<br>

##### Waiting for other containers

The purpose of our entrypoint script is to wait for other containers before actually starting the
application, in this case the database (assumed to be available at the host `db` and port `27017`).

When starting, our Node.js application will exit with an error if the database is not reachable.
This script checks whether a TCP connection can be established at the database's presumed address
every second during 60 seconds. It stops successfully as soon as it can establish a connection, or
fails if it can't (after 60 seconds):

```sh
#!/usr/bin/env sh
command -v nc &>/dev/null || { >&2 echo "This script requires the 'nc' command (from netcat-openbsd)"; exit 2; }
command -v seq &>/dev/null || { >&2 echo "This script requires the 'seq' command (from coreutils)"; exit 2; }

attempts=${ATTEMPTS:-60}
host=${DATABASE_HOST:-"db"}
port=${DATABASE_PORT:-"27017"}

for n in `seq 1 $attempts`; do
  if nc -z -w 1 $host $port; then
    break
  elif test $n -eq $attempts; then
    exit 1
  fi

  echo "Attempting to connect to database on ${host}:${port} ($n)..."
  sleep 1
done

exec "$@"
```

Why do this when you can declare dependencies between containers with Docker Compose and have it
start them in the correct order? Because while Docker Compose will start the containers in the
correct order, it will not wait for the processes inside the containers to finish starting before
moving on to the next container.

In our example, the MongoDB container is "started" as far as Docker is concerned, as soon as Docker
has successfully run the command to start the MongoDB server. But the server may take a few seconds
to initialize before it starts listening on the 27017 port, which Docker doesn't know (the `EXPOSE`
instruction doesn't help in that regard).

As soon as it's "started" the MongoDB container, Docker will attempt to start the Node.js
application container. It's entirely possible that the Node.js application will start faster than
the MongoDB server, and attempt to connect to it before the database is available.

The purpose of this script is to make sure the Node.js container waits for the MongoDB server to be
available before starting the application.

**Warning:** while this pattern is a quick fix that solves one issue, the initial connection to the
database, it does nothing to help if the database connection is lost during your application's
lifetime, after it has started. The best practice would be to change your code to make your
application resilient to connection loss at or after startup, which would solve both the initial
connection and connection loss problems.

[Back to top](#readme)

---

<br><br><br>



### Multi-stage builds

[Squashing image layers](#squashing-image-layers) is one way to reduce the size of your images,
[using multi-stage builds][docker-multi-stage-builds] is another. It's a more recent feature.

One of the most challenging things about building images is **keeping the image size down**. Each
instruction in the Dockerfile adds a layer to the image, and you need to remember to clean up any
artifacts you don’t need before moving on to the next layer. To write a really efficient Dockerfile,
you have traditionally needed to employ shell tricks and other logic to keep the layers as small as
possible and to ensure that each layer has the artifacts it needs from the previous layer and
nothing else.

Take a look at the `multi-stage-builds/hello` directory in this repository. It contains a simple
Hello World [Go][go] script. The `Dockerfile` looks like this:

```
FROM golang:1.10-alpine

LABEL maintainer="mei-admin@heig-vd.ch"

WORKDIR /go/src/app
COPY . /go/src/app
RUN go install

ENTRYPOINT [ "/go/bin/app" ]
CMD [ "World" ]
```

It starts from the [official `golang` image][hub-golang], compiles the script, and sets it to
execute automatically when running a container based on the image.

Build it:

```bash
$> docker build -t hello .
Sending build context to Docker daemon  4.096kB
Step 1/7 : FROM golang:1.10-alpine
1.10-alpine: Pulling from library/golang
ff3a5c916c92: Pull complete
f32d2ea73378: Pull complete
c6678747892c: Pull complete
16b5f22d8b23: Pull complete
Digest: sha256:356aea725be911d52e0f2f0344a17ac3d97c54c74d50b8561f58eae6cc0871bf
Status: Downloaded newer image for golang:1.10-alpine
 ---> 52d894fca6d4
Step 2/7 : LABEL maintainer="mei-admin@heig-vd.ch"
 ---> Running in c41503b15319
Removing intermediate container c41503b15319
 ---> bcd48903ecd3
Step 3/7 : WORKDIR /go/src/app
Removing intermediate container 8830fb0be6d6
 ---> 5ef9446fb4ca
Step 4/7 : COPY . /go/src/app
 ---> e6568af7ab4f
Step 5/7 : RUN go install
 ---> Running in 9b43ccffbe3c
Removing intermediate container 9b43ccffbe3c
 ---> 5700e7915587
Step 6/7 : ENTRYPOINT [ "/go/bin/app" ]
 ---> Running in b1d829be3c3f
Removing intermediate container b1d829be3c3f
 ---> 001c1d6ed7f8
Step 7/7 : CMD [ "World" ]
 ---> Running in db6a78eddded
Removing intermediate container db6a78eddded
 ---> a59edc1b640d
Successfully built a59edc1b640d
Successfully tagged hello:latest
```

Check that it works:

```bash
$> docker run --rm hello
Hello World
$> docker run --rm hello Bob
Hello Bob
```

As you can see by listing the images, the new `hello` image you just built takes up 387MB in size.

```bash
$> docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
hello               latest              a59edc1b640d        15 seconds ago      378MB
golang              1.10-alpine         52d894fca6d4        3 weeks ago         376MB
```

As we've seen before, the 376MB of the `golang` image are actually taken up only once on your disk.
But if you share this image on the Docker hub, and someone who doesn't already have the `golang`
image pulls it, they will have to download 378MB worth of layers.

But that's quite ridiculous considering the Go script itself takes up barely 2MB, as you can check
yourself with the following command. And it's standalone once compiled. It doesn't need the Go
language or compilation tools to run.

```bash
$> docker run --entrypoint ls --rm -it hello -lh /usr/local/bin/hello
-rwxr-xr-x    1 root     root        2.0M May  3 07:42 /usr/local/bin/hello
```

One very common solution to this problem in the past was to have one Dockerfile to use for
development (which contained everything needed to build your application), and a slimmed-down one to
use for production, which only contained your application and exactly what was needed to run it.
This has been referred to as the "builder pattern". Maintaining two Dockerfiles is not ideal.

With **multi-stage builds**, you use **multiple FROM statements in your Dockerfile**. Each FROM
instruction can use a different base, and each of them begins a new stage of the build. You can
selectively copy artifacts from one stage to another, leaving behind everything you don’t want in
the final image.

The `Dockerfile.multistage` file in the same directory uses this feature:

```
FROM golang:1.10-alpine as builder

WORKDIR /go/src/app
COPY . /go/src/app
RUN go install

FROM alpine:3.7

LABEL maintainer="mei-admin@heig-vd.ch"

COPY --from=builder /go/bin/app /usr/local/bin/hello

ENTRYPOINT [ "hello" ]
CMD [ "World" ]
```

The first `FROM golang:1.10-alpine` statement starts a build based on the `golang` image,
and names that **stage of the build** by adding `as builder`, which allows us to refer to it later
(you can also refer to it by its 0-based index, but that's less readable).

As before, this first stage copies the script into the container and compiles it.

The second `FROM alpine:3.7` starts a **new build stage** from the minimal `alpine:3.7` image.

Since the compiled script is already available from the `builder` stage, you only have to copy that
into the new image, which is what the `COPY --from=builder <src> <dest>` instruction does. Instead
of `<src>` referring to a file in the build context, it refers to a file in the `builder` stage.

You can build it as well:

```bash
$> docker build -f Dockerfile.multistage -t hello .
Sending build context to Docker daemon  4.096kB
Step 1/9 : FROM golang:1.10-alpine as builder
 ---> 52d894fca6d4
Step 2/9 : WORKDIR /go/src/app
Removing intermediate container 63c1f5deec54
 ---> f2dc1f94f6f9
Step 3/9 : COPY . /go/src/app
 ---> 773db44160be
Step 4/9 : RUN go install
 ---> Running in 3c3f3cfc0c6b
Removing intermediate container 3c3f3cfc0c6b
 ---> ccce7bbe8b7b
Step 5/9 : FROM alpine:3.7
3.7: Pulling from library/alpine
ff3a5c916c92: Already exists
Digest: sha256:7df6db5aa61ae9480f52f0b3a06a140ab98d427f86d8d5de0bedab9b8df6b1c0
Status: Downloaded newer image for alpine:3.7
 ---> 3fd9065eaf02
Step 6/9 : LABEL maintainer="mei-admin@heig-vd.ch"
 ---> Running in 4973c4c55c4a
Removing intermediate container 4973c4c55c4a
 ---> 45295cf0ab94
Step 7/9 : COPY --from=builder /go/bin/app /usr/local/bin/hello
 ---> a9469b20a365
Step 8/9 : ENTRYPOINT [ "hello" ]
 ---> Running in b689d4bb16cf
Removing intermediate container b689d4bb16cf
 ---> b77644abfb2c
Step 9/9 : CMD [ "World" ]
 ---> Running in b1389e43c798
Removing intermediate container b1389e43c798
 ---> addbf7909f03
Successfully built addbf7909f03
Successfully tagged multi-stage-hello:latest
```

It should work as before:

```bash
$> docker run --rm multi-stage-hello
Hello World
$> docker run --rm multi-stage-hello Bob
Hello Bob
```

But the image now only contains the Alpine Linux base and the compiled script, reducing its total
size from 378MB to 6.29MB:

```bash
$> docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
multi-stage-hello   latest              88e1abecdc0c        1 second ago        6.29MB
<none>              <none>              b32a434ebac2        2 seconds ago       378MB
hello               latest              bb1aafd1c3ea        16 seconds ago      378MB
golang              1.10-alpine         52d894fca6d4        3 weeks ago         376MB
alpine              3.7                 3fd9065eaf02        3 months ago        4.15MB
```

Another example where you may apply this pattern is a client-side application using a framework with
a build chain, such as Angular or React. You only need the build tools to produce a production
build, i.e. the final HTML, JavaScript and CSS files that will be served to the browser. Once you
have that, you don't need the build tools any more.

You can build your application in the first stage, then copy only the HTML, JavaScript and CSS
products to the final stage.

[Back to top](#readme)

---

<br><br><br><br><br>





## TODO

* run multiple commands in a container: add `docker ps` example of running container
* docker daemon
* docker swarm secrets
* dockerfile inheritance (all instructions, entrypoint, cmd)
* unix process signals
* appendix: init process
* appendix: multi-stage builds, react example
* appendix: multi-process container (s6)
* appendix: developing with Docker

[Back to top](#readme)

---

<br><br><br><br><br>





## References

* [What is Docker?][what-is-docker]
* [What is a Container?][what-is-container]
* [Docker Security][docker-security]
* [Dockerfile Reference][dockerfile]
  * [Best Practices for Writing Dockerfiles][dockerfile-best-practices]
  * [Trapping Signals in Docker Containers][docker-signals]
* [Docker Networking Overview][docker-networking]
  * [Docker Bridge Networks][docker-bridge-networks]
  * [Docker Networking Deep Dive (SlideShare)][docker-networking-deep-dive]
* [Manage Data in Docker][docker-storage]
  * [Docker Storage Drivers][docker-storage-drivers]
  * [Use Volumes][docker-storage-volumes]
  * [Use Bind Mounts][docker-storage-bind]
  * [Use tmpfs Mounts][docker-storage-tmpfs]
* [Docker Compose Overview][docker-compose]
  * [Docker Compose Installation][docker-compose-install]
  * [Docker Compose File Reference][docker-compose-file]
  * [Overview of Docker Compose CLI][docker-compose-cli]
* [Starting containers automatically][docker-restart-policy]
* [The Twelve-Factor App][12factor]
* [Deploy a Registry Server][docker-registry]
* [Docker Swarm][docker-swarm]
  * [SwarmKit][swarmkit]
  * [Docker Swarm Key Concepts][docker-swarm-concepts]
  * [Getting Started with Swarm Mode][docker-swarm-getting-started]

[Back to top](#readme)





[12factor]: https://12factor.net
[12factor-processes]: https://12factor.net/processes
[alpine]: https://alpinelinux.org
[alpine-size]: https://news.ycombinator.com/item?id=10782897
[bash]: https://en.wikipedia.org/wiki/Bash_(Unix_shell)
[consul]: https://www.consul.io
[cow]: https://en.wikipedia.org/wiki/Copy-on-write
[dmz]: https://en.wikipedia.org/wiki/DMZ_(computing)
[docker-bridge-networks]: https://docs.docker.com/network/bridge/
[docker-ce]: https://www.docker.com/community-edition
[docker-compose]: https://docs.docker.com/compose/overview/
[docker-compose-cli]: https://docs.docker.com/compose/reference/overview/
[docker-compose-file]: https://docs.docker.com/compose/compose-file/
[docker-compose-install]: https://docs.docker.com/compose/install/
[docker-compose-restart-policy]: https://docs.docker.com/compose/compose-file/#restart_policy
[docker-ignore]: https://docs.docker.com/engine/reference/builder/#dockerignore-file
[docker-multi-stage-builds]: https://docs.docker.com/develop/develop-images/multistage-build/
[docker-network-overlay]: https://docs.docker.com/network/overlay/
[docker-networking]: https://docs.docker.com/network/
[docker-networking-deep-dive]: https://www.slideshare.net/MadhuVenugopal2/dcus17-docker-networking-deep-dive
[docker-restart-policy]: https://docs.docker.com/config/containers/start-containers-automatically/
[docker-registry]: https://docs.docker.com/registry/deploying/
[docker-registry-insecure]: https://docs.docker.com/registry/insecure/#use-self-signed-certificates
[docker-security]: https://docs.docker.com/engine/security/security/
[docker-signals]: https://medium.com/@gchudnov/trapping-signals-in-docker-containers-7a57fdda7d86
[docker-storage]: https://docs.docker.com/storage/
[docker-storage-bind]: https://docs.docker.com/storage/bind-mounts/
[docker-storage-drivers]: https://docs.docker.com/storage/storagedriver/
[docker-storage-tmpfs]: https://docs.docker.com/storage/tmpfs/
[docker-storage-volume-drivers]: https://docs.docker.com/storage/volumes/#use-a-volume-driver
[docker-storage-volumes]: https://docs.docker.com/storage/volumes/
[docker-swarm]: https://docs.docker.com/engine/swarm/
[docker-swarm-concepts]: https://docs.docker.com/engine/swarm/key-concepts/
[docker-swarm-getting-started]: https://docs.docker.com/engine/swarm/swarm-tutorial/
[docker-swarm-placement]: https://docs.docker.com/engine/swarm/services/#control-service-placement
[docker-swarm-routing-mesh]: https://docs.docker.com/engine/swarm/ingress/
[dockerfile]: https://docs.docker.com/engine/reference/builder/
[dockerfile-best-practices]: https://docs.docker.com/develop/develop-images/dockerfile_best-practices/
[dockerfile-tips]: #dockerfile-tips
[dockerfile-tips-waiting]: #waiting-for-other-containers
[express]: http://expressjs.com
[fortune]: https://en.wikipedia.org/wiki/Fortune_(Unix)
[git-bash]: https://git-scm.com/downloads
[glibc-etc]: http://www.etalabs.net/compare_libcs.html
[go]: https://golang.org
[horizontal-scaling]: https://en.wikipedia.org/wiki/Scalability#Horizontal_and_vertical_scaling
[hub]: https://hub.docker.com
[hub-alpine]: https://hub.docker.com/_/alpine/
[hub-explore]: https://hub.docker.com/explore/
[hub-golang]: https://hub.docker.com/_/golang/
[hub-mongo]: https://hub.docker.com/_/mongo/
[hub-mongo-dockerfile]: https://github.com/docker-library/mongo/blob/dd8ceb3b3552d11c901a603d0b8b303e2fe4bc2e/3.6/Dockerfile
[hub-nginx]: https://hub.docker.com/_/nginx/
[hub-node]: https://hub.docker.com/_/node/
[hub-ubuntu]: https://hub.docker.com/_/ubuntu/
[lxc]: https://linuxcontainers.org
[mongo]: https://www.mongodb.com
[mongo-replication]: https://docs.mongodb.com/manual/replication/
[musl-libc]: http://www.musl-libc.org
[nginx]: https://www.nginx.com
[nginx-lb]: http://nginx.org/en/docs/http/load_balancing.html
[node]: https://nodejs.org/en/
[open-container-initiative]: https://www.opencontainers.org
[raft]: https://raft.github.io
[serf]: https://www.serf.io
[service-discovery]: https://en.wikipedia.org/wiki/Service_discovery
[squashing-layers]: #squashing-image-layers
[swarmkit]: https://github.com/docker/swarmkit
[ubuntu]: https://www.ubuntu.com
[union-fs]: https://en.wikipedia.org/wiki/UnionFS
[vagrant]: https://www.vagrantup.com
[virtualbox]: https://www.virtualbox.org
[what-is-container]: https://www.docker.com/what-container
[what-is-docker]: https://www.docker.com/what-docker
[wsl]: https://docs.microsoft.com/en-us/windows/wsl/install-win10
[yaml]: http://yaml.org
[zookeeper]: https://zookeeper.apache.org
