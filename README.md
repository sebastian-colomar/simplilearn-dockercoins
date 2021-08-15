# simplilearn-dockercoins

## CLONE GITHUB REPOSITORY
```
export ADVERTISE_ADDR=$( ip route | grep dev.eth0.proto.kernel | awk '{ print $9 }' )
export ENV_FILE=common.env
export GITHUB_BRANCH=2021-08
export GITHUB_PROJECT=simplilearn-dockercoins
export GITHUB_RELEASE=test
export GITHUB_USERNAME=academiaonline
export NODEPORT=80

cd ${HOME}
git clone https://github.com/${GITHUB_USERNAME}/${GITHUB_PROJECT}
cd ${GITHUB_PROJECT}
git pull
git checkout ${GITHUB_BRANCH}
git pull
```
## BUILD AND PUSH DOCKER IMAGES
```
SERVICE=hasher
sudo docker image build --file ${SERVICE}/Dockerfile --tag ${GITHUB_USERNAME}/${GITHUB_PROJECT}:${GITHUB_RELEASE}-${SERVICE} ${SERVICE}/
sudo docker image push ${GITHUB_USERNAME}/${GITHUB_PROJECT}:${GITHUB_RELEASE}-${SERVICE}

SERVICE=rng
sudo docker image build --file ${SERVICE}/Dockerfile --tag ${GITHUB_USERNAME}/${GITHUB_PROJECT}:${GITHUB_RELEASE}-${SERVICE} ${SERVICE}/
sudo docker image push ${GITHUB_USERNAME}/${GITHUB_PROJECT}:${GITHUB_RELEASE}-${SERVICE}

SERVICE=webui
sudo docker image build --file ${SERVICE}/Dockerfile --tag ${GITHUB_USERNAME}/${GITHUB_PROJECT}:${GITHUB_RELEASE}-${SERVICE} ${SERVICE}/
sudo docker image push ${GITHUB_USERNAME}/${GITHUB_PROJECT}:${GITHUB_RELEASE}-${SERVICE}

SERVICE=worker
sudo docker image build --file ${SERVICE}/Dockerfile --tag ${GITHUB_USERNAME}/${GITHUB_PROJECT}:${GITHUB_RELEASE}-${SERVICE} ${SERVICE}/
sudo docker image push ${GITHUB_USERNAME}/${GITHUB_PROJECT}:${GITHUB_RELEASE}-${SERVICE}

```
## CREATE DOCKER NETWORKS, VOLUMES AND CONTAINERS
```

SERVICE=hasher
sudo docker network create --driver bridge ${SERVICE}

SERVICE=redis
sudo docker network create --driver bridge ${SERVICE}

SERVICE=rng
sudo docker network create --driver bridge ${SERVICE}

CMD=redis-server
ENTRYPOINT=docker-entrypoint.sh
SERVICE=redis
NETWORK=${SERVICE}
VOLUME=${SERVICE}
WORKDIR=/data/
sudo docker volume create ${SERVICE}
sudo docker container run --cpus 0.010 --detach --entrypoint ${ENTRYPOINT} --memory 100M --name ${SERVICE} --network ${NETWORK} --read-only --restart always --volume ${VOLUME}:${WORKDIR}:rw --workdir ${WORKDIR} library/redis:alpine ${CMD}

sudo docker container logs ${SERVICE}
sudo docker container stats --no-stream ${SERVICE}
sudo docker container top ${SERVICE}

CMD=hasher.rb
ENTRYPOINT=ruby
SERVICE=hasher
NETWORK=${SERVICE}
VOLUME=${PWD}/${SERVICE}
WORKDIR=/${SERVICE}/
sudo docker container run --cpus 0.010 --detach --entrypoint ${ENTRYPOINT} --memory 100M --name ${SERVICE} --network ${NETWORK} --read-only --restart always --volume ${VOLUME}/:${WORKDIR}:ro --workdir ${WORKDIR} ${GITHUB_USERNAME}/${GITHUB_PROJECT}:${GITHUB_RELEASE}-${SERVICE} ${CMD}

sudo docker container logs ${SERVICE}
sudo docker container stats --no-stream ${SERVICE}
sudo docker container top ${SERVICE}

CMD=rng.py
ENTRYPOINT=python
SERVICE=rng
NETWORK=${SERVICE}
VOLUME=${PWD}/${SERVICE}
WORKDIR=/${SERVICE}/
sudo docker container run --cpus 0.010 --detach --entrypoint ${ENTRYPOINT} --memory 100M --name ${SERVICE} --network ${NETWORK} --read-only --restart always --volume ${VOLUME}/:${WORKDIR}:ro --workdir ${WORKDIR} ${GITHUB_USERNAME}/${GITHUB_PROJECT}:${GITHUB_RELEASE}-${SERVICE} ${CMD}

sudo docker container logs ${SERVICE}
sudo docker container stats --no-stream ${SERVICE}
sudo docker container top ${SERVICE}

CMD=worker.py
ENTRYPOINT=python
SERVICE=worker
NETWORK=redis
VOLUME=${PWD}/${SERVICE}
WORKDIR=/${SERVICE}/
sudo docker container run --cpus 0.010 --detach --entrypoint ${ENTRYPOINT} --memory 100M --name ${SERVICE} --network ${NETWORK} --read-only --restart always --volume ${VOLUME}/:${WORKDIR}:ro --workdir ${WORKDIR} ${GITHUB_USERNAME}/${GITHUB_PROJECT}:${GITHUB_RELEASE}-${SERVICE} ${CMD}

NETWORK=hasher
sudo docker network connect ${NETWORK} ${SERVICE}

NETWORK=rng
sudo docker network connect ${NETWORK} ${SERVICE}

sudo docker container logs ${SERVICE}
sudo docker container stats --no-stream ${SERVICE}
sudo docker container top ${SERVICE}

CMD=webui.js
ENTRYPOINT=node
SERVICE=webui
NETWORK=redis
VOLUME=${PWD}/${SERVICE}
WORKDIR=/${SERVICE}/
sudo docker container run --cpus 0.010 --detach --entrypoint ${ENTRYPOINT} --memory 100M --name ${SERVICE} --network ${NETWORK} --read-only --publish ${NODEPORT}:8080 --restart always --volume ${VOLUME}/:${WORKDIR}:ro --workdir ${WORKDIR} ${GITHUB_USERNAME}/${GITHUB_PROJECT}:${GITHUB_RELEASE}-${SERVICE} ${CMD}

sudo docker container logs ${SERVICE}
sudo docker container stats --no-stream ${SERVICE}
sudo docker container top ${SERVICE}

```
## DEPLOY WITH DOCKER SWARM
```
source ${ENV_FILE}
docker swarm init --advertise-addr ${ADVERTISE_ADDR}
docker stack deploy --compose-file docker-compose.yaml ${GITHUB_PROJECT}_${GITHUB_RELEASE}
```
