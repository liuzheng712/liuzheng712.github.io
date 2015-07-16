---
layout: post
title: "微软2016年校招探星夏令营第二题：最多约数问题"
description: ""
category: 
tags: []
---
{% include JB/setup %}

表示对微软不感冒，然后呢也没关注，今天看到同学在玩这题，感觉很好玩，就玩玩哈。

原题：<http://hihocoder.com/contest/mstest2015july1/problem/2>

时间限制:10000ms

单点时限:1000ms

内存限制:256MB

## 描述

Given an integer n, for all integers not larger than n, find the integer with the most divisors. If there is more than one integer with the same number of divisors, print the minimum one.

## 输入
One line with an integer n.

For 30% of the data, n ≤ 103

For 100% of the data, n ≤ 1016

## 输出
One line with an integer that is the answer.

##样例输入

    100

## 样例输出

    60

下面做简单的建模分析：

设定 $ N = 2,3,5,7,11... (Prime Number) $, $ q_i \in N^+ $

根据约数个数计算公式，我们可以得到

$$ 
\Pi_1 ^N ( q_i + 1 )
$$

则我们可以写出如下模型：

$$ 
Max : \Pi_1 ^N ( q_i + 1 ) \\\\
s.t. : \Pi_1 ^i ( N_i ^{q_i} ) < num 
$$

