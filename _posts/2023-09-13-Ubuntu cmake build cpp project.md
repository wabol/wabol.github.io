---
title: Ubuntu build cpp project 
date: 2023-09-09 11:11:00 -0700
categories: [Linux]
tags: c++
Pin:
---

### This is a personal project, it may not be suitable for you.

1. #### Install denpendcies

   ```
   apt update && apt upgrade
   apt install -y \
       libunwind-dev  \     
       libboost-all-dev \  
       libevent-dev   \ 
       libdouble-conversion-dev  \ 
       libgoogle-glog-dev  \
       libgflags-dev   \
       libiberty-dev   \
       liblz4-dev \ 
       liblzma-dev \  
       libsnappy-dev  \ 
       make \    
       cmake \
       zlib1g-dev \    
       binutils-dev \     
       libjemalloc-dev \     
       libssl-dev \    
       pkg-config \
       clang \
       g++ \
       build-essential \
       tar  \
       curl  \
       zip  \
       unzip 
   ```

2. #### Build fmt

   ```
   git clone https://github.com/fmtlib/fmt.git
   cd fmt/
   mkdir _build && cd _build
   cmake ..
   make
   make install
   ```

3. #### Build folly

   ```
   git clone https://github.com/facebook/folly.git
   cd folly
   mkdir _build && cd _build
   cmake ..
   make
   make install
   ```

4. #### Install vcpkg 

   ```
   git clone https://github.com/microsoft/vcpkg.git
   ./vcpkg/bootstrap-vcpkg.sh
   ./vcpkg/vcpkg install folly
   ```

   