---
title: Nginx unit getting started
date: 2022-11-28 11:37:00 
categories: [Nginx unit]
tags: [Nginx]
---

Unit is a lightweight open-source server, it is a webserver, application server and a proxy. How to use Unit? Let's getting started.

### Install

please read the official document.

[https://unit.nginx.org/installation/](https://unit.nginx.org/installation/)

### Unit with Docker

1.**Pull images**

```
sudo docker pull nginx/unit
```

2.**Run images**

```
sudo docker run -d --mount type=bind,src="$(pwd)/config/",dst=/docker-entrypoint.d/ --mount type=bind,src="$(pwd)/webapp",dst=/www -p 8080:8080 docker.io/nginx/unit:latest
```

3.**Directory permission setting**

```
chown -R nobody:nogroup "$(pwd)"
```

4.**Put config and webapp files to "$(pwd)/config/" and "$(pwd)/webapp"**

config example, add a config.json

```
{
    "listeners": {
        "*:8080": {
            "pass": "routes"
        }
    },

    "routes": [
        {
            "action": {
                "share": "/www/"
            }
        }
    ]
}
```

5.**Put config to Unit**

```
sudo docker exec -it contaniersID curl -X PUT --data-binary @/docker-entrypoint.d/config.json  --unix-socket /var/run/control.unit.sock http://localhost/config

#Delete config

sudo docker exec -it 4e6cded816dd curl -X DELETE --data-binary @/docker-entrypoint.d/config.json --unix-socket /var/run/control.unit.sock  http://localhost/config
```

### Unit with Linux (Centos, Ubuntu, Debian, etc.)

1. **setting config**

config example

```
{
    "listeners": {
        "*:8080": {
            "pass": "routes"
        }
    },

    "routes": [
        {
            "action": {
                "share": "/home/www$uri"
            }
        }
    ]
}
```

2. **Put config to Unit**

```
sudo curl -X PUT --data-binary @/$(pwd)/config.json  --unix-socket /var/run/control.unit.sock http://localhost/config

#Delete config
sudo curl -X PUT --data-binary @/$(pwd)/config.json  --unix-socket /var/run/control.unit.sock http://localhost/config
```

