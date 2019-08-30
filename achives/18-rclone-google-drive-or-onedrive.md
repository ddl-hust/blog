---
title: vps 挂载 OneDrive 或 Google drive
date:
---

自用的moto z play 的bootloader还是用的Android7.0的，如果刷入Android9.0的ROM会警告bootloader版本太低，想要安全刷入高版本的bootloader，找到了官方的固件，但要下载1.7GB,但为了仅仅几MB的bootloader.img 却要下载1.7GB的整个固件，使用月租流量包的我实在穷了。将固件保存到Google drive，然后VPS上挂载Google drive，再解压到vps目录上，再下载下来。完美解决！
使用gcp创建微型实例，磁盘空间为10GB，空间太小，遂想到使用网盘来增加空间。
