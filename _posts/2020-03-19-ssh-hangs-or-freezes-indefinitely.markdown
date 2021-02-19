---
layout: post
title:  "ssh挂起的解决办法"
date:   2020-03-19 22:15:00
categories: [WSL]
tags:   [Lifelong learning, WSL2, ssh]
---

最近从WSL切换到WSL2之后发现无法ssh到VPS。
使用```-vv```选项查看，发现最后挂起前提示：
```bash
debug2: channel 0: open confirm rwindow 0 rmax 32768
```
然后Google了一下，发现这其实是路由器的问题。
具体原因就是：
当某些路由器位于NAT之后使用OpenSSH时。
在会话建立期间，在输入密码或交换密钥后，OpenSSH会在IP数据报中设置TOS（服务类型）字段。
部分路由器会被阻塞，导致SSH会话被无限期挂起。表现上来看就是ssh命令很少起作用或根本不起作用。

要解决这个问题其实也简单，临时方式就是在ssh命令后加选项```-o IPQoS=0x00```。

长期方式就是修改配置文件，修改```~/.ssh/config```或者```/etc/ssh/ssh_config```都行。

```vim
# vim /etc/ssh/ssh_config
Host *
  IPQoS 0x00
```

P.S. 另一个不推荐的做法就是使用```ProxyCommand nc %h %p```。
