---
layout: post
title: 'Java 对象内存布局'
categories: Java
---

Java 对象占用内存是如何分配的。

1. 对象头
 - Mark Word：包含一系列的标记位，比如轻量级锁的标记位，偏向锁标记位等等。在32位系统占4字节，在64位系统中占8字节
 - Class Pointer：用来指向对象对应的Class对象（其对应的元数据对象）的内存地址。在32位系统占4字节，在64位系统中占8字节(如果在64位JVM上开启了压缩指针，那么占用4个字节)；
 - Length：如果是数组对象，还有一个保存数组长度的空间，占4个字节；

32位的对象头:
  32 位JVM上，非数组对象头占用8个字节，数组对象头占用12个字节。
  ![appearance-theme](https://raw.githubusercontent.com/xiaokuicui/xiaokuicui.github.io/master/assets/images/idea-appearance-theme.png)
64位的对象头:
  64 为JVM上，非数组对象头占用16个字节，数组对象占用20个字节。如果开启压缩指针的话，非数组对象头占用12个字节（8 + 4），数组对象头占用16个字节（8 + 4 + 4）
  ![appearance-theme](https://raw.githubusercontent.com/xiaokuicui/xiaokuicui.github.io/master/assets/images/idea-appearance-theme.png)


2. 实例数据
3. 内存填充
