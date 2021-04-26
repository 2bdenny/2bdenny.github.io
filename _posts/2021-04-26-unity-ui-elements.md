---
layout: post
title:  "Unity UIElements Notes - Unity UIElements手册笔记"
date:   2021-04-26
categories: blogs
author: 2bdenny
---

{:toc}

> 开发指引

# Visual Tree
## VisualElement 所有组件的基类，大概类似<div>
- 包含styles/layout/本地变形/事件handler等等
- connectivity：通过测试ve有没有panel属性，来判断ve有没有连接到界面上呈现在屏幕上
- ve按照parent-child，从上到下的顺序依次draw
- 定位：
    + World：相对于panel
    + Local：相对于ve自己
    + layout.position相对于parent的位置
    + layout.transform相对于parent的位置
    + VisualElementExtensions提供三种转换
        - WorldToLocal
        - LocalToWorld
        - ChangeCoordinatesTo (ve间local转换)
# Layout Engine
- container按照从上到下的顺序自动分布children
- position的rect默认包括了它children的rect, 可以改掉
- 有text的ve默认用text size计算size, 可以改掉
- 如何使用
    + width/height来定义size
    + flexGrow属性让ve的size动态变化
    + flexDirection:row定义一个水平排布layout
    + 相对位置
    + position设置为absolute相对于parent定位
# UXML
- 可以自定义new elements, prefix, uxml name(默认是ve), factory, 默认属性值等
## UXML模板
- ve提供如下通用属性
    + name
    + picking mode
    + tabindex
    + focusable
    + class
    + tooltip
    + view-data-key
- uss到uxml的使用类似css到html
- uxml引用
    + src: 相对uxml文件路径(../uss/xx.uss), 或者直接src=template.uxml; 或者相对于project: "/Asset/..."; "project:/Assets/..."
    + path: Resources或者EditorDefaultResources
        - Resources: 不要后缀, 如path="template"引用Assets/Resources/template.uxml
        - EditorDefaultResources: 要后缀, 如path="template.uxml"引用"Assets/EditorDefaultResources/template.uxml"
- 现有可用uxml组件[列表](https://docs.unity3d.com/Manual/UIE-ElementRef.html)
- UQuery: jquery...

# USS
- Selector(ve选择器)
    + 跟jquery很相似
    + uss里可以引用图片资源
 
# Event System
## Event dispatcher
- Event target：发出事件的ve
- focus order：默认情况tree进行DFS决定
- Event.currentTarget (callback registered ve)
- Event.target (event trigger ve)
## Event handler
- default action (virtual function): target上执行
- callback: 事件传递序列里只要遇到都会调用
- 两种方法都适用的时候推荐使用前者
    + 希望在propagation phase时,执行完target的callbacks后立刻执行, overriding EvecuteDefaultActionsAtTarget()
    + 希望dispatch结束后执行, override ExecuteDefaultActions() [Preferred]
- 穿透事件时可以使用callback, 并自定义事件顺序
## Event synthesizer：自定义的事件创建和分发
- 可以自定义发送系统事件(UnityEngine.Event)
## Event Types
- 总之很多

# Built-in Controls
- 总之也很多

# Binding
- 用过

# Supporting IMGUI
- 支持的

# ViewData Persistence
- 用于restart或者重开窗口时保持上一次状态