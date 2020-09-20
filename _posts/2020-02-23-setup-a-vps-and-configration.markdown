---
layout: post
title:  "新建VPS并进行一些设置"
date:   2020-02-23 13:22:21
categories: [Life, Computer Science]
tags:   [Life, Vultr, CentOS]
---
今年过了一个难忘的年。。。

珠海跨年之旅受疫情影响，提前结束。

回来之后天天在家，天天陪孩子玩。晚上闲下来发现去年整的profit VPS用起来太慢了，看不存在网站视频也就380p，离1080p差远了。。。

想起来之前的评论系统也没整，还有个cn域名也没用，就想着整个vps自建一个静态blog。。。

## 购买VPS并进行基本设置

其实profit也是够用了，不过眼瞅着快到期了，就想着换一个试试。对比了多家之后选择了Vultr的小VPS，首充送了50美刀，但是只有一个月的有效期，所以干脆买了一堆不同地方的VPS，测试完再删吧。

VPS的具体购买方式网上有很多教程了，各大主机厂商也提供了很多教程，就不赘述了。

Vultr官方提供了很多镜像选择，开始的适合还是和之前一样选择了FreeBSD，后来发现他们家有CenOS 8的官方镜像，就体验了下新版，由于笔记记得都是CentOS下的操作，所以以下介绍都按照CentOS 8来。

### 1. 修改密码并更新系统

VPS建好之后，用控制面板带的Console登录root账户，进行初步设置。

```shell
# Change root passwd
passwd

# Update
yum update
```

### 2. 初步设置SSH并添加常用普通用户

更新完系统后就要进行ssh设置以便客户端登录操作了，具体步骤如下：

```shell
vim /etc/ssh/sshd_config
```
首先修改了一部分配置。

```vim
# /etc/ssh/sshd_config
Port SSH_PORT_NUMBER  # 修改默认ssh端口
PermitRootLogin no # 禁止root用户登录
```

**注意** 修改端口号后需要修改对应的防火墙设置。

```shell
# SELinux 设置
semanage port -a -t ssh_port_t -p tcp SSH_PORT_NUMBER #SSH_PORT_NUMBER是上一步设置的ssh端口号
# firewall 设置
firewall-cmd --zone=public --add-port=SSH_PORT_NUMBER/tcp --permanent
# 使能firewall设置
firewall-cmd --reload
# 重启ssh服务
systemctl restart sshd  #restart ssh server
```

然后添加一个普通用户并设置密码。CentOS的adduser居然和useradd是一样的，囧，还是“小恶魔”的adduser好用 - -

```shell
# 添加用户king并添加到wheel组中
useradd king -G wheel -U
passwd king
```

由于禁止了root通过ssh登录，所以可以安装sudo以便刚才添加的用户进行日常维护。（此步骤不是必须的）

```shell
yum install sudo
visudo
```
参照文件注释编辑sudo的配置文件
```vim
# 允许wheel用户组以root身份运行所有命令
%wheel ALL=(ALL)
```
### 3. 上传SSH公钥并完善SSH设置

上述设置完成后，即可从客户端上传公钥文件,并用ssh方式登录服务器。

```shell
# 上传公钥文件
ssh-copy-id -i .ssh/xxx.pub king@SERVER_IP_ADDRESS -p SSH_PORT_NUMBER
# ssh登录至服务器
ssh  -i .ssh/xxx.pub king@SERVER_IP_ADDRESS -p SSH_PORT_NUMBER

上述测试成功后，即可禁用密码登录，并关闭浏览器管理面板登录的Console。

```vim
# /etc/ssh/sshd_config
# 禁用密码登录
PasswordAuthentication no
```
不放心密钥的可将ssh登录方式改为密钥+密码，然后重启ssh服务即可。

```shell
systemctl restart sshd
```

到这里服务器初步设置就基本完成了，之后的所有步骤都从客户端完成。

## 其他设置

**再次提醒** 以下的所有操作均在客户端完成，ssh登录方式如下：

```shell
ssh  -i .ssh/xxx.pub king@SERVER_IP_ADDRESS -p SSH_PORT_NUMBER
```

### 1. 安装fail2ban
安装fail2ban并设置把一切尝试ssh登录错的都给禁半天。

```shell
sudo yum install -y epel-release
# 安装fail2ban
sudo yum install fail2ban
# 修改fail2ban设置
sudo vim /etc/fail2ban/jail.local
```
```vim
# /etc/fail2ban/jail.local
[DEFAULT]
# Ban hosts for 12 hour:
bantime = 43200
findtime = 600
maxretry = 2

# Override /etc/fail2ban/jail.d/00-firewalld.conf:
banaction = iptables-multiport

[sshd]
enabled = true
```

启动fail2ban服务并添加默认启动。

```Shell
sudo systemctl start fail2ban
sudo systemctl enable fail2ban
```

### 2. ~~安装代理软件并设置~~

该部分笔记都有记录，不便于发表在blog中，概述如下：

+ 在/etc/yum.repos.d中添加源，需要注意系统版本对应的源不同
+ 更新并安装主程序和混淆插件
+ 修改默认配置文件
+ 测试安装与设置（由于使用多端口设置，不能使用ss-server，应该使用ss-manager)
+ 创建自启动服务（需要修改**\*.service文件中的User等字段，和ExecStart等）
+ 添加防火墙规则

```shell
# Update firewall rule
firewall-cmd --zone=public --add-port=xxx1-xxx3/tcp --permanent
firewall-cmd --zone=public --add-port=xxx1-xxx3/udp --permanent
firewall-cmd --reload
sudo systemctl start YOURSERVICE
sudo systemctl enable YOURSERVICE
```

### 3. 添加BBR支持

具体是啥可自行Google，不过Vultr的CenOS 8默认已经开啦。就从之前的笔记粘贴过来啦。

开启BBR详细步骤如下，如果内核版本低于4.9需要更新内核，不想编译的添加源就行，老Gentoo用户表示不想多说什么了，哈哈

```shell
uname -r
# 版本号低于4.9需要更新内核并重启
sudo rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
sudo rpm -Uvh http://www.elrepo.org/elrepo-release-7.0-2.el7.elrepo.noarch.rpm
sudo yum --enablerepo=elrepo-kernel install kernel-ml -y
# 默认启动项（grub2）
sudo egrep ^menuentry /etc/grub2.cfg | cut -f 2 -d \'  # 确保第一行就是新版内核并且版本号高于4.9
sudo reboot
```

版本号高于4.9的可直接开启，具体步骤如下：

```shell
# sudo echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf
echo 'net.core.default_qdisc=fq' | sudo tee -a /etc/sysctl.conf
# sudo echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf
echo 'net.ipv4.tcp_congestion_control=bbr' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

测试一下吧
```shell
sudo sysctl net.ipv4.tcp_available_congestion_control
# output should be:
net.ipv4.tcp_available_congestion_control = reno cubic bbr

sudo sysctl -n net.ipv4.tcp_congestion_control
# output should be:
bbr

lsmod | grep bbr
#output should be
tcp_bbr xxxx xx
```

## 后记

此次VPS设置设置说了个七七八八，就当是个记录吧。还有在VPS部署Git服务器等部分笔记，整理整理回头再发吧。。。


