---
title: Frp configuration
date: 2023-03-22 22:11:00 -0800
categories: [Linux]
tags: 
---

1. Windows client start

   start.bat in the frp directory

```
if "%1"=="hide" goto CmdBegin
start mshta vbscript:createobject("wscript.shell").run("""%~0"" hide",0)(window.close)&&exit
:CmdBegin
@echo off
:home
frpc -c frpc.ini
goto home
```

