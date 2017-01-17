# Docker Swarm Presentation


## Building a docker swarm cluster
```
for i in 1 2 3; do
    docker-machine create -d virtualbox swarm-$i
done
```

Checking my machines
```
docker-machine ls
```

The output is
```
NAME      ACTIVE   DRIVER       STATE     URL                         SWARM   DOCKER    ERRORS
swarm-1   -        virtualbox   Running   tcp://192.168.99.100:2376           v1.12.5
swarm-2   -        virtualbox   Running   tcp://192.168.99.101:2376           v1.12.5
swarm-3   -        virtualbox   Running   tcp://192.168.99.102:2376           v1.12.5
```

Creating the cluster
```
eval "$(docker-machine env swarm-1)"

docker swarm init --advertise-addr $(docker-machine ip swarm-1)
```

## Adding the visualizer service
```
eval "$(docker-machine env swarm-1)"
docker service create \
  --name=visualizer \
  --publish=8000:8080/tcp \
  --constraint=node.role==manager \
  --mount=type=bind,src=/var/run/docker.sock,dst=/var/run/docker.sock \
  manomarks/visualizer

open http://$(docker-machine ip swarm-1):8000
```

## Adding workers to the cluster
```
eval "$(docker-machine env swarm-1)"
JOIN_TOKEN=$(docker swarm join-token -q worker)

for i in 2 3; do
    eval "$(docker-machine env swarm-$i)"

    docker swarm join --token $JOIN_TOKEN \
        --advertise-addr $(docker-machine ip swarm-$i) \
        $(docker-machine ip swarm-1):2377
done
```

```
eval "$(docker-machine env swarm-1)"
docker node ls
```

## Deploy a new service
```
eval "$(docker-machine env swarm-1)"
docker service create \
  --name=docker-routing-mesh \
  --publish=8080:8080/tcp \
  albertogviana/docker-routing-mesh

curl http://$(docker-machine ip swarm-1):8080
curl http://$(docker-machine ip swarm-2):8080
curl http://$(docker-machine ip swarm-3):8080
```

## Scaling a service
```
docker service scale docker-routing-mesh=3
```

## Calling a service
```
while true; do curl http://$(docker-machine ip swarm-1):8080; sleep 1; echo "\n";  done
```
