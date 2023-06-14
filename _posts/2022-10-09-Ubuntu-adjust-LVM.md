---
title: Ubuntu adjusts space that uses LVM
# author: Daxia
date: 2022-10-09 22:25:00 +0800
categories: [Ubuntu]
tags: [Ubuntu]

---

My MongoDB can't start today. My server is Ubuntu 22. When I checked the logs that said 'No space left on device'. I guess maybe when installling Ubuntu they use LVM. Below is adjusted space.

###### 1. Check device

`df -h`

###### 2. Check LVM information

`vgdisplay`

###### 3. Adjusts device

```
lvextend -L 10G /dev/mapper/ubuntu--vg-ubuntu--lv      # adjusts 10G
lvextend -L +10G /dev/mapper/ubuntu--vg-ubuntu--lv     # add 10G
lvreduce -L -10G /dev/mapper/ubuntu--vg-ubuntu--lv     # reduce 10G
lvresize -l  +100%FREE /dev/mapper/ubuntu--vg-ubuntu--lv   # use percentage
```

You can use any of the four ways to adjusts your device.

###### 4. Resize device 

`resize2fs /dev/mapper/ubuntu--vg-ubuntu--lv`

###### 5. Double check

`vgdisplay`

`df -h`

All set.

