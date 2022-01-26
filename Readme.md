# Learning Docker

Much of the notes below are stolen verbatim from [Engineer Man's video about
docker](https://www.youtube.com/watch?v=6aBsjT5HoGY). He deserves all credit.
I'm just keeping this as an online notepad.

## Docker Concepts Introduction

What is Docker? Docker is a container management system. Containers are
isolated environments for running software. Containers are somewhat like virtual
machines in that they have their own file system and process tree. However,
they are unlike virtual machine's in that they share the same kernel as the
host operating system.

The inert form of a docker container is called a docker image. A container is
just an actively running instance of an image. There are docker images out
there for everything from [`Redis`](https://redis.io/) to
[`Xeyes`](https://linux.die.net/man/1/xeyes)`. Docker images can be downloaded
through an [official
repository](https://docs.docker.com/docker-hub/official_images/).

You can also create / modify docker images yourself to containerize your own
custom software.

## Installing Docker On Pair Servers

TBD

## Docker Tools

### Docker Run

To start a docker container from an image you use the `docker run` command. If
you don't have the specified container already `docker run` will download it
automatically from its central repository.

Below are three example python 3 docker run commands. 

* Run a script:

```
docker run \
	--rm \
	-v $(pwd):/src \
	python:3 \
	python /src/hello.py
```

Lets go over the above options on that docker run command. 

The first flag, `--rm` tells docker to remove the container on exit. 

The next option `-v` is the volume flag. This mounts the specified directory on
the host system to the internal file system of the container. 

In the above case we're mounting the present working directory to /src inside of
the container. 

The next option `python:3` is the image name. This image will be pulled from
the central repo on initial startup if we don't already have it. 

Finally, `python /src/hello.py` is the command that will be run within the
container.

* Output:

```
Unable to find image 'python:3' locally
3: Pulling from library/python
0e29546d541c: Pull complete 
9b829c73b52b: Pull complete 
cb5b7ae36172: Pull complete 
6494e4811622: Pull complete 
6f9f74896dfa: Pull complete 
fcb6d5f7c986: Pull complete 
3445db4c939c: Pull complete 
def920d3ef5d: Pull complete 
aabf25a1ee4b: Pull complete 
Digest: sha256:c75f8d97a16f5f3c785496fe25a96d65be273c8b58ff10bc40ce9197b61f7a53
Status: Downloaded newer image for python:3
Hello World!
```

And if you run it a second time after fetching the python 3 image it just says
the following.

```
Blue@Home:$Learning_Docker> docker run --rm -v $(pwd):/src python:3 python /src/hello.py
Hello World!
```

* Run the python interpreter:

```
docker run \
	--rm \
	-it \
	-v $(pwd):/src \
	python:3 \
	python
```

To run the raw python 3 interpreter itself inside of the docker container all we
need to do differently is add the `-it` flag for interactive session.

Then when we run the container we get an interactive python 3 shell.

* Output:

```
Blue@Home:$Learning_Docker> docker run --rm -it -v $(pwd):/src python:3 python 
Python 3.10.2 (main, Jan 18 2022, 19:45:14) [GCC 10.2.1 20210110] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> import os
>>> for k, v in os.environ.items():
...     print(f'{k}={v}')
... 
PATH=/usr/local/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=fac87d172320
TERM=xterm
LANG=C.UTF-8
GPG_KEY=A035C8C19219BA821ECEA86B64E628F8D684696D
PYTHON_VERSION=3.10.2
PYTHON_PIP_VERSION=21.2.4
PYTHON_SETUPTOOLS_VERSION=57.5.0
PYTHON_GET_PIP_URL=https://github.com/pypa/get-pip/raw/3cb8888cc2869620f57d5d2da64da38f516078c7/public/get-pip.py
PYTHON_GET_PIP_SHA256=c518250e91a70d7b20cceb15272209a4ded2a0c263ae5776f129e0d9b5674309
HOME=/root
```

* Run a normal bash shell:

```
docker run \
	--rm \
	-it \
	-v $(pwd):/src \
	python:3 \
	/bin/bash
```

As you can see the above command is very similar to the python interpreter
command except instead of running python it just runs a bash shell inside of
the python 3 container.

* Output:

```
Blue@Home:$Learning_Docker> docker run --rm -it -v $(pwd):/src python:3 bash
root@b80375c6272e:/# id
uid=0(root) gid=0(root) groups=0(root)
root@b80375c6272e:/# pwd
/
root@b80375c6272e:/# ls /src/
Readme.md  hello.py
root@b80375c6272e:/# printenv
HOSTNAME=b80375c6272e
PYTHON_VERSION=3.10.2
PWD=/
PYTHON_SETUPTOOLS_VERSION=57.5.0
HOME=/root
LANG=C.UTF-8
GPG_KEY=A035C8C19219BA821ECEA86B64E628F8D684696D
TERM=xterm
SHLVL=1
PYTHON_PIP_VERSION=21.2.4
PYTHON_GET_PIP_SHA256=c518250e91a70d7b20cceb15272209a4ded2a0c263ae5776f129e0d9b5674309
PYTHON_GET_PIP_URL=https://github.com/pypa/get-pip/raw/3cb8888cc2869620f57d5d2da64da38f516078c7/public/get-pip.py
PATH=/usr/local/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
_=/usr/bin/printenv
```

### Docker Build 

Let's say we want to modify the default python 3 base image slightly and make a
new image that includes a bunch of python modules like `numpy` or `requests`.
We'll we can do so by utilizing the `docker build` command and a `Dockerfile`.

`Dockerfile`

```
FROM python:3

RUN pip3 install numpy
```

* Build Dockerfile with docker build:

```
docker build \
	-t python_numpy \
	.
```

The `-t` option is to specify the image tag (name) and the `.` says build an
image from a Dockerfile in the present working directory.

* Output:

```
Blue@Home:$Learning_Docker> docker build \ 
>     -t python_numpy \
>     .
Sending build context to Docker daemon  26.11kB
Step 1/2 : FROM python:3
 ---> cecf555903c6
Step 2/2 : RUN pip3 install numpy
 ---> Running in 542b9ae1e7b6
Collecting numpy
  Downloading numpy-1.22.1-cp310-cp310-manylinux_2_17_x86_64.manylinux2014_x86_64.whl (16.8 MB)
Installing collected packages: numpy
Successfully installed numpy-1.22.1
Removing intermediate container 542b9ae1e7b6
 ---> db2eacc55052
Successfully built db2eacc55052
Successfully tagged python_numpy:latest

```

As you can see the above spins up a python 3 container then runs the `pip3
install numpy` command and then exports the container to a new python_numpy
image.

If we list the images with `docker image ls` we can see our newly created
image.

```
REPOSITORY               TAG                IMAGE ID       CREATED         SIZE
python_numpy             latest             db2eacc55052   2 minutes ago   1.01GB
```

And now we can spin up a container of this image.

```
docker run \
	--rm \
	-v $(pwd):/src \
	python_numpy \
	python /src/numpy_quicksort.py
```

* Output:

```
[1, 1, 2, 3, 6, 8, 10]
```

## Running a Daemonized Container

So far we've seen simple containers that run a script and then close and
destroy themselves. Lets look at a more complicated container for `Nginx`.

* Start container in foreground:

```
docker run \
	--rm \
	-v $(pwd):/usr/share/nginx/html \
	-p 8080:80 \
	nginx:latest
```

That will start an Nginx server running in the foreground and any requests are
logged to the terminal. If you make a request to localhost:8080 you'll see that
port is forwarded to docker container 80.

* Output: 

```
172.17.0.1 - - [26/Jan/2022:16:32:10 +0000] "GET / HTTP/1.1" 200 242 "-" "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/97.0.4692.99 Safari/537.36" "-"
```

* Start container in the background:

```
docker run \
	--rm \
	-d \
	-v $(pwd):/usr/share/nginx/html \
	-p 8080:80 \
	nginx:latest
```

The only real difference is the `-d` switch which starts the container in
'daemon mode.'

We can see the container running in the background with the `docker container
ls` command.

```
CONTAINER ID   IMAGE          COMMAND                  CREATED         STATUS         PORTS                                   NAMES
75057a6ee859   nginx:latest   "/docker-entrypoint.…"   9 seconds ago   Up 8 seconds   0.0.0.0:8080->80/tcp, :::8080->80/tcp   peaceful_chebyshev
```

To log into a running container with a shell simply run the following.

```
docker exec -it peaceful_chebyshev /bin/bash
```

Where "peaceful_chebyshev" is whatever name docker has given your container.


## Networking Containers

Docker networks allow containers to communicate between each other.

To create a new docker network run the command below.

```
docker network create blahfart
```

We can list the networks with.

```
docker network ls
```

Docker will also automatically assign DNS entries for the hosts on the docker
network so the can be communicated between using hostnames.

We'll start up two docker containers to see the networking between them.

* Host 1:

```
docker run \
	--rm \
	-d \
	--net blahfart \
	--name host1 \
	-v $(pwd):/usr/share/nginx/html \
	-p 8080:80 \
	nginx:latest
```

* Host 2:

```
docker run \
	--rm \
	-d \
	--net blahfart \
	--name host2 \
	-v $(pwd):/usr/share/nginx/html \
	-p 8081:80 \
	nginx:latest
```

Then we can connect to the two hosts and send messages back and forth.

```
docker exec -it host1 /bin/bash
```

First install ping inside of the container.

```
apt update && apt install iputils-ping -y
```

And then you can use it to ping the other host over the docker network.

```
root@358d6830cd2a:/# ping host2
PING host2 (172.18.0.3) 56(84) bytes of data.
64 bytes from host2.blahfart (172.18.0.3): icmp_seq=1 ttl=64 time=0.093 ms
64 bytes from host2.blahfart (172.18.0.3): icmp_seq=2 ttl=64 time=0.064 ms
64 bytes from host2.blahfart (172.18.0.3): icmp_seq=3 ttl=64 time=0.069 ms
```

Finally to kill our containers run,

```
docker container stop host1 host2
```

## More to Come...

More to come in next section of notes about `docker compose`.
