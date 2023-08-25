---
title: ntfy.sh docker service
date: 2023-06-24 11:11:00 -0700
categories: [Docker]
tags: ntfy 
---

### Local files

```
ntfy
├── cache
│   └──  
├── auth
│		└── user.db 
└── config
    └── server.yml
```

### server.yml

```
base-url: "https://ntfy.xxx.com"
upstream-base-url: "https://ntfy.sh" #iOS push must use this
attachment-cache-dir: "/var/cache/ntfy/attachments"
attachment-total-size-limit: "5G"
attachment-file-size-limit: "15M"
attachment-expiry-duration: "3h"
visitor-attachment-total-size-limit: "100M"
visitor-attachment-daily-bandwidth-limit: "500M"

auth-file: "/var/lib/ntfy/user.db"
auth-default-access: "deny-all"

behind-proxy: true # If you want use nginx or apache proxy by the service, you should use behind-proxy true
```

### docker run

```
docker run -p 4000:80 -v /oss/data/ntfy/cache:/var/cache/ntfy -v /oss/data/ntfy/config:/etc/ntfy -v /oss/data/ntfy/auth:/var/lib/ntfy -d binwiederhier/ntfy serve
```

### create user

```
docker exec -it ntfy ntfy user add --role=admin admin # add admin
docker exec -it ntfy ntfy user add test # add user test
```

