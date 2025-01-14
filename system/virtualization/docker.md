# Docker

Docker implements a container solution. Containers are a lighter weight alternative to a full virtual machine. They are run on the host operating system, but they are encapsulated to provide isolation, security, and compartmentalization. 

Docker documentation:  
https://docs.docker.com/

Overview:  
https://docs.docker.com/engine/understanding-docker/

https://docs.docker.com/engine/userguide/

Cheat sheet with an overview (similar intent as this doc)  
https://github.com/wsargent/docker-cheat-sheet

Introduction to what containers are:  
https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-18-04

## See Also

[Kubernetes](kubernetes.md)

[Docker Compose](docker-compose.md)

[Orchestration](orchestration.md)

https://gitlab.com/fern-seed/web-ui-api-db/

https://gitlab.com/fern-seed/web-ui-api-db/-/blob/main/README-docker.md

https://github.com/veggiemonk/awesome-docker  
GitHub - veggiemonk/awesome-docker: A curated list of Docker resources and projects  


## Dockerfile

How you set up the image that gets run in a container

https://docs.docker.com/engine/reference/builder/

A Dockerfile determines how your image is configured (and ultimately what is run in your container). These can be tracked as part of the project's source code. 

https://docs.docker.com/get-started/part2/

to keep a container running, choose a process that won't exit:

```
CMD [ "tail", "-f", "/dev/null" ]
```

See also [dockerfiles](dockerfiles.md)

```
FROM node:lts

# Set the working directory.
WORKDIR /srv
```


## Installation

https://docs.docker.com/engine/install/ubuntu/

Just follow along. These stay more up to date


https://docs.docker.com/engine/install/


At this point Docker should be installed and you can verify with:

```
sudo systemctl status docker
```


### Docker Compose

Go ahead and grab docker-compose

```
sudo apt-get install docker-compose -y
```


### Rootless & Permissions

Visit https://docs.docker.com/go/rootless/ to learn about rootless mode.

Seems necessary to still install docker as usual above. 


```
sudo apt-get install -y uidmap
```

With Ubuntu, I already had:

```
sudo apt-get install -y dbus-user-session
```

If the system-wide Docker daemon is already running, consider disabling it: 

```
sudo systemctl disable --now docker.service docker.socket
```

To run Docker as a non-privileged user, consider setting up the
Docker daemon in rootless mode for your user:

```
dockerd-rootless-setuptool.sh install
```

Automatically start up when user logs in:

```
systemctl --user enable docker
sudo loginctl enable-linger $(whoami)
```

Make sure the following environment variables are set by adding them to `~/.bashrc`

```
export PATH=/usr/bin:$PATH
export DOCKER_HOST=unix://$XDG_RUNTIME_DIR/docker.sock

```

To expose privileged ports (< 1024), 

```
sudo micro /etc/sysctl.conf 
```

Add the line: 
```
net.ipv4.ip_unprivileged_port_start=443
```

Then apply changes to the system
```
sudo sysctl -p
```

Alternatively, set CAP_NET_BIND_SERVICE on rootlesskit binary and restart the daemon.

```
sudo setcap cap_net_bind_service=ep $(which rootlesskit)
systemctl --user restart docker
```



### Add user to docker group

This allows you to execute docker without using `sudo` for every command.  
`docker-compose` is a frequent command. 

```
sudo groupadd docker

sudo usermod -aG docker ${USER}
```

Log out and log back in, or:

```
su - ${USER}
```
    
Test that you have permissions to run docker commands without sudo:

```
docker ps
```

To run the Docker daemon as a fully privileged service, but granting non-root
users access, refer to https://docs.docker.com/go/daemon-access/

WARNING: Access to the remote API on a privileged Docker daemon is equivalent
         to root access on the host. Refer to the 'Docker daemon attack surface'
         documentation for details: https://docs.docker.com/go/attack-surface/




## Docker Desktop for Linux

Looking forward to giving this a try. KVM/QEMU -- sounds promising!

```
sudo usermod -aG kvm $USER
```

