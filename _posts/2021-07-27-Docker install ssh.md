---
layout: post
tag: docker
title: Docker install ssh
---

`docker run -it -d ubuntu:lastest /bin/bash`

**Ubuntu** 

```
		1  apt-get update
    2  apt-get install openssh-server
    3  apt-get install vim
    4  vim /etc/ssh/sshd_config
    5  /etc/init.d/ssh restart
    6  cd /root/
    7  mkdir .ssh
    8  cd .ssh/
    9  vi authorized_keys
   10  /etc/init.d/ssh restart
```

**Centos**

```
		1  yum install openssh-server
    2  vi /etc/ssh/sshd_config
    4  systemctl start ssh
    5  /usr/sbin/sshd
    6  ssh-keygen -t dsa -f /etc/ssh/ssh_host_dsa_key
    7  ssh-keygen -t rsa -f /etc/ssh/ssh_host_rsa_key
    8  /usr/sbin/sshd
    9  ssh-keygen -t ecdsa -f /etc/ssh/ssh_host_ecdsa_key
   10  ssh-keygen -t ed25519 -f /etc/ssh/ssh_host_ed25519_key
   11  /usr/sbin/sshd
   12  ps -ef 
   13  ss -ln
   14  exit
   15  cd /root/
   16  ll
   17  ls
   18  mkdir .ssh
   19  cd .ssh/
   20  vi authorized_keys

```

**Alpine**

```
1	apk add --no-cache openssh
2	ssh-keygen -t dsa -P "" -f /etc/ssh/ssh_host_dsa_key
3	ssh-keygen -t rsa -P "" -f /etc/ssh/ssh_host_rsa_key
4	ssh-keygen -t ecdsa -P "" -f /etc/ssh/ssh_host_ecdsa_key
5	ssh-keygen -t ed25519 -P "" -f /etc/ssh/ssh_host_ed25519_key
6	passwd root
7	vi /etc/ssh/sshd_config
8	/usr/sbin/sshd

```

