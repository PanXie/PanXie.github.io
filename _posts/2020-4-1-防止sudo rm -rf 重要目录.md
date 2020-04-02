---
layout:     post
title:      防止sudo rm -rf 重要目录
subtitle:   
date:       2019-4-1
author:     XP
header-img: img/problems.jpg
catalog: true
tags:
    - safe-rm
---

## 防止sudo rm -rf 重要目录

ubuntu上误删了/usr，导致很多命令不能使用:sudo, scp, apt, 系统自带的python.... 


### 1, 从其他服务器拷贝一份 ？

虽然账户有sudo权限，但是因为sudo命令不在了，无法将文件拷贝到/usr下面.`sudo su`进入root? ubuntu安装时没有设置root密码，那么root的密码是一个谁也不知道的玩意，也进不去了，只能重装系统了。

### 2，safe-rm
重装完后，马上安装safe-rm, safe-rm可以将一些重要的目录,比如/ , /bin, /usr, /home ... 写在配置文件里，执行sudo rm -rf时，如果所要删除的目录在配置文件列表中，则直接跳过。

安装:
```
sudo apt install safe-rm
```

配置文件/etc/safe-rm.conf:  
```
/
/bin
/boot
/dev
/etc
/home
/initrd
/lib
/proc
/root
/sbin
/sys
/usr
/usr/bin
/usr/include
/usr/lib
/usr/local
/usr/local/bin
/usr/local/include
/usr/local/sbin
/usr/local/share
/usr/sbin
/usr/share
/usr/src
/var
```

测试一下：  
```
$ sudo rm -rf /var
safe-rm: skipping /var
```

如果需要新增目录，添加到/etc/safe-rm.conf即可.

