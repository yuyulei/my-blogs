---
title: "Samba 使用简介"
date: 2018-08-29T17:02:43+08:00
archives: "2018"
tags: ["docker", "CIFS", "SMB"]
author: Yu Yulei
---

介绍如何建立一个 samba 服务器; 如何根据本地的网络共享文件; 尝试多种实践：1.客户端访问; 2.Mac访问; 3.docker。

<!--more-->

# Samba 使用简介

介绍如何建立一个 samba 服务器; 如何根据本地的网络共享文件; 尝试多种实践：1.客户端访问; 2.Mac访问; 3.docker。

## 安装部署

### 环境

```
root@csgpu5:/home/yuyulei# lsb_release -a
No LSB modules are available.
Distributor ID: Ubuntu
Description:    Ubuntu 16.04.2 LTS
Release:    16.04
Codename:   xenial
root@csgpu5:/home/yuyulei# uname -a
Linux csgpu5 4.8.0-36-generic #36~16.04.1-Ubuntu SMP Sun Feb 5 09:39:57 UTC 2017 x86_64 x86_64 x86_64 GNU/Linux
```

### 安装流程

#### install

```
sudo apt update
sudo apt install samba

root@csgpu5:/home/yuyulei/sambashare# samba -V
Version 4.3.11-Ubuntu
```

#### mkdir

需要注意的地方：
* 首先这个 username 必须已经存在系统当中，否则 samba 认为这个 user 是不安全的。
* 该 user 有 samba 文件目录读写的权限

```
mkdir /home/<username>/<share_dir>
```
#### edit smb.conf

在 `/etc/samba/smb.conf` 最后面加上下面的配置

```
[<share_dir>]
    comment = Samba on Ubuntu
        path = /home/<username>/<share_dir>
            read only = no
                browsable = yes
                ```

#### restart

```
sudo service smbd restart
```

### 实践

samba 通过 TCP 端口 445 通信, 确保防火墙未阻止。

#### smbclient

```
root@cs46:~# sudo apt-get install smbclient

...

root@cs46:~# smbclient -: //10.200.20.93/sambashare -U yuyulei
WARNING: The "syslog" option is deprecated
Enter yuyulei's password:
Domain=[WORKGROUP] OS=[Windows 6.1] Server=[Samba 4.3.11-Ubuntu]
smb: \> ls
  .                                   D        0  Wed Aug 29 14:42:26 2018
    ..                                  D        0  Wed Aug 29 14:42:26 2018
      test.json                           N       34  Wed Aug 29 14:41:55 2018
        smb.conf                            N     9676  Wed Aug 29 14:42:19 2018

                230619284 blocks of size 1024. 212963236 blocks available
                smb: \>
                ```

#### Mac

点击 `Finder` -> `Go` -> `Connect to Server`, 输入 `smb://<ip>/<share_dir>` 即可看到文件目录。

#### docker 


测试环境需要两台机器：一台作为 samba 服务器, 另一台作为 docker host.

在 docker host 上挂在 samba 文件系统目录，

```
mount -t cifs //10.200.20.93/sambashare /home/yuyulei/samba_test -o vers=3.0,username=yuyulei,password=yuyulei,dir_mode=0777,file_mode=0777

// check 
df -h

...
//10.200.20.93/sambashare  220G   17G  204G   8% /home/yuyulei/samba_test
```
获取目录信息

```
root@cs29:/home/yuyulei/samba_test# pwd
/home/yuyulei/samba_test
root@cs29:/home/yuyulei/samba_test# la
huangbin.conf  smb.conf  test.json
```

启动 docker

```
docker run -d -v /home/yuyulei/samba_test:/home/yuyulei --name yuyulei dora.qiniu:5000/1380538768/stupid-server:v3 

root@cs29:/home/yuyulei/samba_test# docker ps
CONTAINER ID        IMAGE                                         COMMAND             CREATED             STATUS              PORTS               NAMES
ffb5b038b048        dora.qiniu:5000/1380538768/stupid-server:v3   "./stupid-server"   5 seconds ago       Up 4 seconds                            yuyulei
```

登录 container

```
root@cs29:~# docker exec -it yuyulei /bin/bash

root@ffb5b038b048:/home/yuyulei# ls /home/yuyulei/
huangbin.conf  smb.conf  test.json

root@ffb5b038b048:/home/yuyulei# touch a
touch: cannot touch 'a': Permission denied

root@ffb5b038b048:/home/yuyulei# cat test.json
{
        "name":"yuyulei",
            "age": 18
}
```

**权限问题:**

根据决定性要求排序：

* samba 服务器允许的读写权限
    * 在修改 `/etc/samba/smb.conf` 文件时候，可以对指定目录设置 `read only` 和 `writeable` 参数来控制权限 
        * 不同版本下, 上述两个参数有所差异
        * docker host 上挂载(mount) samba 服务的权限时候设置 
            * mount -o rw
                * -o `dir_mode=0777,file_mode=0777`
                * docker run -v 参数指定权限, 比如说 `-v /home/yuyulei/samba_test:/home/yuyulei:rw`

## 参考文献

References:

https://docs.docker.com/storage/volumes/ 
https://tutorials.ubuntu.com/tutorial/install-and-configure-samba#0
https://askubuntu.com/questions/31147/how-to-grant-write-permissions-in-samba
