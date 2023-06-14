---
title: Ubuntu setting desktop resolution in Hyper-V
date: 2023-04-17 22:11:00 -0800
categories: [Linux]
tags: Ubuntu
---



1. ###### adjust grub in the /etc/default/grub

   ```shell
   sudo vim /etc/default/grub
   ```

   Change value of GRUB_CMDLINE_LINUX_DEFAULT with below.

   ```
   GRUB_CMDLINE_LINUX_DEFAULT="quiet splash video=hyperv_fb:3840x2160"
   ```

   Replace '3840x2160' to your resolution.

2.  ###### Then run below commands

   ```shell
   sudo update-grub
   
   sudo apt install linux-image-extra-virtual
   
   sudo reboot
   ```



