---
layout: post
title:  "UE Knowledge Base"
date:   2023-06-14
categories: blogs
author: 2bdenny
---

---
### Actor
- [怎么检查对象的类型](https://forums.unrealengine.com/t/how-to-check-if-an-actor-is-from-a-certain-class/278582/5?u=2bdenny)
- [UE里面动态创建Actor然后逐个add component跟Unity里面new GameObject不一样](https://blog.csdn.net/qq_16756235/article/details/53157914)

### Audio - 音频
- [怎么在切换关卡时保留Actor, 以及怎么让WWise的Listener Actor一直存在于关卡中](https://www.cnblogs.com/lingchuL/p/14751703.html#5184301)

### Blueprint - 蓝图
- [怎么定义多output pin的蓝图节点](https://forums.unrealengine.com/t/how-to-define-a-function-with-multiple-return-values-as-blueprint-in-c/16934/3?u=2bdenny)
- [蓝图怎么检测null](https://forums.unrealengine.com/t/blueprints-null-actor/277986/2?u=2bdenny)

### Camera - 相机
- [UE相机基础和使用](https://zhuanlan.zhihu.com/p/564571102)
- [一种推荐的自建CameraManager模块方案](https://forums.unrealengine.com/t/setting-the-active-camera-on-an-aplayercontroller/417122/3?u=2bdenny)

### CDO
- [怎样可以让继承UObject的类不创建CDO](https://forums.unrealengine.com/t/how-do-i-prevent-instances-of-uobjects-from-being-created-automatically/345850/2?u=2bdenny)
- [怎样让Subsystem不要Tick两次](https://forums.unrealengine.com/t/subsystem-tick-always-called-twice-per-frame-what-am-i-doing-wrong/695649/3?u=2bdenny)

### Compileing
- [VS编译]()
  - 应该选择 "Development Editor"

### Crash
- [Gpu crash/d3d device removed error](https://forums.unrealengine.com/t/gpu-crash-d3d-device-removed-error/514360/4?u=2bdenny)
- [UE里面的RiderLink插件编译报错]()
  - 添加两条pragma
```
#pragma warning(push)
#pragma warning(disable : 4250)
```
- [重命名导致引用丢失]()
  - 在`DefaultEngine.ini`里面加上对应的redirect就可以了,不用一个个修复

### Log
- [怎么定义自己的Log分类](https://forums.unrealengine.com/t/where-do-i-put-declare-log-category-extern/302371/3)

### Sequence
- [官方示例Slay Animation](https://www.unrealengine.com/en-US/spotlights/mold3d-studio-to-share-slay-animated-content-sample-project-with-unreal-engine-community)

### Streaming
- [怎么在切换关卡时保留Actor](https://forums.unrealengine.com/t/keeping-actors-between-levels/3085?u=2bdenny)
- [使用OpenLevel蓝图节点切换关卡导致蓝图Trigger失效](https://forums.unrealengine.com/t/opening-level-with-trigger-not-working-on-overlap/508767/2?u=2bdenny)
- [Level切换的回调](https://forums.unrealengine.com/t/how-can-i-execute-code-immediately-after-loading-a-non-streaming-level/351320?u=2bdenny)

### Tools
- [一个开源的icon库](https://github.com/MahApps/MahApps.Metro.IconPacks)
- [怎么把UE的蓝图做成网页](https://blueprintue.com/)
- [让Confluence支持自己插入代码](https://marketplace.atlassian.com/apps/1215813/iframes-for-confluence?hosting=server&tab=support)
