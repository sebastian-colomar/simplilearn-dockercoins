# simplilearn-dockercoins

```
git clone https://github.com/academiaonline/simplilearn-dockercoins 
cd simplilearn-dockercoins/
git checkout 2021-09

alias docker='sudo docker'
```
```
docker container rm --force $( docker container ls --all --quiet )
docker system prune --force --volumes
```
```
docker image build --file Dockerfile-hasher --tag local/simplilearn-dockercoins:test-hasher /mnt/
docker image build --file Dockerfile-rng --tag local/simplilearn-dockercoins:test-rng /mnt/
docker image build --file Dockerfile-webui --tag local/simplilearn-dockercoins:test-webui /mnt/
docker image build --file Dockerfile-worker --tag local/simplilearn-dockercoins:test-worker /mnt/ 

docker network create hasher
docker network create redis
docker network create rng

docker volume create redis
docker volume create rng_pycache
docker volume create rng_http_pycache
docker volume create worker_pycache

docker container run --detach \
  --name redis --network redis \
  --read-only \
  --restart always \
  --volume redis:/data:rw \
  library/redis:6.2.5-alpine3.14@sha256:649d5317016d601ac7d6a7b7ef56b6d96196fb7df433d10143189084d52ee6f7
  
docker container run --detach \
  --entrypoint ruby \
  --name hasher --network hasher \
  --read-only \
  --restart always \
  --volume ${PWD}/hasher/hasher.rb:/hasher.rb:ro \
  local/simplilearn-dockercoins:test-hasher \
  hasher.rb
  
docker container run --detach \
  --entrypoint python \
  --name rng --network rng \
  --read-only \
  --restart always \
  --volume ${PWD}/rng/rng.py:/rng.py:ro \
  --volume rng_pycache:/usr/local/lib/python3.9/__pycache__/:rw \
  --volume rng_http_pycache:/usr/local/lib/python3.9/http/__pycache__/:rw \
  local/simplilearn-dockercoins:test-rng \
  rng.py

docker container run --detach \
  --entrypoint python \
  --name worker --network redis \
  --read-only \
  --restart always \
  --volume ${PWD}/worker/worker.py:/worker.py:ro \
  --volume worker_pycache:/usr/local/lib/python3.9/distutils/__pycache__/:rw \
  local/simplilearn-dockercoins:test-worker \
  worker.py

docker network connect hasher worker
docker network connect rng worker

docker container run --detach \
  --entrypoint node \
  --name webui --network redis \
  --read-only \
  --restart always \
  --publish 80:8080 \
  --volume ${PWD}/webui/webui.js:/webui.js:ro \
  --volume ${PWD}/webui/files/:/files/:ro \
  local/simplilearn-dockercoins:test-webui \
  webui.js
```
```
docker container top hasher
docker container top redis
docker container top rng
docker container top webui
docker container top worker
```
```
docker container diff hasher
docker container diff redis
docker container diff rng
docker container diff webui
docker container diff worker
```
```
docker container logs hasher --tail 1
docker container logs redis --tail 1
docker container logs rng --tail 1
docker container logs webui --tail 1
docker container logs worker --tail 1
```
```
docker container stats --no-stream hasher
docker container stats --no-stream redis
docker container stats --no-stream rng
docker container stats --no-stream webui
docker container stats --no-stream worker
```
