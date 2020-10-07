---
layout: post
title:  "使用yum-cron自动更新CentOS系统"
date:  2020-10-07 22:25:47
categories: [Computer Science]
tags:  [VPS, Linux, CentOS]
---

之前（大约十年前还在读书时）因为经常更新系统所以手写过crontab文件以实现自动更新。

最近时常登陆VPS去更新系统就想起这事，结果一查发现CentOS有一种简单的方法——就是使用yum-cron。

## 安装yum-cron
使用yum-cron的第一步当然毫不例外是安装 - -

就像安装其他应用一样

```bash
sudo yum install yum-cron
```

## 设置

参照说明编辑```/etc/yum/yum-cron.conf```。

```vim
# sudo vim /etc/yum/yum-cron.conf
apply_updates = yes  # 确保更新被安装
exclude = kernel* mysql*  # 自动更新排除的软件包
```

## 启用
编辑完成后，启用服务以使自动更新生效。
```bash
sudo systemctl start yum-cron
```

