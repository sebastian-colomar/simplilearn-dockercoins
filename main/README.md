# simplilearn-dockercoins

## CLONE GITHUB REPOSITORY
```
GITHUB_BRANCH=main
GITHUB_PROJECT=simplilearn-dockercoins
GITHUB_RELEASE=test
GITHUB_USERNAME=sebastian-colomar
DOCKERHUB_USERNAME=academiaonline

cd ${HOME}
rm -rf ${GITHUB_PROJECT}/
git clone https://github.com/${GITHUB_USERNAME}/${GITHUB_PROJECT}
cd ${GITHUB_PROJECT}/
git pull
git checkout ${GITHUB_BRANCH}
git pull
```
## BUILD AND PUSH DOCKER IMAGES
```
cd main/

SERVICE=hasher
docker image build --file ${SERVICE}/Dockerfile --tag ${DOCKERHUB_USERNAME}/${GITHUB_PROJECT}:${GITHUB_RELEASE}-${SERVICE} ${SERVICE}/
docker image push ${DOCKERHUB_USERNAME}/${GITHUB_PROJECT}:${GITHUB_RELEASE}-${SERVICE}

SERVICE=rng
docker image build --file ${SERVICE}/Dockerfile --tag ${DOCKERHUB_USERNAME}/${GITHUB_PROJECT}:${GITHUB_RELEASE}-${SERVICE} ${SERVICE}/
docker image push ${DOCKERHUB_USERNAME}/${GITHUB_PROJECT}:${GITHUB_RELEASE}-${SERVICE}

SERVICE=webui
docker image build --file ${SERVICE}/Dockerfile --tag ${DOCKERHUB_USERNAME}/${GITHUB_PROJECT}:${GITHUB_RELEASE}-${SERVICE} ${SERVICE}/
docker image push ${DOCKERHUB_USERNAME}/${GITHUB_PROJECT}:${GITHUB_RELEASE}-${SERVICE}

SERVICE=worker
docker image build --file ${SERVICE}/Dockerfile --tag ${DOCKERHUB_USERNAME}/${GITHUB_PROJECT}:${GITHUB_RELEASE}-${SERVICE} ${SERVICE}/
docker image push ${DOCKERHUB_USERNAME}/${GITHUB_PROJECT}:${GITHUB_RELEASE}-${SERVICE}

```
## CREATE DOCKER NETWORKS, VOLUMES AND CONTAINERS
```
SERVICE=hasher
docker network create --driver bridge ${SERVICE}

SERVICE=redis
docker network create --driver bridge ${SERVICE}

SERVICE=rng
docker network create --driver bridge ${SERVICE}

CPUS=0.010
MEMORY=100M
USER=1000620000:1000620000

WORKDIR=/data/

CMD=redis-server
ENTRYPOINT=/usr/local/bin/docker-entrypoint.sh
SERVICE=redis
NETWORK=${SERVICE}
VOLUME=${SERVICE}
docker volume create ${SERVICE}
docker container run --cpus ${CPUS} --detach --entrypoint ${ENTRYPOINT} --memory ${MEMORY} --name ${SERVICE} --network ${NETWORK} --read-only --restart always --user ${USER} --volume ${VOLUME}:${WORKDIR}:rw --workdir ${WORKDIR} library/redis:alpine ${CMD}

docker container logs ${SERVICE}
docker container stats --no-stream ${SERVICE}
docker container top ${SERVICE}

WORKDIR=/app

CMD=hasher.rb
ENTRYPOINT=/usr/local/bin/ruby
SERVICE=hasher
NETWORK=${SERVICE}
VOLUME=${PWD}/${SERVICE}
docker container run --cpus ${CPUS} --detach --entrypoint ${ENTRYPOINT} --memory ${MEMORY} --name ${SERVICE} --network ${NETWORK} --read-only --restart always --user ${USER} --volume ${VOLUME}/:${WORKDIR}:ro --workdir ${WORKDIR} ${DOCKERHUB_USERNAME}/${GITHUB_PROJECT}:${GITHUB_RELEASE}-${SERVICE} ${CMD}

docker container logs ${SERVICE}
docker container stats --no-stream ${SERVICE}
docker container top ${SERVICE}

CMD=rng.py
ENTRYPOINT=/usr/local/bin/python
SERVICE=rng
NETWORK=${SERVICE}
VOLUME=${PWD}/${SERVICE}
docker container run --cpus ${CPUS} --detach --entrypoint ${ENTRYPOINT} --memory ${MEMORY} --name ${SERVICE} --network ${NETWORK} --read-only --restart always --user ${USER} --volume ${VOLUME}/:${WORKDIR}:ro --workdir ${WORKDIR} ${DOCKERHUB_USERNAME}/${GITHUB_PROJECT}:${GITHUB_RELEASE}-${SERVICE} ${CMD}

docker container logs ${SERVICE}
docker container stats --no-stream ${SERVICE}
docker container top ${SERVICE}

CMD=worker.py
ENTRYPOINT=python
SERVICE=worker
NETWORK=redis
VOLUME=${PWD}/${SERVICE}
docker container run --cpus ${CPUS} --detach --entrypoint ${ENTRYPOINT} --memory ${MEMORY} --name ${SERVICE} --network ${NETWORK} --read-only --restart always --user ${USER} --volume ${VOLUME}/:${WORKDIR}:ro --workdir ${WORKDIR} ${DOCKERHUB_USERNAME}/${GITHUB_PROJECT}:${GITHUB_RELEASE}-${SERVICE} ${CMD}

NETWORK=hasher
docker network connect ${NETWORK} ${SERVICE}

NETWORK=rng
docker network connect ${NETWORK} ${SERVICE}

docker container logs ${SERVICE}
docker container stats --no-stream ${SERVICE}
docker container top ${SERVICE}

CMD=webui.js
ENTRYPOINT=/usr/local/bin/node
SERVICE=webui
NETWORK=redis
NODEPORT=80
TARGETPORT=8080
VOLUME=${PWD}/${SERVICE}
docker container run --cpus ${CPUS} --detach --entrypoint ${ENTRYPOINT} --memory ${MEMORY} --name ${SERVICE} --network ${NETWORK} --publish ${NODEPORT}:${TARGETPORT} --read-only --restart always --user ${USER} --volume ${VOLUME}/:${WORKDIR}:ro --workdir ${WORKDIR} ${DOCKERHUB_USERNAME}/${GITHUB_PROJECT}:${GITHUB_RELEASE}-${SERVICE} ${CMD}

docker container logs ${SERVICE}
docker container stats --no-stream ${SERVICE}
docker container top ${SERVICE}

```
## DEPLOY WITH DOCKER SWARM
```
docker stack deploy --compose-file docker-compose.yaml ${GITHUB_PROJECT}_${GITHUB_RELEASE}
```
## DEPLOY WITH KUBERNETES/OPENSHIFT
```
oc new-project ${GITHUB_PROJECT}-${GITHUB_RELEASE}
kubectl create namespace ${GITHUB_PROJECT}-${GITHUB_RELEASE}
kubectl create secret generic hasher-secret --from-file hasher/hasher.rb --namespace ${GITHUB_PROJECT}-${GITHUB_RELEASE}
kubectl create secret generic rng-secret --from-file rng/rng.py --namespace ${GITHUB_PROJECT}-${GITHUB_RELEASE}
kubectl create secret generic webui-secret --from-file webui/webui.js --namespace ${GITHUB_PROJECT}-${GITHUB_RELEASE}
kubectl create secret generic worker-secret --from-file worker/worker.py --namespace ${GITHUB_PROJECT}-${GITHUB_RELEASE}
kubectl apply --filename etc/kubernetes/manifests/ --namespace ${GITHUB_PROJECT}-${GITHUB_RELEASE}
```
