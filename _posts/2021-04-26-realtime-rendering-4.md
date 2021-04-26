---
layout: post
title:  "Realtime Rendering 4th Notes - 实时渲染4th笔记"
date:   2021-04-26
categories: blogs
author: 2bdenny
---

## Chpt 1 Intro
> - 游戏通常需要30、60、72或者更高的fps
> - 电影通常是24帧，但是把每一帧播放2到4次来避免频闪
> - 虚拟现实一般要求90 fps
### 章节介绍（冲突检测部分文档要自己去下，不在书里）
### 符号
- 注意一下向量/点都是column向量
    - 常见记法：$(v_x, v_y, v_z)$，$(v_x\ v_y\ v_z)^T$
- 注意一下矩阵元素的记法
    - ![image.png](http://pfp.ps.netease.com/kmpvt/file/5d43fa882dcadec265321d923TSgtIBa01?sign=A7h5vVGPVum9TUPbdc1V2dcjOYM=&expire=1599638797)
- 注意下是右手系

## Chpt 2 图形渲染管线
### 2.1 体系结构
- 所有的stage都是并行执行的，因此总体的执行速度取决于最慢的一个stage
- 基本的管线：
![image.png](http://pfp.ps.netease.com/kmpvt/file/5d7f4e8668d8640bb466a531ZSsOB9f901?sign=PYuNCDnUNJffw90pcM8kI2LehDs=&expire=1599638797)

### 2.2 应用stage
- **superscalar construct**：指像应用stage一样可以在多个core的CPU上并行执行的构造？（迷惑）
- 一般冲突检测在这个阶段执行

### 2.3 几何处理
![image.png](http://pfp.ps.netease.com/kmpvt/file/5d7f4f4d2dcadede7fee6de6djtZwZ5X01?sign=3sc29G9ygbtVKANSTDX_epmR00M=&expire=1599638797)
- Vertex shading
    - 通过把z坐标保存到zbuffer，可以把模型投影到xy的二维平面上
- 可选的：
    - geometry shader可以生成vertex，所以可以用在粒子系统等等方面，把单个点生成朝向各个view和camera的面，从而允许贴图等等
    - 一般来说三种可选的stage的顺序是：tesellation（镶嵌）、geometry shading、stream output
- 裁剪：把不在视锥里的部分裁剪掉
- 屏幕映射：把立方体按照屏幕（窗口）的尺寸进行映射

### 2.4 栅格化
- 用来找到所有primitives的pixels，包含三角setup和三角遍历两个阶段
- ![image.png](http://pfp.ps.netease.com/kmpvt/file/5d7f553f68d8640bb466a535u6xlMF9m01?sign=IcKCn5xyXns1frN_BVnYbAFbARs=&expire=1599638797)
    - 三角setup：计算三角形的differentials、edge equations
    - 三角遍历：有了前面阶段的东西后，就可以计算每个pixel是不是属于某个三角形了，以及计算fragment了

### 2.5 Pixel processing
- pixel shading（fragment shading）：读取贴图什么的在这里
- merging：读取color buffer，然后混合一下什么的，另外还需要解决可见性
    - 但是透明物体没法解决，按照透明的特性，透明物体一般需要从后往前画
- framebuffer包含了所有的buffer
    - 为了避免正在渲染过程中的样子被看见，现在使用的都是双buffer，互相切换的方式来做的

### 2.6 纵观整个管线
- application stage
- geometry processing：vertex和normal等信息进行变换，变换到view space
- 栅格化：找到所有在primitive里的pixel
- pixel processing：计算primitive里的pixel的颜色
- 另外：这本书假设所有的开发都是在可编程GPU上做的

## Chpt 4 变换
> 线性变换
    满足：
    1. `f(x)+f(y) = f(x+y)`
    2. `kf(x)=f(kx)`
### 基础变换
1. shear matrix: 相当于拉着长方形的一条边把它拉成矩形,但是保持面积不变
2. euler transform: 按照yaw(head)\pitch\roll (偏航\投掷\翻滚)三个方向依次旋转
3. slerp: 两个四元数之间的一个过渡变换

#### 平移
#### 旋转
- 保持距离和手性不变
- 对于任意3·3旋转矩阵**R**,沿着任意轴旋转x弧度,**R**的秩与轴无关,都为1+2cos(x)
- 所有旋转矩阵R的determinant都是1,并且都是正交的
- Ri(x)的逆就是Ri(-x)
- 沿着某个点旋转一个角度的计算方法:先把该点平移到旋转轴,然后旋转,然后平移回去
#### 缩放
- 下面这条可以记一下!
![image.png](http://pfp.ps.netease.com/kmpvt/file/5d3e60e72dcade3f19afdc93ysKiKukr01?sign=3bjiT17NWKIKOb94PFxWDvk320w=&expire=1599638797)
- 所谓反射矩阵,就是对应的某个scale参数取反
- 检查某个变换矩阵是否是反射矩阵: 左上33的元素的determinant为负数就是反射矩阵
#### shear
![image.png](http://pfp.ps.netease.com/kmpvt/file/5d3e61b92dcade3f19afdc942qN3u9pp01?sign=iWWiO3PyBVpv9kwtMcUCpGIHTVY=&expire=1599638797)
- shear矩阵的determinant=1,并且被shear的图形的面积不变
#### 组合变换
- 矩阵相乘不可变换,但是可以结合(常见顺序:TRSp)
#### Rigid-Body Transform
- 只进行旋转和平移,长度\角度\手性不变(?)
![image.png](http://pfp.ps.netease.com/kmpvt/file/5d3e625668d8641b8b33c671wfsIoeY701?sign=Y2qL0P2-2sFqsPYzPNWMsuC7NFc=&expire=1599638797)
- 例如变换相机的朝向
![image.png](http://pfp.ps.netease.com/kmpvt/file/5d3e62c62dcade5dc9ecf5a0TBcwDI4f01?sign=1qWLMRlK7ewTZuAVn-d-yRFX8DA=&expire=1599638797)
#### Normal Transform
- 对于normal transformation来说,真正需要计算伴随矩阵的只有左上的33矩阵
### 特殊变换
#### 欧拉变换
- 变换顺序:yxz
![image.png](http://pfp.ps.netease.com/kmpvt/file/5d3e699868d8641b8b33c67cKCZe7VHu01?sign=tekkYFQ4hZzF7JP4zmSfdY8IpEY=&expire=1599638797)
- 一般我们都是y-up
> gimbal lock(万向节锁)
#### 欧拉变换的参数
![image.png](http://pfp.ps.netease.com/kmpvt/file/5d3e6b0168d8641b8b33c67fpjHTOAfz01?sign=ifNnGZ2l-Ihz6iEp8_lV20lG6_g=&expire=1599638797)
![image.png](http://pfp.ps.netease.com/kmpvt/file/5d3e6b0b68d864dcf70fb4b1zBXDQi7p01?sign=IxoGBWcJhNwMMoa8qds0xbWXlBw=&expire=1599638797)
- 关于如何限制旋转方向: 从输入中提取出欧拉角,只执行其中一个方向的变换
#### 任意射线和角的变换
- 先把坐标系建到以轴为x轴的坐标系,然后进行x轴旋转,然后把坐标系变回来
- rst的st的计算方式:
![image.png](http://pfp.ps.netease.com/kmpvt/file/5d3e6f592dcade5dc9ecf5a6TWZXVyFT01?sign=jVXid9LPRkeVcZVbM_V-ir1X8Xg=&expire=1599638797)
### 四元数
- 懵逼
- 总之可以用来表示任意的三维变换
- 那一堆公式和推导略过吧（Orz）
- 欢迎大佬补充
### 4.4 顶点混合（骨骼动画）【重要!!!】
![image.png](http://pfp.ps.netease.com/kmpvt/file/5d3e98962dcade3f19afdcc9Pg9533x301?sign=DRCxFVQGi3ObmZa-CQ1YozZDjis=&expire=1599638797)
- 这个公式里，$w_i$代表该vertex收到第i个bone（joint）影响的weight（哎哟，吹爆km，居然支持公式）
- p初始状态的点的位置
- $M_i^{-1}p$代表把p从随便什么空间的坐标（一般是模型空间或者世界空间），变到它所在bone space的坐标
- $B_i(t)$通常是一系列bone的坐标变换相乘（代表依次从初始bone space变换到根的bone space，再变换到world space），然后再乘一个变换到世界空间的坐标，代表从0到t时刻的变化矩阵
- 如果p和u是在同一个坐标系里，那么$B_i(t)M_i^{-1}$就是骨骼i的变换矩阵

### 4.5 变形
- 基本变形公式
原理：![image.png](http://pfp.ps.netease.com/kmpvt/file/5d40ee552dcaded1faff4aafqCGOPUiN01?sign=kDhp10nBDhPl6q1Z-pxMpXBz5qw=&expire=1599638797)
通用：![image.png](http://pfp.ps.netease.com/kmpvt/file/5d40ee942dcaded1faff4ab03fpyxY4h01?sign=6ybh-IaDJJ7kcg7oEZhnmnQVUPY=&expire=1599638797)
- trick：weight也可以为负数：例如让一个笑脸变成哭脸

### 缓存
- 主要是利用了某些heuristic, 例如n-1帧到n帧的动画, 很可能是跟n帧到n+1帧的动画是一样的

### 4.7 投影
- DirectX使用的是左手系!
- **正交投影: 平行线在投影之后, 仍然是平行的!!!**
- near/far plane的概念不知道在哪, 回去补充一下...
- $(l, r, b, t, n, f)$, 其中n > f (近处物体看起来总是比远处的大)
- opengl的正交变换矩阵
![image.png](http://pfp.ps.netease.com/kmpvt/file/5d40f13a2dcaded1faff4ab373ZO1KVi01?sign=k9oiL1R64zs3WrSoKOYW5n1Kjoc=&expire=1599638797)
- DirectX的正交变换矩阵
![image.png](http://pfp.ps.netease.com/kmpvt/file/5d40f1592dcaded1faff4ab4lMwg8wxC01?sign=iCx6pICYxrMQhtSowwcX8m_b7wA=&expire=1599638797)
- 关于perspective matrix
    - 不对称视锥通常被用于stereo viewing和虚拟现实
- 深度公式(不用记, 都是封装好的函数调用一下就行了)
![image.png](http://pfp.ps.netease.com/kmpvt/file/5d40f67168d8641a762ab0b9tDaXp8zD01?sign=-xG-PhqEkboAwfnae2cq4WDwdD4=&expire=1599638797)

## Chpt 9 PBS（PBR，基于物理的渲染）
### 9.1 光的物理特性
- 光的波粒二象性/光传播过程的能量守恒等等（。。。）
- 折射率、入射角、反射角、折射角
![image.png](http://pfp.ps.netease.com/kmpvt/file/5d7eefae2dcadefd5eb24bcb9c9K0QVZ01?sign=zoGqfiai-CANSPtJ239oUeKXR10=&expire=1599638797)
![image.png](http://pfp.ps.netease.com/kmpvt/file/5d7eefb82dcade6dc4117c8cQESx5c9501?sign=iRi_ZgdiWEWXEZuGkFj7OL5bx90=&expire=1599638797)
- 金属材料的自由电子可以吸收折射光，然后将其重定向为反射光，因此金属材料有很高的光吸收率，同时又有很高的光反射率
- 空气被加热会导致空气的折射率发生变化，进而使得光线弧形传播
- 很少漫反射、很少吸收光线的材料看上去就是透明的
- **在渲染过程中，specular项建模了平面的反射，diffuse项建模了local subsurface scattering（光线进入和退出是在同一个subsurface里的）**，如果光线进入和退出在不同的subsurface里，那么就需要**global subsurface scattering**模型来建模
    - 简单来说，就是需要精细显示的时候用global模型，需要粗糙显示的时候用local模型

### 9.2 摄像机
- 摄像机由挡板、针孔、透镜组成，我们所说的camera position其实是针孔的位置
![image.png](http://pfp.ps.netease.com/kmpvt/file/5d7ef2d82dcade6dc4117c9bWF1wOrXz01?sign=Rgwo-7k8fAVhbuhyoFrW-q7kbQ0=&expire=1599638797)

### 9.3 BRDF
![image.png](http://pfp.ps.netease.com/kmpvt/file/5d7ef35f2dcade6dc4117c9dcLOL8Syh01?sign=-0Ov7ZFQYHtx96MqWixmLOyun3E=&expire=1599638797)
- 这里取负是因为v总是指向摄像机方向
![image.png](http://pfp.ps.netease.com/kmpvt/file/5d7ef49368d8648104b71d210h8Y5YRK01?sign=T_PgIKVVXeSe7P3a-sxLYzKJrM8=&expire=1599638797)
- 本章假设真空，因此进入摄像机的光的数量等于离摄像机最近的平面出射到摄像机的光
- 可以使用反射方程计算BRDF：
![image.png](http://pfp.ps.netease.com/kmpvt/file/5d7ef63468d8648104b71d2aWFqxp6gO01?sign=iypIMKZyiRoiriftgDz3i3CnN54=&expire=1599638797)
- 简化一下就是：
![image.png](http://pfp.ps.netease.com/kmpvt/file/5d7ef6942dcadedc1a6440bfR7ZtNyN801?sign=EONBymzrgdSmoMDqg6-8wA8UthE=&expire=1599638797)
- 需要注意，BRDF仅在相机和光线都在平面上方的时候才有用
    - 但是有时候经过对法线的插值，会使得这一假设不成立，对此就可以采用clamp操作（当计算出的值为负时取0）
    - 根据光传播的性质，交换光和摄像机的方向，结果不变，因此有![image.png](http://pfp.ps.netease.com/kmpvt/file/5d7ef8122dcadefd5eb24bd7dM3iEykg01?sign=Jw2fW0EDGeQ_ibP4JlDwBnw3OZE=&expire=1599638797)，但是实际上我们用的BRDF很可能是不满足这条性质的
- ![image.png](http://pfp.ps.netease.com/kmpvt/file/5d7ef8d568d8640bb466a4b70j7LBt7v01?sign=XMEAv2XmWorIsScMNyeDoDM0KMw=&expire=1599638797)
- 总的来说，BRDF这个函数描述了不同入射方向光的对于出射方向光的贡献

### 9.4 照明
- 如果光源确定，那么BRDF就可以得到简化，进而简化整个反射方程：
![image.png](http://pfp.ps.netease.com/kmpvt/file/5d7efaa22dcade6dc4117ca6thyJupfD01?sign=j7Xg423ADYkhtMnFs2Rzn4Edjtk=&expire=1599638797)

### 9.5 菲涅尔反射系数
- 菲涅尔系数描述了反射光的数量在入射光数量中的比例（还有一部分是折射光）
- 计算公式：![image.png](http://pfp.ps.netease.com/kmpvt/file/5d7efb472dcade6dc4117ca7QZObwrQ201?sign=ME4Qso1B3a9HILzeC5YF8Ulgqx0=&expire=1599638797)
    - 其中（假设n1=1,n2=n）：![image.png](http://pfp.ps.netease.com/kmpvt/file/5d7efb6868d8640bb466a4b82Bpx68XP01?sign=pf81B7Yhs9jUZGHSR5BZvg3U9uM=&expire=1599638797)
    - 当n1不等于1时：![image.png](http://pfp.ps.netease.com/kmpvt/file/5d7efc0668d8640bb466a4baH8TbTyhG01?sign=co74nJ7quSCJM_DUgpazVNmu4GA=&expire=1599638797)
- 也有人用另外一个公式来计算：![image.png](http://pfp.ps.netease.com/kmpvt/file/5d7efb962dcadefd5eb24bd9cb5ApPSi01?sign=0WTqVPo0zHz345tqAXKcMlg8w9w=&expire=1599638797)
- 内部反射仅在绝缘体里才会发生，因为导体和半导体会吸收光线

### 9.6 微平面
- 回射（retroreflection）：参见交通标志这种反射特别强的材料

### 9.7 微平面理论
- 每个微平面根据它自己的BRDF对入射光进行反射
- 可见微平面的投影面积之和等于宏观平面的投影面积，遮罩函数$G_{1}(m,v)$描述了对于法线m，摄像机v来说可见的微平面
![image.png](http://pfp.ps.netease.com/kmpvt/file/5d7f003c68d8640bb466a4c4RvCJi8dO01?sign=WV2tjfamLihcGM-oKR9ouOVG6GI=&expire=1599638797)
- 前面的![image.png](http://pfp.ps.netease.com/kmpvt/file/5d7f004b2dcade6dc4117cb1rNbVZmwh01?sign=v6ZHv9_iblQrD1XSIewYD6fWuyc=&expire=1599638797)代表可见法线的分布
- 可用的遮罩函数：
    - ![image.png](http://pfp.ps.netease.com/kmpvt/file/5d7f007d68d8640bb466a4c6bVMWWhpx01?sign=lIGdZIAdupRMihHygu9IgLQosPQ=&expire=1599638797)
- 然后代入得到宏观的BRDF：
    - ![image.png](http://pfp.ps.netease.com/kmpvt/file/5d7f00c62dcade6dc4117cb2KvJSxNQ501?sign=5E7hC0eCEvcZMcAgAyWqiXxzNEM=&expire=1599638797)
- 由于光线的遮挡是双向的（有的平面照不到，有的光线出不去），所以他们提出了：
    - ![image.png](http://pfp.ps.netease.com/kmpvt/file/5d7f01212dcade6dc4117cb6PsZCdCNY01?sign=bRXdGN_2ahFNXJ8pSunLLoNvQKI=&expire=1599638797)
    - 但是这个函数会导致过暗，这是因为，双向遮罩在l和v趋向于相同的时候，遮罩应该等于阴影和遮罩之间的最小值，而不是两者相乘，因此引入一个参数：
        - ![image.png](http://pfp.ps.netease.com/kmpvt/file/5d7f018568d8640bb466a4cdbNtIcXyy01?sign=ldhwprYGj-QB-ZwMYqnVNR5cEtU=&expire=1599638797)
        - ![image.png](http://pfp.ps.netease.com/kmpvt/file/5d7f01962dcade6dc4117cb9TiYcjgwa01?sign=veosdoEft0kCv3t8tcp3C2Z7bPo=&expire=1599638797)
        - ![image.png](http://pfp.ps.netease.com/kmpvt/file/5d7f019f2dcade6dc4117cbawWBNLsYI01?sign=4nzYnSvCPBdOdL2Q6f9R2Gv7AvM=&expire=1599638797)
- 最后推荐的遮罩函数是：
    - ![image.png](http://pfp.ps.netease.com/kmpvt/file/5d7f02062dcade6dc4117cbfPGiYQGb301?sign=73DRJmoAtZ0mzSHc_mlE7qYFlCM=&expire=1599638797)

### 9.8 平面反射推荐的BRDF模型
- 反射模型（代码里用到的）
    - ![image.png](http://pfp.ps.netease.com/kmpvt/file/5d7f02b92dcade6dc4117cc1dxH5NDyj01?sign=eqc5dnjgMTMn96sseYxb7CmuD1c=&expire=1599638797)
- 其中的法线分布函数（代码里用到的一般是GGX）
    - ![image.png](http://pfp.ps.netease.com/kmpvt/file/5d7f02f82dcade6dc4117cc2FSExGBLz01?sign=fg9UBZu43g9ioswaw_PHASOlUEc=&expire=1599638797)
    - 其中：![image.png](http://pfp.ps.netease.com/kmpvt/file/5d7f032a2dcade6dc4117cc4Yhobij8a01?sign=IIz_n-gsGEEcFrijWBeTt7OpYiA=&expire=1599638797)，r是粗糙度
- 此外还有GTR法线分布函数，可以用于控制光晕的形状
- 对于各向异性的法线分布函数，一般配合tangent map来使用，处理例如头发、拉丝金属等材料
- 微平面BRDF模型无法处理微平面间光线多次反射的情况，这种情况可以用![image.png](http://pfp.ps.netease.com/kmpvt/file/5d7f04042dcade6dc4117cc8sMY6mHla01?sign=jisetLib1k6FmsM8iC1JX7xsIjg=&expire=1599638797)这个参数来描述

### 9.9 子平面漫反射的的BRDF模型
- surface albedo用$\rho_{ss}$表示，一般就是diffuse color，对应的菲涅尔反射系数用$F_0$表示，一般就是specular color
- 三种微平面粒度：
    - ![image.png](http://pfp.ps.netease.com/kmpvt/file/5d7f068868d8640bb466a4d9oBo0eZ2R01?sign=CHsk3dDCXkr827bOu4I6Qih_dsE=&expire=1599638797)
    - 出射光线在微平面内
    - 出射光线在微平面外
    - 出射光线在小粒度的微平面外，在大粒度的微平面内

### 9.10 衣服（织物）的BRDF模型
- 神秘海域2里用到的：![image.png](http://pfp.ps.netease.com/kmpvt/file/5d7f07332dcade6dc4117cc9ZlfoLiqS01?sign=O-tMY8M5g3YXj1xYr52LBGv35Vw=&expire=1599638797)
- 教团1886里出现了天鹅绒材料
- 此外还有微柱体织物模型

### 9.11 把光看作波模型的BRDF模型
- 薄膜干涉
    - 薄膜厚度大于1微米就不会有这种现象了
- 衍射模型
    - CD和DVD上面可以看到的彩色

### 9.12 多层材料
- 例如涂漆的效果
- 清澈的小溪
- 铺在土地上的床单（各种褶皱）

### 9.13 Blending and Filtering 材料
- 建筑物、车等模型上的动态的破坏
- 游戏中人物的换装、装备
![image.png](http://pfp.ps.netease.com/kmpvt/file/5d7f09542dcade6dc4117ccaTBrPSibe01?sign=q19ZT0KBjtdMqIcYBqt-G2oGM-A=&expire=1599638797)
- 闪烁的雪等等