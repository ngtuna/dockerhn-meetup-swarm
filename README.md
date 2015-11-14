# Dockerhn meetup 14/11 - Toong co-working space
# `Docker Swarm` - ready for production use.

## Create Swarm with Hosted Discovery Service from Docker Hub

```
### Create local Virtualbox machine
$ docker-machine create --driver=virtualbox --virtualbox-memory=1024 local
### load ~local~ machine configuration into shell
$ eval "$(docker-machine env local)"
### generate a discovery token using the Docker Swarm iamge
$ docker run swarm create
..
Digest: sha256:51a30269d3f3aaa04f744280e3c118aea032f6df85b49819aee29d379ac313b5
Status: Downloaded newer image for swarm:latest
9cec215f2925252d37a485fc18a16127
```
### Launch the Swarm Manager
#### create a swarm master under Virtualbox
```
$ docker-machine create --driver=virtualbox \
        --swarm \
        --swarm-master \
        --swarm-discovery token://9cec215f2925252d37a485fc18a16127 \
        swarm-master
```
### Generate two Swarm nodes
```
$ docker-machine create --driver=virtualbox \
        --swarm \
        --swarm-discovery token://9cec215f2925252d37a485fc18a16127 \
        swarm-agent-00
$ docker-machine create --driver=virtualbox \
        --swarm \
        --swarm-discovery token://9cec215f2925252d37a485fc18a16127 \
        swarm-agent-01
```
### Direct your swarm
```
### point Docker env to the machine running swarm master
$ eval $(docker-machine env --swarm swarm-master)
### get information on the new swarm
$ docker info
### check images currently running on your swarm
$ docker ps -a
### check swarm version
$ docker version
```

## Create Swarm cluster with Consul service discovery (overlay network, volume demo)

### Launch a Consul master
```
$ docker-machine create --driver=virtualbox \
        consul-master
$ docker $(docker-machine config consul-master) run -d \
        -p "8500:8500" \
        -h "consul" \
        progrium/consul -server -bootstrap
```
### Launch a Swarm master
```
$ docker-machine create \
    --driver=virtualbox \
    --swarm \
    --swarm-master \
    --swarm-discovery="consul://$(docker-machine ip consul-master):8500" \
    --engine-opt="cluster-store=consul://$(docker-machine ip consul-master):8500" \
    --engine-opt="cluster-advertise=eth1:0" \
    swarm-master
```
### Launch two Swarm nodes
```
$ docker-machine create \
    --driver=virtualbox \
    --swarm \
    --swarm-discovery="consul://$(docker-machine ip consul-master):8500" \
    --engine-opt="cluster-store=consul://$(docker-machine ip consul-master):8500" \
    --engine-opt="cluster-advertise=eth1:0" \
    swarm-agent-00
$ docker-machine create \
    --driver=virtualbox \
    --swarm \
    --swarm-discovery="consul://$(docker-machine ip consul-master):8500" \
    --engine-opt="cluster-store=consul://$(docker-machine ip consul-master):8500" \
    --engine-opt="cluster-advertise=eth1:0" \
    swarm-agent-01
```
### Check info
```
$ eval $(docker-machine env --swarm swarm-master)
$ docker info
```

### Multi-host networking
```
### Create an overlay network
$ docker network create -d overlay overlay_test
### Run couple of containers on that network
$ docker run --name=app1 -it --net=overlay_test busybox
$ docker run --name=db1 -it --net=overlay_test busybox
### remember to set eval $(docker-machine env --swarm swarm-master)

### Check info
$ cat /etc/hosts
$ ifconfig
$ ping db1
$ ping app1
$ ping app2
```
#### Demo with docker-compose
```
$ docker-compose --x-network-driver=overlay --x-networking up -d
ERROR: Couldn't connect to Docker daemon - you might need to run `docker-machine start default`.
```

#### Kubernetes over Swarm
```
$ git clone https://github.com/docker/swarm-frontends.git
$ cd kubernetes
$ docker-compose --x-networking -f k8s-swarm.yml up
```
Modify your `.kuber/config`  to point to your kubernetes cluster. To do this, type `docker ps`  and fetch the IP and port exposed from the `apiserver`
You can then list nodes using:
```
$ kubectl get nodes
```
Scale
```
$ docker-compose --x-networking -f k8s-swarm.yml scale kubelet=2 proxy=2
```

### Volume
```
$ docker volume create --name=data
$ docker run -it -v data:/data alpine /bin/sh -c "echo hello world > /data/demo"
$ docker run -it -v data:/data alpine cat /data/demo
ERROR: cat: can't open '/data/demo': No such file or directory
```

# Reference
```
https://github.com/dave-tucker/docker-network-demos
https://github.com/docker/community
```
