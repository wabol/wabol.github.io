---
title: Docker registry deploy
date: 2023-05-26 12:11:00 -0800
categories: [Docker]
tags: docker
---

[![Hits](https://hits.seeyoufarm.com/api/count/incr/badge.svg?url=https%3A%2F%2Fnotes.966885.xyz%2Fposts%2FDocker-registry-depoly%2F&count_bg=%2379C83D&title_bg=%23555555&icon=&icon_color=%23E7E7E7&title=hits&edge_flat=false)](https://hits.seeyoufarm.com)

##### 1.Basic authentication, usally use htpasswd for authentication.

```
sudo mkdir -p /home/auth
cd /home/auth
sudo docker run --entrypoint htpasswd   httpd:2 -Bbn testuser testpassword | sudo tee htpasswd
# 'tee' is used to capture the output of the 'docker run' command
```

##### 2.Start registry with below command

```shell
docker run -d \
  -p 5000:5000 \
  --name registry \
  -v /home/auth:/auth \
  -v /mnt/registry:/var/lib/registry \
  -e "REGISTRY_AUTH=htpasswd" \
  -e "REGISTRY_AUTH_HTPASSWD_REALM=Registry Realm" \
  -e REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd \
  registry:2
```

##### 3.Use Nginx proxy to registry service

```
# Nginx conf

location / {
    # Do not allow connections from docker 1.5 and earlier
    # docker pre-1.6.0 did not properly set the user agent on ping, catch "Go *" user agents
    if ($http_user_agent ~ "^(docker\/1\.(3|4|5(?!\.[0-9]-dev))|Go ).*$" ) {
      return 404;
    }

    proxy_pass                          http://localhost:5000;
    proxy_set_header  Host              $http_host;   # required for docker client's sake
    proxy_set_header  X-Real-IP         $remote_addr; # pass on real client's IP
    proxy_set_header  X-Forwarded-For   $proxy_add_x_forwarded_for;
    proxy_set_header  X-Forwarded-Proto $scheme;
    proxy_read_timeout                  900;
}

# There are more information. https://www.digitalocean.com/community/tutorials/how-to-set-up-a-private-docker-registry-on-ubuntu-20-04

```

##### 4.how to use

```
docker tag 56f8xxxxxID hub.xxx.com/python-image
docker login https://hub.xxx.com
docker push hub.xxx.com/python-image

```

