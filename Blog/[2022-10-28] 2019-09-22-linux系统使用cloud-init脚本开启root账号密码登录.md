# 2019-09-22-linux系统使用cloud-init脚本开启root账号密码登录
---
layout: post

cid: 1534

title: linux系统使用cloud-init脚本开启root账号密码登录

slug: linux-cloud-init-root

date: 2019-09-22 20:29:17

timezone: UTC+8

updated: 2020/12/16 20:12:54

status: publish

author: Joker

categories:
  - Codes

tags:
---

脚本如下：

```
#!/bin/bash
echo root:Joker |sudo chpasswd root
sudo sed -i 's/^#\?PermitRootLogin.*/PermitRootLogin yes/g' /etc/ssh/sshd_config;
sudo sed -i 's/^#\?PasswordAuthentication.*/PasswordAuthentication yes/g' /etc/ssh/sshd_config;
sudo service sshd restart
```

默认密码是：Joker
