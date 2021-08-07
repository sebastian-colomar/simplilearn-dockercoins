# simplilearn-dockercoins

```
GITHUB_USERNAME=academiaonline
GITHUB_PROJECT=simplilearn-dockercoins
GITHUB_BRANCH=2021-08
GITHUB_RELEASE=test
NODEPORT=80

cd ${HOME}
git clone https://github.com/${GITHUB_USERNAME}/${GITHUB_PROJECT}
cd ${GITHUB_PROJECT}
git pull
git checkout ${GITHUB_BRANCH}

SERVICE=hasher
sudo docker image build --file ${SERVICE}/Dockerfile --tag ${GITHUB_USERNAME}/${GITHUB_PROJECT}:${GITHUB_RELEASE}-${SERVICE} ${SERVICE}/

SERVICE=rng
sudo docker image build --file ${SERVICE}/Dockerfile --tag ${GITHUB_USERNAME}/${GITHUB_PROJECT}:${GITHUB_RELEASE}-${SERVICE} ${SERVICE}/

SERVICE=webui
sudo docker image build --file ${SERVICE}/Dockerfile --tag ${GITHUB_USERNAME}/${GITHUB_PROJECT}:${GITHUB_RELEASE}-${SERVICE} ${SERVICE}/

SERVICE=worker
sudo docker image build --file ${SERVICE}/Dockerfile --tag ${GITHUB_USERNAME}/${GITHUB_PROJECT}:${GITHUB_RELEASE}-${SERVICE} ${SERVICE}/

SERVICE=hasher
sudo docker network create ${SERVICE}

SERVICE=redis
sudo docker network create ${SERVICE}

SERVICE=rng
sudo docker network create ${SERVICE}

CMD=redis-server
ENTRYPOINT=docker-entrypoint.sh
SERVICE=redis
NETWORK=${SERVICE}
WORKDIR=/data/
sudo docker volume create ${SERVICE}
sudo docker container run --detach --entrypoint ${ENTRYPOINT} --name ${SERVICE} --network ${NETWORK} --restart always --volume ${SERVICE}:${WORKDIR}:rw --workdir ${WORKDIR} ${GITHUB_USERNAME}/${GITHUB_PROJECT}:${GITHUB_RELEASE}-${SERVICE} ${CMD}

CMD=hasher.rb
ENTRYPOINT=ruby
SERVICE=hasher
NETWORK=${SERVICE}
WORKDIR=/${SERVICE}/
sudo docker container run --detach --entrypoint ${ENTRYPOINT} --name ${SERVICE} --network ${NETWORK} --restart always --volume ${SERVICE}/:${WORKDIR}:ro --workdir ${WORKDIR} ${GITHUB_USERNAME}/${GITHUB_PROJECT}:${GITHUB_RELEASE}-${SERVICE} ${CMD}

CMD=rng.py
ENTRYPOINT=python
SERVICE=rng
NETWORK=${SERVICE}
WORKDIR=/${SERVICE}/
sudo docker container run --detach --entrypoint ${ENTRYPOINT} --name ${SERVICE} --network ${NETWORK} --restart always --volume ${SERVICE}/:${WORKDIR}:ro --workdir ${WORKDIR} ${GITHUB_USERNAME}/${GITHUB_PROJECT}:${GITHUB_RELEASE}-${SERVICE} ${CMD}

CMD=worker.py
ENTRYPOINT=python
SERVICE=worker
NETWORK=redis
WORKDIR=/${SERVICE}/
sudo docker container run --detach --entrypoint ${ENTRYPOINT} --name ${SERVICE} --network ${NETWORK} --restart always --volume ${SERVICE}/:${WORKDIR}:ro --workdir ${WORKDIR} ${GITHUB_USERNAME}/${GITHUB_PROJECT}:${GITHUB_RELEASE}-${SERVICE} ${CMD}

NETWORK=hasher
sudo docker network connect ${NETWORK} ${SERVICE}

NETWORK=rng
sudo docker network connect ${NETWORK} ${SERVICE}

CMD=webui.js
ENTRYPOINT=node
SERVICE=webui
NETWORK=redis
WORKDIR=/${SERVICE}/
sudo docker container run --detach --entrypoint ${ENTRYPOINT} --name ${SERVICE} --network ${NETWORK} --publish ${NODEPORT}:8080--restart always --volume ${SERVICE}/:${WORKDIR}:ro --workdir ${WORKDIR} ${GITHUB_USERNAME}/${GITHUB_PROJECT}:${GITHUB_RELEASE}-${SERVICE} ${CMD}

```
