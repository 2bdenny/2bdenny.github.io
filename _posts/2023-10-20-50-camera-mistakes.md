---
layout: post
title:  "Notes of GDC-Talk 50 Camera Mistakes (50个错误的相机用法)"
date:   2023-10-20
categories: blogs
author: 2bdenny
---

---
# [GDC Talk](https://www.gdcvault.com/play/1020460/50-Camera) 50 Camera Mistakes (50个错误的相机用法)
- 当其他相机（固定、第一人称）可用的时候，用了动态相机
- 关卡和相机不匹配
- 使用global坐标和四元数来记录相机状态
	- yaw pitch roll； distance from pivot； lateral、vertical offsets for framing、 fov
- 使用不合适的相机距离导致看不到主角
- 障碍从边上打断视线
- 当玩家想绕过障碍的时候把相机移太远
- 让玩家把相机推进墙里
- 让多条规则/force抢着移动相机
	- 可以把规则按照自由度（上面的7类）分类，然后优先级排序
- 保留窄墙打断视线
- 让相机跟窄墙相交
- 把山和墙当作一回事来做回避
	- 应该分开的
- 在occludes的同时还在swing相机
- 让相机的near clipping plane跟主角相交
- 所有角度都使用一样的distance
- 鱼眼镜头和普通镜头使用一样的fov
	- 例如朝天空看的时候应该把fov调大一点
- pitch、distance和fov单独改变
- 在主角穿过不透明区域的时候不做cutting
- 在cut的时候改变按键操作的对应调整
- 破坏玩家的方向感
- 违反180度规则（不要在脚底和头顶旋转相机）
- 只focus角色
- 让玩家完全控制相机
- 当玩家在跑的时候让yaw alone
- 让玩家感到很难判断距离
- 当角色爬墙的时候也朝前看
- 当角色跑在一个slope上的时候也让相机水平
	- 应该调整pitch
- 错误使用 rule of thirds
	- 当角色不在中间的时候，图像往往更好看
	- 一般当角色站定的时候才适用这条规则，然后角色跑起来的时候还是让他在中间
- 在地上和空中的时候用同样的逻辑
	- 跳起来或者在空中的时候，一般玩家想看落点，所以应该朝下看
- 完全依赖相机procedural行为
	- raycast应该只做limit/hint，而不是决定相机出现在哪里
- 让玩家觉得他们迷路或者失去方向
	- recenter/一点点相机方向提示
- 旋转相机才能看到附近的targets
- 移动相机去看远处的目标
- 让主角的身体occludes来看到target
- 让玩家控制相机，然后又突然不让了
	- 玩家控制应该可以override一些轨道
- 在玩家转完相机之后马上hint，让相机看向target
	- 可以delay或者干脆不要
- 不让experts玩家探索场景
- 不提供inverted控制
	- 有的玩家是左撇子
- 对于突然的特殊输入马上反应，导致画面不够smooth
- 使用线性的curve
	- analog sticks
	- 最好可以提供曲线的调整
- 让相机pivot漂移太远
- 使用一个很小的fov
	- 画面会移动太快
- 迅速改变fov
- shake相机
	- 应该给一个选项来禁止shake相机
- 让相机跟随角色的角度而上下抖动
	- 如果一定要有，也应该有选项关掉
- 角色jump的时候，相机跟着角色一起移动或者转向
- 快速移动到一个相机位置
	- 比较好的是直接做一个camera cuts
- 保持pitch速度直到pitch limits
	- 最好可以减速（curve调整）
- 为VR设备而开发，并且还保留这些feature给普通玩家
- 只让少量的人测试
	- 熟悉游戏类型的人测试不会有太多帮助
	- 不熟悉游戏的人能给很多新反馈
- 实现一个通用的constraint solver