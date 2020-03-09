---
layout:     post
title:      "Effective-java-3rd"
subtitle:   " 01 - 考虑使用静态工厂方法替代构造方法 "
date:       2020-03-09 14:00:00
author:     "linjiankai"
header-img: "img/post-bg-effective-java-3rd.png"
catalog: true
tags:
    - 学习
    - effective-java
---
一个类允许客户端获取其实例的传统方式是提供一个公共构造方法。除此之外，还有另外一种方法，提供一个公共静态工厂方法——一个返回类实例的静态方法。

## 静态工厂方法的优点
* 有对应的方法名（可以用来描述被返回的对象）
