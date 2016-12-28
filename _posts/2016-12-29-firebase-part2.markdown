---
layout:     post
title:      "Firebase-part1-Realtime Database"
subtitle:   " A new way to build apps"
date:       2016-12-28 12:00:00
author:     "shadowera"
header-img: "img/about-bg.jpg"
catalog: true
tags:
    - android 
    - Firebase
---
# Before
    既然Firebase是做后端数据库起家的，那就自然少不了实时数据库这一块了 
    实时数据库，顾名思义，就是不需要客户端自己去轮询，而是每当监听的区域发生变动时，数据库都会通知客户端去更新，设备将会在毫秒级的时间内接收到数据更新 
    除此之外，考虑到写数据时遇到的无网络连接问题，Firebase的数据库API使用了本地缓存，使得在离线状态下也能保持读写不失败，并且会在网络恢复连接时和服务器进行同步 
    和常见的数据库服务器不同，Firebase使用JSON存储数据，这使得数据结构非常可观

# 数据结构
    
