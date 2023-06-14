---
layout: post
tag: [Ubuntu]
category: 
title: Ubuntu install Qbittorrent-box

---

**Install add-apt-repository**  

```csharp
sudo apt-get update && sudo apt-get install software-properties-common -y
```

**Add qbittorrent-nox source** 

```csharp
# qBittorrent stable
sudo add-apt-repository ppa:qbittorrent-team/qbittorrent-stable
```

```csharp
# qBittorrent unstable
sudo add-apt-repository ppa:qbittorrent-team/qbittorrent-unstable
```

**Install qbittorrent-nox**

```csharp
sudo apt-get update && sudo apt-get install qbittorrent-nox -y
```

**Setting start-up**

```csharp
sudo apt-get install vim -y && vim /etc/systemd/system/qbittorrent-nox.service
```

Most of the time just use 'vim', no need to install vim

```csharp
[Unit]
Description=qBittorrent-nox
After=network.target
[Service]
User=root
Type=simple  #forking
RemainAfterExit=yes
ExecStart=/usr/bin/qbittorrent-nox -d
[Install]
WantedBy=multi-user.target
```

In the Ubuntu server 20, the "Type=simple", if you use forking when you start qbittorrent you should be failed.

```undefined
sudo systemctl daemon-reload
```

**Start, stop, enable and check status**

```undefined
sudo systemctl start qbittorrent-nox
```

```undefined
sudo systemctl stop qbittorrent-nox
```

```bash
sudo systemctl enable qbittorrent-nox
```

```undefined
sudo systemctl status qbittorrent-nox
```

Username: admin 

Password: adminadmin

Url: http://ip:8080

