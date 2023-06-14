---
title: Frp service Auto start
date: 2023-04-25 12:11:00 -0800
categories: [Linux]
tags: Frp
---

Vim frcp_start.sh file

```shell
#!/bin/bash

cd /home/frp/
# Define variables for frpc URL and configuration options
FRPC_URL="https://github.com/fatedier/frp/releases/download/v0.37.1/frp_0.37.1_linux_amd64.tar.gz"
FRPC_CONFIG="[common]\nserver_addr = 192.168.1.1\nserver_port = 7000\ntoken = DfA%87iU\nlog_file = ./frpc.log\n\n[tttssh]\ntype = tcp\nlocal_ip = 127.0.0.1\nlocal_port = 22\nremote_port = 6000"

# Check if frpc is already running, if Ture, exit script.

pid=$(ps -C frpc -o pid=)
if kill -0 $pid; then
    echo "Process is still running."
    exit 0
fi

# Check if frpc is already installed，if [ -e "frpc" ] means' check the file frpc exists in current directory.
if [ -e "frpc" ]; then
    echo "frpc is already installed."

    # Create frpc.ini configuration file
    echo -e $FRPC_CONFIG > frpc.ini
    nohup ./frpc -c frpc.ini >> /home/frp/nohup.log 2>&1 &
else
    # Download and extract frpc binary
    wget -O - $FRPC_URL | tar xzvf -

    # Move frpc to directory frp
    mv frp_0.37.1_linux_amd64/frpc .

    # Create frpc.ini configuration file
    echo -e $FRPC_CONFIG > frpc.ini

    # Start frpc using configuration file
    nohup ./frpc -c frpc.ini >> /home/frp/nohup.log 2>&1 &
fi

```

