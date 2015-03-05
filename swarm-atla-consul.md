
## Start 5 docker macine on google

```
machine create -d google --google-machine-type n1-standard-1 --google-zone europe-west1-b --google-project siq-haas boot-1
machine create -d google --google-machine-type n1-standard-1 --google-zone europe-west1-b --google-project siq-haas boot-2
machine create -d google --google-machine-type n1-standard-1 --google-zone europe-west1-b --google-project siq-haas boot-3
machine create -d google --google-machine-type n1-standard-1 --google-zone europe-west1-b --google-project siq-haas boot-4
machine create -d google --google-machine-type n1-standard-1 --google-zone europe-west1-b --google-project siq-haas boot-5
```

## Consul bootstrap

Starting 3 consul servers, and 2 additional agents. Let swarm schedule them randomly

```
docker run --name=consul-1 -d --net=host -p 8500:8500 -p 8400:8400  sequenceiq/consul:v0.5.0-v4 -server -bootstrap -atlas-join -atlas-token=$ATLAS_TOKEN -atlas=sequenceiq/swarm

docker run --name=consul-2 -d --net=host -p 8500:8500 -p 8400:8400  sequenceiq/consul:v0.5.0-v4 -server -atlas-join -atlas-token=$ATLAS_TOKEN -atlas=sequenceiq/swarm

docker run --name=consul-3 -d --net=host -p 8500:8500 -p 8400:8400  sequenceiq/consul:v0.5.0-v4 -server -atlas-join -atlas-token=$ATLAS_TOKEN -atlas=sequenceiq/swarm

docker run --name=consul-4 -d --net=host -p 8500:8500 -p 8400:8400  sequenceiq/consul:v0.5.0-v4 -atlas-join -atlas-token=$ATLAS_TOKEN -atlas=sequenceiq/swarm

docker run --name=consul-5 -d --net=host -p 8500:8500 -p 8400:8400  sequenceiq/consul:v0.5.0-v4 -atlas-join -atlas-token=$ATLAS_TOKEN -atlas=sequenceiq/swarm
```