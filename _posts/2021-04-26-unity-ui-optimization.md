---
layout: post
title:  "Unity UI Optimization Tutorial Notes - Unity UI优化教程笔记"
date:   2021-04-26
categories: blogs
author: 2bdenny
toc: true
---

# 渲染细节
- 所有UnityUI的集合体都会在一个Transparent队列里按照次序进行渲染. **每一个多边形的每一个点都会被采样! 即视这个点被其他的不透明多边形所覆盖了也一样会参与计算! 所以UI重叠会消耗性能!**

# The Batch building process (Canvases)
- 就是把代表UI的Mesh进行整合然后生成合适的渲染command的过程.
- 由于这个过程是多线程的, 所以桌面CPU和移动端CPU的性能表现会有很大区别

# The rebuild Process (Graphics)
> 这个过程包括layout和mesh的重新计算, 通过CanvasUpdateRegistry这个类来实现. 当一个Canvas发起一个WillRenderCanvases事件的时候, 就会invoke这个类里的PerformUpdate方法, 这个方法里通过3步来完成rebulid:
1. dirty layout组件通过ICanvasElement.Rebuid方法进行重新计算
2. 遮挡相关的Component(例如Mask等), 通过ClippingRegistry.Cull方法进行重新计算
3. Dirty Graphic重新计算

其中:
- Layout rebuilds: 根据depth来sort所有dirty的layout, 父layout会排在list的前面, 然后按照sorted list的顺序进行重新计算
- Graphic rebuilds: 通过ICanvasElement接口来进行rebuild, 在PreRender阶段进行2步操作:
    1. 如果vertex data是Dirty(e.g. RectTransform被修改了), 则mesh会被rebuilt
    2. 如果material data是Dirty(e.g. texture或者material被修改了), 则Canvas Renderer的material会被更新
    - 注意这一步不需要进行排序相关的操作

# chpt3 profiler
## Unity Profiler
## Frame Debugger
- 把UI组件用一个batch来绘制是一种常用的方法, UI组件之间的overlap会导致batch被打断
- A叠B叠C, 如果A和C使用相同材质, B使用不同材质, 那么因为重叠关系, 就算AC在同一个canvas上, 他们也不能被合并到一个batch里

## Profiler结果分析
- `Canvas.BuildBatch`或者`Canvas::UpdateBatches`占用过多CPU时间: 
    + 原因: 单个Canvas里有过多的CanvasRenderer
    + 推荐: 切分为不同Canvas
- 绘制UI占用过多时间(会提示: fragment shader pipeline是瓶颈)
    + 原因: UI重叠过多
- Graphic Rebuilds占用过多CPU时间 (Canvas.SendWillRenderCanvases, 或者Canvas::SendWillRenderCanvases调用很多)
    + 原因: 可能是某些Graphic Reduild
- 在IndexedSet_Sort或者CanvasUpdateRegistry_SortLayoutList里, WillRenderCanvas占了很大一部分
    + 原因: 给Dirty的Layout排序占用了很多时间, 考虑减少Canvas上的Layout数量(用RectTransform替换, 或者切分Canvas)
- Text_OnPopulateMesh占用过多时间
    + 原因: 生成text mesh占用时间太多
    + 推荐: 去查一下BestFit/取消Canvas, 考虑切分Canvas
- Shadow_ModifyMesh/Outline_ModifyMesh(或者其他xxx_ModifyMesh)
    + Mesh Modifier计算占用时间过多
    + 考虑用静态图片替换这些动态Mesh
- 如果Canvas.SendWillRenderCanvases没有特定的hotspot, 或者每帧都很慢
    + 可能是因为同一个Canvas上包含动态elements和静态elements, 导致Canvas一直在被rebuild, 应该考虑把这两种分开在不同Canvas上

# Chpt 4 Fill-rate, Canvases和input
## 减少渲染UI的GPU压力
- 减少fragment shader的复杂度->`UI Shaders and low-spec devices`
- 减少采样像素个数

> 也就是要: 减少UI重叠

1. 减少不可见UI
    + 最简单的方式: 把相关go设置为disable; 或者Disabling Canvases
    + **注意**: 不要通过设置alpha为0的方式隐藏UI, 因为alpha为0仍然需要被render!!
2. 简化UI结构
    - 不要用blended go来改element颜色, 直接操作材质属性
    - 不要像创建文件夹一样创建go
3. disable不可见的相机输出
    - 但是Screen Space-Overlay不管相机是否active都会被绘制
4. 相机部分可见
    - 相机拍摄的全屏UI可能只有一部分会随着相机拍摄内容而变化(剩下一部分被UI什么的挡住了), 这样的话可以把相机拍摄内容的可见部分保存在RT里
5. 组合UI
    - 可以把多个元素的sprites集合到一个单独atlas, 但是这样会增大atlas的体积. 还可以把多个go合并成1个, 但是这样会导致不可复用, 谨慎操作
6. UI Shader和低端设备优化
    - Unity的UI自带的Shader支持包括masking, clipping和很多其他复杂的操作, 为了支持这些操作, 会影响一部分性能, 如果确定不需要, 那就可以单独写一个简化版的shader
7. UI Canvas rebuilds优化
    - UI元素过多的话, sorting和分析每个元素本身就会占用过多时间 (排序算法不是线性的)
    - dirty太频繁也会花费很多时间去绘制一个canvas
    - **当一个drawable UI元素变化(sprite修改, transform修改, text修改等等)的时候, 就会触发它所在Canvas的重新绘制! 而Canvas重新绘制会需要对Canvas上的所有Element进行分析, 排序, 重新绘制!**
