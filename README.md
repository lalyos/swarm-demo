## Start 5 docker macine on google

```
machine create -d google --google-machine-type n1-standard-1 --google-zone europe-west1-b --google-project siq-haas boot-1
machine create -d google --google-machine-type n1-standard-1 --google-zone europe-west1-b --google-project siq-haas boot-2
machine create -d google --google-machine-type n1-standard-1 --google-zone europe-west1-b --google-project siq-haas boot-3
machine create -d google --google-machine-type n1-standard-1 --google-zone europe-west1-b --google-project siq-haas boot-4
machine create -d google --google-machine-type n1-standard-1 --google-zone europe-west1-b --google-project siq-haas boot-5
```

## Start swarm manager

For discovery lets use a file:
```
for m in boot-{1..5}; do 
  echo $(machine ip $m):2376
  # machine ip $m &> /dev/null &&  echo $(machine ip $m):2376
done > swarm
```

locally on laptop/host
```
swarm-0.1.0 manage \
  -H tcp://0.0.0.0:3376 \
  --tlsverify \
  --tlscacert=./certs/ca.pem \
  --tlscert=./certs/server.pem \
  --tlskey=./certs/server-key.pem \
  file://./swarm
```

In a Docker container:
```
docker run -it \
  -p 3376:3376 \
  -v $PWD/swarm:/tmp/swarm \
  -v /Users/lalyos/.docker/machine/machines/boot-01:/certs \
    swarm manage -H tcp://0.0.0.0:3376  \
      --tlsverify \
      --tlscacert=/certs/ca.pem \
      --tlscert=/certs/server.pem \
      --tlskey=/certs/server-key.pem \
      file:///tmp/swarm
```

## Configure docker client

Tell docker client to talk to swarm manager:

```
docker() { 
  DOCKER_TLS_VERIFY='' \
  DOCKER_CERT_PATH=/Users/lalyos/.docker/machine/machines/boot-1 \
  /usr/local/bin/docker \
    --tls \
    -H tcp://127.0.0.1:3376 \
    "$@"
}
```

## Consul bootstrap

Starting 3 consul servers, and 2 additional agents. Let swarm schedule them randomly

```
export CON_DOCKER_IMG=sequenceiq/consul:v0.5.0-v4
export CON_DOCKER_OPTS="-d --net=host -p 8500:8500 -p 8400:8400"
export ATLAS_OPTS="-atlas-join -atlas-token=$ATLAS_TOKEN -atlas=sequenceiq/swarm"

docker run --name=consul-1 $CON_DOCKER_OPTS $CON_DOCKER_IMG $ATLAS_OPTS -server -bootstrap
docker run --name=consul-2 $CON_DOCKER_OPTS $CON_DOCKER_IMG $ATLAS_OPTS -server
docker run --name=consul-3 $CON_DOCKER_OPTS $CON_DOCKER_IMG $ATLAS_OPTS -server
docker run --name=consul-4 $CON_DOCKER_OPTS $CON_DOCKER_IMG $ATLAS_OPTS
docker run --name=consul-5 $CON_DOCKER_OPTS $CON_DOCKER_IMG $ATLAS_OPTS
```

starting 5 registrator:
```
echo {1..5} | xargs -n 1 docker run \
  -d \
  -p 9999:9999 \
  --net=host \
  -v /var/run/docker.sock:/tmp/docker.sock \
  gliderlabs/registrator:v5 consul://127.0.0.1:8500
```

## Consul commands

You can talk to remote consuls, via the RPC addre. To get the consul ip you can simple `docker inspect`

```
CONSUL_RPC_ADDR=$(docker inspect -f '{{.Node.IP}}'  consul-1):8400 consul members
```