Dependencies:

Install docker as usual -- ends up expecting `docker-ce-cli` package anyway

Install KVM/QEMU ahead of time

~/public/system/virtualization/kvm.md

```
sudo apt-get install pass tree
```

download package for system

https://docs.docker.com/desktop/linux/install/

```
sudo dpkg -i docker-desktop-4.8.0-amd64.deb 
```



## Status

```
systemctl status docker
```

To see a list of *currently running* docker containers:

```
docker ps
```

To see the name of the container (and the size of disk in use):

```
docker ps -s 
```

https://docs.docker.com/engine/reference/commandline/ps/


























































## Shares & Storage

It is possible to share storage between the host and containers. For a general overview:

https://docs.docker.com/storage/

Bind Mounts are shares data between the container and the host:

https://docs.docker.com/storage/bind-mounts/

Volumes are encapsulated in the container engine itself (managed separately from the host):

https://docs.docker.com/storage/volumes/

For development, a bind mount may work well. For deployments, a volume is a better choice. 

These can be specified when running a container, or as part of a compose setup:

```
docker run -d \
  -it \
  --name devtest \
  --mount type=bind,src="$(pwd)"/target,dst=/app \
  nginx:latest
```

### Copy Data

You can copy data into and out of a container manually:

// don't use `*` with `docker cp`
docker cp e9f10889f0da:/synthea/output/fhir ui/public/data/

https://docs.docker.com/engine/reference/commandline/cp/  
docker cp | Docker Documentation  



## Images

See a list of available docker images (see what is currently available):

```
docker image ls -a
```

Is equivalent to:

```
docker images
```

docker images are stored in:

```
/var/lib/docker
```

