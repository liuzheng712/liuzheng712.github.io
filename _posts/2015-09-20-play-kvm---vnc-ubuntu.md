---
layout: post
title: "KVM + VNC (ubuntu)"
description: ""
category: 
tags: [KVM]
---
{% include JB/setup %}

之前有进行过ubuntu server的KVM安装，不过似乎没有记录下来...

# 安装gnome-core

个人一直喜欢GNOME做为桌面系统，在此用vnc的时候也希望默认桌面系统是自己熟悉的，如喜欢其他桌面系统相应安装相关桌面系统的包。

    sudo apt-get install gnome-core

# 安装vnc

vnc安装也是一条命令的事情。。。

    sudo apt-get install vnc4server

至此你就可以使用命令`vncserver`来创建你的密码，一切结束后就可以使用vnc相关连接工具进行使用了，我个人使用的是chrome的插件，https://chrome.google.com/webstore/detail/iabmpiboiopbgfabjmgeedhcmjenhbla

当然这样还是不够的，没有对vnc默认桌面进行配置

    vim .vnc/xstartup

在此我注解掉所有原始配置，添加下文

    unset SESSION_MANAGER
    metacity &
    x-terminal-emulator -geometry 800x600+10+10 -ls -title "$VNCDESKTOP Desktop" &
    gnome-settings-daemon &
    gnome-panel &

# 安装KVM及virt管理软件

    sudo apt-get install kvm qemu
    sudo apt-get install virtinst python-libvirt virt-viewer virt-manager

# 配置桥接网卡

    sudo apt-get install bridge-utils

通过`ifconfig`命令我们可以发现网卡多了一个virbr0，这个是装完KVM后自己生成的虚拟网卡

增加一个虚拟网卡br0，让这个网卡和em1进行桥接

    sudo vim /etc/networks/interfaces

在其后追加如下内容

    auto br0
    iface br0 inet dhcp
    bridge_ports em1

保存，重起网卡设置

    sudo /sbin/ifup br0 # 激活br0
    sudo /etc/init.d/networking restart


# 新建虚机

    qemu-img create -f qcow2 ubuntu.img 10G

创建虚机

    sudo qemu-system-x86_64 -m 1024 -had ./ubuntu.img -localtime -net nic,vlan=0,macaddr=ff-ff-ff-ff-ff-01 -net

参考：

<http://www.havetheknowhow.com/Configure-the-server/Install-VNC.html>

<http://blog.fens.me/vps-kvm/>

<http://wiki.ubuntu.org.cn/Kvm_%E7%BD%91%E7%BB%9C%E6%A1%A5%E6%8E%A5%E6%96%B9%E6%A1%88>
