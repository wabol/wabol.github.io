---
title: Common Script
date: 2023-05-03 12:11:00 -0800
categories: [Linux]
tags: 
---

[![Hits](https://hits.seeyoufarm.com/api/count/incr/badge.svg?url=https%3A%2F%2Fnotes.966885.xyz%2Fposts%2FCommon-Script%2F&count_bg=%2379C83D&title_bg=%23555555&icon=&icon_color=%23E7E7E7&title=hits&edge_flat=false)](https://hits.seeyoufarm.com)

### About qnap

##### History start.sh

```shell
#!/bin/bash

# sleep 3;
# /bin/ln -sf /share/CACHEDEV1_DATA/.qpkg/config/htop/htoprc /root/.config/htop/htoprc &&
/bin/ln -sf /share/CACHEDEV1_DATA/.qpkg/config/bash_history /root/.bash_history
```

#####  Check IPV6

```shell
#!/bin/bash

ip -6 address show | grep inet6 | awk '/:632/ {print $2}' | cut -d'/' -f1 > /share/CACHEDEV1_DATA/.qpkg/shells/nasyy.html &&

cp /share/CACHEDEV1_DATA/.qpkg/shells/nasyy.html /share/CACHEDEV1_DATA/Container/nginx/www/index.html &&
scp -P 2222 /share/CACHEDEV1_DATA/.qpkg/shells/nasyy.html root@server.com:/home/www/ip/index.html

exit 0
```

##### Ssl_deploy

```shell
#!/bin/bash

cd /root/.acme.sh/qnap.server.com_ecc &&
cp qnap.server.com.cer /etc/stunnel/backup.cert &&
cp qnap.server.com.key /etc/stunnel/backup.key &&
cat /etc/stunnel/backup.cert /etc/stunnel/backup.key > /etc/stunnel/stunnel.pem &&
/etc/init.d/stunnel.sh restart

exit 0
```

##### Ssl_issue

```shell
#!/bin/bash

cd /root/.acme.sh &&
./acme.sh --issue -d qnap.server.com --challenge-alias 698example.xyz --dns dns_cf --server letsencrypt --dnssleep 300
# You can remove '--server letsencrypt' use defalut server zerossl.
exit 0
```

##### Zerotier check

```shell
#!/bin/bash

zerotier_process=$(ps -ef |grep 'zerotier-one' | grep -v 'grep' | awk '{print $1}')

#check main programme status

if [[ $zerotier_process = "" ]];then
cd /share/CACHEDEV1_DATA/.qpkg/shells/ &&
sh zerotier_start.sh

echo service start successful!

else

echo service is running!

fi
exit 0
```

##### Zerotirer start

```shell
#!/bin/bash

zerotier-one -d

exit 0
#/share/CACHEDEV1_DATA/.qpkg/Entware/bin/zerotier-one -d
```