8. Child order优化
    - 中间layer打破draw call的情况
        + 重新组织hierarchy, 让可以一起draw的element排列在一起
        + 不排列hierarchy, 但是让元素互相不覆盖
9. Splitting Canvases
    - 分到sibling Canvas: 适用于需要独立layer的元素 (例如tutorial)
    - 分到sub canvas(可以继承parent canvas的属性)
10. 通用优化规则
    - 通常一起更新的互相不遮挡的元素应该放在一个canvas上(例如进度条和时间)
11. Raycast优化
    - Raycast实现: 每帧检测一次, 主要检测RectTransform和Ray的关系(所以可以跟Image分开来)
    - Raycasting优化tip: hierarchy越简单, 需要检测raycast的recttransform越少, 就越快
    - 一个transform下的所有子transform(递归)都会被测试是否实现了ICanvasRaycastFilter(costly), 所以如果hierarchy太复杂, 会很慢
    - overrideSorting可以用来减少raycast过深的检测(因为它可以让深的transform显示在上面, 然后先检测, 这样被盖住的就不用检测了)

# Chpt 5 优化UI控制
## UI Text
- Text mesh rebuilds:
    + UI text mesh在每一次UIText相关的显示改变(包括go显隐)时, 都会被rebuild, 而且文字一定会跟图片在不同的batch里, 所以会造成消耗
- 动态字体和字体atlas
	+ 当且仅当2个Text有相同的Size、文字、字体的时候，他们才会使用同一个字体atlas里面的图形（很少见），任何字体差异、字体大小差异都会生成不同的glyph
		- 字体的atlas rebuild分为2个阶段， 第一阶段是尝试在不改变atlas大小的情况下把新的字glyph塞进去，如果成功了就不用第二阶段了
		- 第二阶段是把短边扩大为2倍（例如512\*512扩大为512\*1024）
	+ 当用的文字比较少的时候，推荐用静态字体（例如只用ascii编码的字），但是如果要用unicode字符集，那么必须用动态字体
	+ font atlas会在每一个UI Text组件变化的时候被rebuild，因此如果有特别多的Text组件，就可以先把所有的文字收集起来，构建一个atlas，这样就不会有很多的动态字体atlas生成消耗了
	+ 注意，当font atlas被rebuild的时候，没有出现在active的Text组件里的glyph是不会被加进新的atlas里的！就算手动调用了Font.RequestCharactersInTexture也一样！（解决方案是：订阅Font.textureRebuilt delegate，然后查询Font.characterInfo来保证字符一定存在在atlas里：`public void TextureRebuiltCallback(Font rebuiltFont) { /* ... */ }`

## 特殊glyph渲染
- 例如积分(score), 可以把数字按照digit拆分, 然后一个digit一个sprite, 这样就可以避免文字atlas的重建了

## Fallback fonts
- 应用通常会在一开始Import的时候把字体加载进来, 如果是日语/中文这种字体, 就会占用很大空间

## Best Fit和性能
- "In general, the UI Text component's Best Fit setting should never be used." 草

## TextMeshPro: 额外的文字组件包, 提供更丰富的文字效果(简称TMP)
- TextMeshRebuilds: 推荐使用TextMeshPro而不是TextMeshProUGUI, 因为更高效
- 字体和内存: TMP不提供动态字体特性, 因此全都依赖于fallback fonts
    + 当字体缺失的时候, 会依次搜索字体集合, 如果仍然找不到, 就会使用替代字符
    + 最好把TextMeshPro引用的字体放在settings里, 否则字体会在第一次Text组件第一次activate的时候才会被加载, 可能会造成想不到的内存消耗
    + app启动的时候, 应该以下面的顺序进行字体设置
        * 创建基础TMP字体Assets的AssetBundle
        * 为每一种语言创建需要的fallbackTMP字体Asset的AssetBundle
        * Bootstrap的时候加载基本AssetBundle
        * 基于locale, 加载需要的fallback fonts所在的AB
        * 对于每一个基础AB里的字体, 用locale的AB里的fallback字体进行赋值
        * 继续加载游戏

## ScrollView的优化
- 通过加一个RectMask2D组件给ScrollView, 可以保证不可见的go不会被sort和rendering
- setparent会导致rebuilds, 因为他会触发OnTransformParentChanged回调, 在这里面会调用SetAllDirty(理论上应该是进行sort, 然后判断是否需要setdirty比较好)
- 基于position的scrollview pool(也就是后来我们第二代优化后的circularscrollview), 通过改变相对的position而不是创建/销毁go来进行pool (完全自己实现, 并继承LayoutGroup类(这是我们没做的))

# Chpt Final, 其他优化
## 有些效果可以用RectTransform达到, 它的性能比layout好
## Diabling Canvases: 显隐go会导致vbo的清空和canvas的rebuild/rebatch, 所以可以把ui放在它自己的canvas和sub canvas上, 然后enable/disable该canvas, 这样就可以保留batch的结果
- 但是这样带来一个缺陷, 就是monobehaviour的生命周期需要重新自己实现(由UIRoot来进行callback分发)
- 这一条的原理不是很懂
## 设置EventCameras
- 如果canvas设置为world space或者screen space-camera, 一定要把camera参数设置上, 因为不设置的话unity就会调用GameObject.FindWithTag去找MainCamera, 而这个过程是很慢的
## UI源代码自定义(因为UI源代码是开源的)
- 如果这么搞了,记得重新编译UI的DLL并且覆盖Unity里面的DLL文件(并且还要保证Unity版本适配!)
- 还要保证能够正确地将这个DLL文件分发给打包机, 以及其他的开发者, 慎用