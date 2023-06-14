---
title: Linux process check and restart 
date: 2022-12-01 22:11:00 -0800
categories: [Linux]
tags: 
---

I have a test server, some times I need to monitor one test service and keep it always active. I plan to use crontab and shells to make a small solution to fix this problem.

### 1.Check serivce status

Service_check.sh 

```shell
#!/bin/bash

service_name_process=$(pgrep -f service_name)
# We can use "ps -ef |grep 'service_name' | grep -v 'grep' | awk '{print $1}'" 

# Check main programme status
if [[ $service_name_process = "" ]];then
cd /path &&
# /path replace your own path
sh service_name_start.sh
# start.sh use your service start command 
# like "nohup /path/service /path/service.ini >> /path/service.log 2>&1 &"

echo service start successful!
else
echo service is running!
fi
```

### 2.Timed execution through crontab

`crontab -e`

shellcheck is a good service for shells. [https://www.shellcheck.net/](https://www.shellcheck.net/ ) 