(don't forget, docker sometimes run in a VM, so this location is on that VM)
via:
http://stackoverflow.com/questions/19234831/where-are-docker-images-stored-on-the-host-machine

### Official Images

https://hub.docker.com/search?q=&type=image&image_filter=official

Different dependencies will result in different sized containers. Smaller is generally better, everything else being the same:

https://www.brianchristner.io/docker-image-base-os-size-comparison/

https://docs.docker.com/docker-hub/official_images/


### Building

    docker build -t simple-node .
    docker run -p 3000:3000 simple-node

Now you should be able to connect to localhost without specifying a VM host. Without the explicit forward for the port, the port won't be available:

    http://localhost:3000/
    
To specify a Dockerfile, use -f:

    docker build -t simple-node -f Dockerfile.debug .
    
https://docs.docker.com/engine/reference/commandline/build/

## Running a Container

When you 'run' a command with docker, you specify the docker image to use to run it. The run command will download the image, build the container (if it doesn't exist already), and then run the command in the container.

    docker run mhart/alpine-node node --version
    
Even with single container setups, it may make sense to use docker-compose to specify what the container is named and any volumes that should be mounted. That also makes it easier to integrate with other docker-compose setups. 
    
### Connecting to a Container 

start and connect to a docker container:

    docker run -i -t --entrypoint /bin/bash <imageID>
    docker run -i -t --entrypoint /bin/bash docker_web_run_1

start a new shell in an already running container:

    docker exec -it <containerIdOrName> bash
    docker exec -it 393b12a61839 /bin/sh
    docker exec -it docker_web_run_1 bash

connect to a (already running) docker container (Note: this will share the same shell if another instance is already connected interactively)

    docker attach loving_heisenberg 

via:
http://askubuntu.com/questions/505506/how-to-get-bash-or-ssh-into-a-running-container-in-background-mode

### Stopping a Container

    docker container stop devtest

To stop everything:

    docker stop $(docker ps -q)
    
or 

    docker container stop $(docker container list -q)

### Removing a Container

    docker rm [container]

### Cleaning up old images

```
docker system prune 
```

This seems like a well maintained answer with up-to-date options & descriptions:

https://stackoverflow.com/questions/32723111/how-to-remove-old-and-unused-docker-images

[via](https://duckduckgo.com/?t=ffab&q=docker-compose+remove+old+images&ia=web)

Some of the following options may be more aggressive in what they delete. Be careful if you have important data stored!

Sometimes when testing builds, docker will complain about running out of space:

```
Thin Pool has 2738 free data blocks which is less than minimum required 2915 free data blocks. Create more free space in thin pool or use dm.min_free_space option to change behavior
```

You can get rid of all images with:

    docker image prune -a --force 

https://stackoverflow.com/questions/41531962/docker-run-error-thin-pool-has-free-data-blocks-which-is-less-than-minimum-req

Clear everything out (!!! dangerous !!!)

    docker image rm $(docker image ls -a -q)
    docker image rm -f $(docker image ls -a -q)

See also:
https://docs.docker.com/engine/reference/commandline/image_prune/

Still not enough space?
There are some nuclear options outlined here -- they will clear everything out, including volumes that may have data on them!

https://stackoverflow.com/questions/36918387/space-issue-on-docker-devmapper-and-centos7

### Restarting

Containers can be set to restart automatically. As long as the parent docker process is configured to run at start up (usually is by default), then those containers will restart automatically. 

You could also do a systemctrl setup like:

`sudo systemctl enable docker-MYPROJECT-oracle_db.service`

As described in

https://stackoverflow.com/questions/30449313/how-do-i-make-a-docker-container-start-automatically-on-system-boot
How do I make a Docker container start automatically on system boot? - Stack Overflow

https://docs.docker.com/config/containers/start-containers-automatically/
Start containers automatically | Docker Documentation

https://duckduckgo.com/?q=docker+start+container+at+boot&t=ffab&ia=web
docker start container at boot at DuckDuckGo



## Networking

See all networks currently configured:

```
docker network ls
```

See details for a specific network:

```
docker network inspect bridge
```

Docker containers can be referenced from other containers using the container name. Be sure to use the full container name, not the abbreviated service name that is used in docker-compose files. 

`ping` is not always available. On debian based containers, install it with:

```
apt-get update
apt-get install iputils-ping
```

From there, can testing pinging containers by name:

```
ping nginx
```

See a list of all IP addresses for all containers:

```
sudo docker ps | tail -n +2 | while read cid b; do echo -n "$cid\t"; sudo docker inspect $cid | grep IPAddress | cut -d \" -f 4; done
```

via:
http://stackoverflow.com/questions/17157721/getting-a-docker-containers-ip-address-from-the-host


wasn't sure about how to get one container to talk to another...
they've documented that well:

https://docs.docker.com/engine/userguide/containers/networkingcontainers/

(on macs) set up a terminal to know how to interact with docker by running:

    eval "$(docker-machine env default)"


way to generalize reference to containers in configuration files?
what if the IP for the api server changes?
would require manually updating nginx.conf file
just use docker name

### DNS

I ran into an issue where a container was not able to resolve DNS lookups. (to confirm this, connect to the container via bash and run `ping google.com`)

This turned out to be an issue with the way lookups are configured on my host machine (20.04). 

Docker uses the host's name resolution. Running this on the host fixes the resolution within containers: 

    sudo ln -sf /run/systemd/resolve/resolv.conf /etc/resolv.conf

The link that helped:

https://serverfault.com/questions/642981/docker-containers-cant-resolve-dns-on-ubuntu-14-04-desktop-host

Note:
If you're using hosts defined in `/etc/hosts`, the updated symlink won't resolve those correctly. See also: ~/alpha/web_ui_api_db/local-development.md


### Troubleshooting connections in docker

https://www.docker.com/blog/why-you-dont-need-to-run-sshd-in-docker

A successful approach was to launch the server, connect to the container using another shell

    apk update
    
this provides the "ss" command for "socket statistics"
RUN apk add --no-cache iproute2
e.g. to see if a server is running on expected port:

    ss -lntp

verify server was on correct ports using above command

    apk add lynx

lynx 127.0.0.1:8080

curl is another good option!

To see what ports are open, install `netstat`. This approach will not persist across restarts. 


    apt update
    apt install net-tools

    netstat -pan | egrep " LISTEN "
    

### Restart docker

I was getting an error like:

```
ERROR: for ui  Cannot start service ui: driver failed programming external connectivity on endpoint gpdb_ui (4625d0f53435c85529b8a8d6317401d6aba192fb2adbe742b4e98c828cb76125): Error starting userland proxy: error while calling PortManager.AddPort(): cannot expose privileged port 443, you can add 'net.ipv4.ip_unprivileged_port_start=443' to /etc/sysctl.conf (currently 1024), or set CAP_NET_BIND_SERVICE on rootlesskit binary, or choose a larger port number (>= 1024): listen tcp4 127.0.0.1:443: bind: permission denied
ERROR: Encountered errors while bringing up the project.
```

It didn't look like anything was listening on port 443. 

I think this is an issue with running in `rootlesskit` mode

https://docs.docker.com/engine/security/rootless/#exposing-privileged-ports

TODO: How-to determine which version of docker is being run locally

```
systemctl --user status docker
```

```
sudo service docker stop
sudo rm /var/lib/docker/network/files/local-kv.db
sudo service docker start && service docker status
```

`service` -- that's cool, appears to be the same thing as `sysctl` in other distros. (or is it `systemctl` or `systemcontrol`? I forget.)

I was curious, what is actually in this file that you are asking me to delete as sudo? Looks like `local-kv.db` is where the networking configurations are stored

https://blog.qiqitori.com/2018/10/decoding-dockers-local-kv-db/  
Decoding Docker’s local-kv.db – The Qiqitori Blogs  



https://duckduckgo.com/?q=Cannot+start+service+driver+failed+programming+external+connectivity+on+endpoint+Error+starting+userland+proxy%3A+error+while+calling+PortManager.AddPort()%3A+cannot+expose+privileged+port+443%2C+you+can+add+%27net.ipv4.ip_unprivileged_port_start%3D443%27+to+%2Fetc%2Fsysctl.conf+(currently+1024)%2C+or+set+CAP_NET_BIND_SERVICE+on+rootlesskit+binary%2C+or+choose+a+larger+port+number+(%3E%3D+1024)%3A+listen+tcp4+127.0.0.1%3A443%3A+bind%3A+permission+denied&hps=1&atb=v343-1&ia=web  
Cannot start service driver failed programming external connectivity on endpoint Error starting userland proxy: error while calling PortManager.AddPort(): cannot expose privileged port 443, you can add 'net.ipv4.ip_unprivileged_port_start=443' to /etc/sysctl.conf (currently 1024), or set CAP_NET_BIND_SERVICE on rootlesskit binary, or choose a larger port number (>= 1024): listen tcp4 127.0.0.1:443: bind: permission denied at DuckDuckGo  
https://stackoverflow.com/questions/39508018/docker-driver-failed-programming-external-connectivity-on-endpoint-webserver  
docker: driver failed programming external connectivity on endpoint webserver - Stack Overflow  

https://duckduckgo.com/?t=ffcm&q=%2Fvar%2Flib%2Fdocker%2Fnetwork%2Ffiles%2Flocal-kv.db&atb=v343-1&ia=web  
/var/lib/docker/network/files/local-kv.db at DuckDuckGo  



## Context Specific Applications

### Node

See [Node Notes](../../code/javascript/node.md)

### Python

Example Dockerfile

```
FROM python:3
COPY requirements.txt /srv/app/requirements.txt
WORKDIR /srv/app
RUN pip install -r requirements.txt
```

https://hub.docker.com/_/python/

https://stackoverflow.com/questions/34398632/docker-how-to-run-pip-requirements-txt-only-if-there-was-a-change


## Users

https://medium.com/@mccode/processes-in-containers-should-not-run-as-root-2feae3f0df3b

```
RUN groupadd --gid 9876 projectgroup
RUN useradd -ms /bin/bash --uid 1234567 --gid 9876 projectdev
USER projectdev
```
