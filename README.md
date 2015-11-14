# Dockerhn meetup 14/11 - Toong co-working space
# Docker Swarm - ready for production use.

## Generate swarm token

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
## Launch the Swarm Manager
### create a swarm master under Virtualbox
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
```
