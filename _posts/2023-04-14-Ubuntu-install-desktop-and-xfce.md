---
title: Ubuntu install desktop and xfce
date: 2023-04-14 22:11:00 -0800
categories: [Linux]
tags: Ubuntu xfce
---

1. Use tasksel install desktop environment and xfce, if the server don't have tasksel, you should install it.

   `apt install tasksel`

   `tasksel` and `reboot`

   If tasksel not include xfce, please use below command install xfce which contains a few enhancements for the desktop environment:

   `sudo apt install xfce4 xfce4-goodies`

2. Add user to connect to server with VNC

   `adduser www`

3. Add user to the sudo group 

   `usermod -aG sudo www`

4. change to the new user www, install VNC server

   `sudo apt install tightvncserver`

5. run the `vncserver` command to set a VNC access password, create the initial configuration files, and start a VNC server instance:

   `vncserver`

6. You can use vncpasswd to change password

   `vncpasswd`

7. Configuring the VNC Server

   `vncserver -kill :1`

8. Before you modify the `xstartup` file, back up the original

   `mv ~/.vnc/xstartup ~/.vnc/xstartup.bak`

9. Then

   `sudo vim ~/.vnc/xstartup`

   ```
   #!/bin/bash
   xrdb $HOME/.Xresources
   startxfce4 &
   ```

   `chmod +x ~/.vnc/xstartup`

10. **I have been stuck for this step for a long time**,  If you use root installation, starting the vncserver service requires using a user, such as 'www'.

    `sudo -u www vncserver -localhost`

    -localhost means VNC to only allow connections localhost.

11. start a ssh tunnel in you local, 

    `ssh -L 59000:localhost:5901 -C -N -l www your_server_ip`

    then use VNC client connect to the server.