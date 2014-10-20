---
layout: post
title: "Daily Shell"
description: "Daily Shell"
category: dailyShell
tags: [dailyShell]
---
{% include JB/setup %}

挂载 smb 盘时 mount -t 是 cifs ，不是 smbfs，被坑了一晚上。。。

    mount -t cifs //IP/share /tmp/mountPoint