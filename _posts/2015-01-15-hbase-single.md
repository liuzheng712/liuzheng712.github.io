---
layout: post
title: "HBase 单机运行出的神奇的问题"
description: "regionserver.HRegionServer: No master found; retry"
category: 
tags: [HBase]
---
{% include JB/setup %}

困扰一周的问题！！！

    regionserver.HRegionServer: No master found; retry

这个错误我是百思不得其解！

解决方法，修改 $HBASE_HOME/conf/regionservers , 里面要保证三行以上吧

出错原因

由于实验室环境条件限制，暂时无法使用多机进行hadoop配置，我就腾出一台服务器。。。

所以regionservers我这里只配了一个，这就是坑爹所在。容后查文档研究。。。

应该是xxx选举所致
