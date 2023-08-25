---
title: Setup Webdav using Caddy
date: 2023-08-23 11:11:00 -0700
categories: [Linux]
tags: Caddy
---

## Install Caddy Server

Please see this link https://caddyserver.com/docs/install

## Install Caddy Webdav module

```shell
sudo caddy add-package github.com/mholt/caddy-webdav
sudo systemctl restart caddy
```

## Configuring Caddy

First, you should create a new hash password

```
caddy hash-password
# example: the "testtest" hash password is "$2a$14$zzWDbY.LBf6W20cg1aJMcurvj2YWAUEUevN5GA/bhVFXsUG5IrzbG"
```

Then, set the Caddyfile

```
sudo vim /etc/caddy/Caddyfile
```

```
# configure webdav module
{
    order webdav before file_server
}

# set up webdav for the webdav.example.com host
webdav.example.com {
    root * /data/webdav
    basicauth {
        user $2a$14$zzWDbY.LBf6W20cg1aJMcurvj2YWAUEUevN5GA/bhVFXsUG5IrzbG
    }
    webdav
}
```

then restart the Caddy

```
systemctl restart caddy
```

You can use "user and testtest" login in the webdav service "webdav.example.com"
