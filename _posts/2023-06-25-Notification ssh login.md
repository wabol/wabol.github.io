---
title: ntfy docker service
date: 2023-06-25 11:11:00 -0700
categories: [Linux]
tags: ntfy ssh_login
---

### vim sshrc to /etc/ssh/

```shell
#!/bin/bash

# Get login user
user=$(getent passwd `who` | head -n 1 | cut -d : -f 1)
if [ "" = "$user" ]; then
   user="default"
fi
# Get login user's IP
ip=${SSH_CLIENT%% *}
echo $ip
if [ "$ip" = "" ]; then
   ip="default"
fi
# Get login time
time=$(date +%F-%k:%M)
# Serverip
server=`ifconfig eth1|sed -n '2p'|awk -F ":" '{print $2}'|awk '{print $1}'`
if [ "$server" = "" ]; then
   server="default"
fi

# push to ntfy

curl -u user:password -d "ssh has a new login, $user $time $ip" https://ntfy.sh/mytopic >> /dev/logs/sshlogin.log 2>&1 &

```

