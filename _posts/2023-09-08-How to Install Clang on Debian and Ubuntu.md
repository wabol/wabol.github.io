---
title: How to Install Clang on Debian and Ubuntu
date: 2023-09-08 11:11:00 -0700
categories: [Clang]
Pin:
---

#### Notice: I am install Clang on Debian 12 and Ubuntu 22.04.

1 Using the APT Package Manager

There execute the given command first, which updates the system packages and refreshes the APT packages list.

`sudo apt update && sudo apt upgrade`

2 Install Clang on server

`sudo apt install clang`

3 Check the version

`clang --version`

4 Utilize the LLVM repository script to perform an upgrade for Clang

`wget https://apt.llvm.org/llvm.sh`

5 Modify the file have execute permissions

`chmod +x llvm.sh`

6 Select the specific version you desire

`sudo ./llvm.sh <version number>`

for example upgrade to clang 16

`sudo ./llvm.sh 16`



Thanks for below link

[Click]: https://linux.how2shout.com/how-to-install-clang-on-ubuntu-linux/	"Detail"



