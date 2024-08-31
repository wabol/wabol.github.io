---
title: Docker Swarm
date: 2024-08-30 11:31:00 -0700
categories: [docker]
tags: 
Pin:
---

# Docker Swarm

## Docker Swarm Initialization

### 1.Install Docker and initial
Skip

### 2. Initialize Docker Swarm Cluster
Select everyone to set up cluster.

```shell
docker swarm init --advertise-addr <cluster_ip>
```
You can get a swarm token like this `ddocker swarm join --token SWMTKN-1-2142i8zn <cluster_ip>:2377`

- If you want **mutiple managers**, use `docker swarm join-token manager` to get manager token. `docker swarm join-token worker` to get worker token.
- We can also use `docker node promote <NODE_NAME>` to promote a node to manager.

### 3. Add Nodes
Use `docker swarm join --token SWMTKN-1-2142i8zn <cluster_ip>:2377` to add nodes.

### 4. Remove Nodes
Use `docker swarm leave` to remove nodes.

### 5. View Nodes and Services
Use `docker node ls` to view nodes.
Use `docker service ls` to view services.

## Docker Swarm Configuration Examples

### 1. Create a nginx service

```shell
docker service create --name test-nginx --publish 80:80 --replicas 3 nginx
```

### 2. Check service status

```shell
docker service ls
docker service ps test-nginx # check nginx service
```

### 3. Scale service

```shell
docker service scale test-nginx=5
```

### 4. Remove service

```shell
docker service rm test-nginx
```

## Docker Swarm Compose

### 1. Docker Compose file

```yaml
version: '3.8'

services:
  web:
    image: nginx:latest
    hostname: "{{.Service.Name}}-{{.Task.Slot}}"
    ports:
      - "80:80"
    deploy:
      replicas: 3
      restart_policy:
        condition: on-failure
      placement:
        constraints:
          - node.role == worker
    networks:
      - webnet

networks:
  webnet:
```
placement.constraints means the node must be a worker node. It is optional.

### 2. Run Compose

```shell
docker stack deploy -c docker-compose.yaml test_nginx
```

### 3. Change replicas number

Modify replicas number in `docker-compose.yaml`, and run Compose again.
```shell
docker stack deploy -c docker-compose.yaml test_nginx
```

### 4. Remove Compose

```shell
docker stack rm test_nginx
```