---
layout: post
tag: 
title: Windows get ip
---

#### Windows cmd IP 

**Get ipv6**

```
@echo off
SET IP=
ipconfig /all|FINDSTR "IPv6">ipv6.txt
SETLOCAL EnableDelayedExpansion
FOR /F "delims=" %%a IN (ipv6.txt) DO (
    SET IP=%%a
    ECHO !IP:~37,-5!>>ip.txt
)

ENDLOCAL
del ipv6.txt
type ip.txt
SET IP=
pause
```

`ECHO !IP:~37,-5!>>ip.txt` save ever line 37 to 5 from last to ip.txt

**Cmd get 1st line** 

```
@echo off

if exist ip.txt (
rem get 1st lines

  set n=1

  SetLocal EnableDelayedExpansion

  for /f "delims=" %%i in (ip.txt) do (

    echo %%i

    if !n! equ 1 (

      echo %%i > 1.txt

   ) else (

      echo not matching...

    )

    set /a n=!n!+1

  )

) else (

  echo not found ip.txt

)
```

**Get the content starting from 3rd line**

```
@echo off

if exist Objs.out (

rem get lines from the 3rd line

  set n=1

  SetLocal EnableDelayedExpansion

  for /f "delims=" %%i in (Objs.out) do (

    echo %%i

    if !n! geq 3 (

      echo %%i >> 0.txt

   ) else (

      echo not matching...

    )

    set /a n=!n!+1

  )

) else (

  echo not found Objs.out

)
```



