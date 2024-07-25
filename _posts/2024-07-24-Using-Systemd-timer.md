---
title: Using Systemd timer to keep service running
date: 2024-07-24 11:31:00 -0700
categories: [systemd]
tags: 
Pin:
---

## Overview

This guide explains how to use systemd timer to keep service running. During some sutiuations, you may need to restart the service after some time. or some service is crashing. This is where systemd timer comes in.

I have a test server, I need to monitor mongodb service and keep it always active. Usually, the crontab is a good solution. Today I will show how to use systemd timer to keep the service running.

## Steps

1. **Create systemd service**

    ```bash
    sudo vim /etc/systemd/system/check_mongodb.service
    ```
    ```systemd
    [Unit]
    Description=Check MongoDB Service
    After=network.target

    [Service]
    ExecStart=/home/shells/check_mongo.sh
    ```

2. **Create systemd timer**
   
    ```bash
    sudo vim /etc/systemd/system/check_mongodb.timer
    ```
    ```systemd
    [Unit]
    Description=Run Check MongoDB Service every minute

    [Timer]
    OnBootSec=1min
    OnUnitActiveSec=1min
    Unit=check_mongo.service

    [Install]
    WantedBy=timers.target
    ```

3. **Create check_mongo.sh**

    I have used the following command to create the check_mongo.sh:

    ```bash
    sudo vim /home/shells/check_mongo.sh
    ```

    ```shell
    #!/bin/bash

    LOGFILE="/var/log/check_mongo.log"
    TIMESTAMP=$(date "+%Y-%m-%d %H:%M:%S")

    # Check if MongoDB is running
    if ! pgrep -x "mongod" > /dev/null
    then
        echo "$TIMESTAMP - MongoDB is not running. Starting MongoDB..." | tee -a $LOGFILE
        # Send notification
        curl -d "The MongoDB service has crashed. Please monitor its status." ntfy.sh/mytopic
        # Start MongoDB
        systemctl start mongod
        if [ $? -eq 0 ]; then
            echo "$TIMESTAMP - MongoDB service started successfully." | tee -a $LOGFILE
        else
            echo "$TIMESTAMP - Failed to start MongoDB service." | tee -a $LOGFILE
            # Send status notification
            curl -d "Failed to start MongoDB service, please monitor the status." ntfy.sh/mytopic
        fi
    fi
    ```

4. **Enable and start timer**
   
    ```bash
    sudo systemctl enable check_mongodb.timer
    sudo systemctl start check_mongodb.timer
    ```
5. **Enable and start service**

    ```bash
    sudo systemctl enable check_mongodb.service
    sudo systemctl start check_mongodb.service
    ```