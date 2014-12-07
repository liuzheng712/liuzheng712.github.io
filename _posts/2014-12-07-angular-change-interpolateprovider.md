---
layout: post
title: "Angular change interpolateProvider"
description: ""
category:
tags: [Angular]
---
{% include JB/setup %}

修改Angular的标识符，避免和Django冲突

    xxx.config(function($interpolateProvider){
      $interpolateProvider.startSymbol('{[{');
      $interpolateProvider.endSymbol('}]}');
      })
