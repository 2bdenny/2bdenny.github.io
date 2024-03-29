﻿---
layout: post
title:  "Unity下误删文件恢复案例"
date:   2021-06-27
categories: blogs
author: 2bdenny
---

---
# 事故还原
项目使用Perforce进行版本管理（在目录映射方便的同时，也被不靠谱的Perforce各种坑，本次案例就是其中一大坑）。
1. 使用开启Perforce插件的Rider打开项目
2. 将A目录下的文件a改名为b
3. 将b剪切到B目录，改名为c
4. 发现文件中部分函数必须放在A目录下，于是将文件拆分为b和c，并将b（与a改名后的b名字相同）剪切回A目录
5. 一通操作大改了b和c文件 
6. 查看Perforce的changelist列表，发现操作记录：
    - c文件被记录为Add
    - a, b文件被记录为a move-delete; b move-add
    - 另有一个b文件被记录为Add (即B文件被记录了两次操作)
7. 手动revert被标记为move-add的b文件, 并选择同时删除文件
8. 发现被记录为Add的b被同时删除, 到这里, **所有b文件的修改直接丢失**

# 解决方案
1. 由于Unity没有开启自动编译, Unity的缓存数据中还留有b文件, 点击可以在Inspector中查看, 
   尝试类似浏览器F12来Debug页面Unity的Editor界面, 结果发现相关预览组件为IMGUIContainer, 
   无法直接复制粘贴内容, 于是先截图留下代码 (几百行, 最坏就是再手打一遍...)
2. 惊喜发现上一次编译时b文件所在工程dll还在, 有dll就可以尝试反编译获取源码
3. 使用ILSpy默认选项反编译该工程dll, 竟然提示失败
4. 最后通过尝试ILSpy的多个选项, 发现关闭部分选项后, 可以反编译到除lambda表达式外的代码
5. 结合反编译代码和代码截图, 恢复文件, 最后大约手敲几十行 (比几百行好多了)
6. 其实还准备了一个最终方案, 就是使用OCR识别代码截图

# 反思
1. 谨慎使用Perforce的Revert(同时删除新增文件)选项
2. 关闭Unity的自动编译(救了命了)
3. 先想好怎么分配代码与功能再上手, 不要写一半疯狂改动文件, Perforce和Perforce插件的土豆电池版变动追踪
   功能一复杂就容易出错