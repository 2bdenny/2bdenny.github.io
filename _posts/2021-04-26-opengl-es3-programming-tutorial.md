---
layout: post
title:  "OpenGL ES3.0 Programming Tutorial Notes - OpenGL ES3.0编程指南笔记"
date:   2021-04-26
categories: blogs
author: 2bdenny
toc: true
---

# Chpt 8
## Vertex Skinning
- 顶点位置、模型法线数据的计算公式，需要注意法线是需要先把变化矩阵取逆然后转置
![image.png](http://pfp.ps.netease.com/kmpvt/file/5d4272b62dcade1d93b8ca53H6IcdKAy01?sign=HyTeboAoo6QHpY591w3sqG_U81E=&expire=1599638740)

-------

# Chpt 9 Texturing (贴图/纹理)
## Texture Basics
 > opengl es3.0支持的纹理有：2d纹理、2d纹理数组、3d纹理、cubemap
 ### 2D纹理
 - texel是texture pixel的简称。。
 - 下面这些纹理类型没见过，截图记录一下
 ![image.png](http://pfp.ps.netease.com/kmpvt/file/5d5691642dcade9bac4985a2FsCFUkx801?sign=EAFkjv9r4IsXLL1fI5TPcB_JR5A=&expire=1599638740)
 - uv坐标（从左下角到右上角的坐标）
 ### cubemap
 - 一般用来做天空盒这种环境贴图
 - 本身的图是通过一个摄像机放在中间，然后朝6个面各照一张来完成的
 - 以$s, t, r$建从原点出发的向量，与cube相交的地方的texel就是实际贴图所使用的texel
 - 向量位置不是通过点来存的，而是通过法向量和观察向量来计算获取的（与其他贴图不一样，详见chpt14）
 ### 3D texture
 - 3D texture本身是一堆的2D texture 的slice堆叠放置形成的
 - 它的mipmap包含了其中一半数量的slice
 ### 2D texture
 - 2D texture array一般用来保存一张2D的图片动画
 - 与3D texture不同的是，3D texture在采样的时候会综合多个slice来获取某个点的texel，而2D texture array则是从其中的某一张获取texel
 ### 纹理对象和纹理加载
 - `glGenTextures`可以一次创建一个texture数组
 - 当纹理不被需要的时候，可以直接释放（例如程序结束或者进入下一个关卡）
 - 当需要比较好的性能的时候，可以使用不可变纹理
 - 需要注意，pack和unpack的选项设置是全局状态，并不是单个texture对象的状态
 ### 纹理filtering和mipmaping
 - mipmap解决两个问题：
     - 大纹理小对象的情况下，小对象使用到的点太少导致的小对象显示问题（失真问题）
     - 大纹理造成的性能问题
 - mipmap chain生成的时候，每次生成的图像的dimension都是上一次的一半大小
 - 一般都是取4个mipmap level取平均来获取texel的
 - 纹理的filtering
     - minification：当对象的size比纹理要小的时候发生
     - magXXX：相反
     - seamless cubemap filtering：opengl es 3.0默认是seamless的，也就是无缝的
 ### 自动mipmap生成
 - 有函数可以直接调，不用自己写了
 - 在使用framebuffer来渲染一个纹理的时候，自动mipmap生成就很重要（因为自动mipmap生成是在GPU里做的，详见chpt12）
 ### texture坐标wrap模式
 ![image.png](http://pfp.ps.netease.com/kmpvt/file/5d5695ea2dcade9bac4985a3CzeT41IN01?sign=s6pSBLARWpg5t5mlNWIa8tt8ZBM=&expire=1599638740)
 ### Texture swizzles
 - 简单来说这个机制控制了texture里的各个rgba域在shader里可以被分开使用（例如法线贴图用rgb来存切线空间的法线！）
 ### Texture LOD
 - 使用texture base并最大化mipmap的level会导致新的mipmap被加载的时候出现瞬间变化（popping artifact），而混合LOD可以把这个转换变得更平滑
 
### Depth Texture Compare（Percentage Closest Filtering）
- 简单来说就是纹理的混合会导致深度测试的错误（大概是这个意思）
- PCF就是解决这一问题的机制

### Texture Format（纹理的存储格式）
- normalized texture format
    - 简单来说就是所有的值都在0到1.0之间
- 浮点 format
    - OpenGL ES 3.0不支持直接将浮点格式用作render的颜色，仅可以使用16bit的半float类型来用作filter
    - 32bit的颜色数据还有R11G11B10这种存储方式（指数部分都是5bit），显然比8、8、8、8的RGBA的格式有更好的RGB精度（但是没有A通道啦）。
        - 这种形式可以保存的最大颜色数值是$6.5\times 10^5$，最小是$6.1\times 10^{-5}$，有人能告诉我是怎么算的吗
        - 11bit的有2.5的10进制数字精度（啥意思）10bit有2.32（？？？）
- integer format
    - 使用integer texture作为color attachment的时候，alpha blend state会被忽略
- shared exponent texture
    - 一般 high dynamic range（HDR）的图片会用到这个格式
- sRGB texture
    - sRGB是一个非线性颜色空间的纹理
    - 这种颜色空间的出现是因为人类在这种颜色空间中可以更轻松地区分颜色
    - 但是这种颜色的一大缺点就是，光照的计算也是非线性的
    - **所以不能直接把srgb的颜色取出来，然后进行光照计算，正确的方法是将srgb的颜色转换为线性rgb颜色，然后进行光照计算。现在的很多app实现都没有考虑到这一点，导致颜色出现偏差。**
        - sRGB转线性空间通过`pow(value, 2.2)`来完成，反过来转通过`pow(value, 1/2.2)`来完成
- depth texture

### 在shader里使用纹理
![image.png](http://pfp.ps.netease.com/kmpvt/file/5d56992268d8641dcc63d101ti8rHiDF01?sign=7RVknrlLIW7UAixutfauXc7RiMA=&expire=1599638740)
这个precision是干啥的？
- sampler uniform就会会从固定的texture通道里读取texture信息
- shader里的texture函数返回的是vec4，其中的数据依赖于texture里的base format
- 注意下3D和2D array纹理的加载函数，其中r坐标是一个float point数值，对于3D纹理来说，真正获取的texel根据filter mod的设置，可能会span2个不同的slice

## 压缩纹理
> 好处1是省内存，好处2是省内存带宽
- 压缩纹理通用格式是爱立信的ETC2，是个被opengl组织买断授权的格式，可以放心用，但是这个格式不支持3D纹理
- 这个格式的纹理size计算公式为：
    - ![image.png](http://pfp.ps.netease.com/kmpvt/file/5d569a9068d8641dcc63d102tYKvjv2g01?sign=YaDLyBiEF1PhCF7E8xl51TLwVHI=&expire=1599638740)
- 相关的生成这个纹理的工具有：
    - ![image.png](http://pfp.ps.netease.com/kmpvt/file/5d569abb68d864719eef61de01ThRLX301?sign=u3ygDXHbB2MpZsb8QaNgzI_gPD0=&expire=1599638740)
- 当尝试在opengl es 3.0里使用一个不支持的压缩格式的时候，一般会得到一个GL_INVALID_ENUM错误
- 查询支持哪些压缩格式的方法
    - ![image.png](http://pfp.ps.netease.com/kmpvt/file/5d569b042dcade9bac4985a4VzuT5jCM01?sign=bM51nmHe0RruJ8MQZlbdYVGaR6c=&expire=1599638740)
### texture subimage specification
- 部分纹理，省空间
- 但是对于3D纹理来说，由于通用压缩纹理格式不支持，所以要查查自己的设备是否支持 
### 从Color Buffer拷贝Texture Data
- framebuffer提供了一个更快的render to texture的方式，比拷贝image数据快
- 注意从color buffer 拷贝image data的时候，必定是从back buffer拷贝的
- 只拷贝一部分也可以的（`glCopyTexSubImage2D`）
- 在拷贝的时候，color buffer的颜色component必须大于等于image的数量（例如color buffer没有A通道，那么image也不可以有A通道）
### Sampler Object
- 之前我们总是给texture设置属性，但是遇到有很多texture，属性都相同的时候，我们可以直接对sampler设置属性，这就用到了sampler object
### 不变纹理
- 如果一个texture不会发生改变，可以把它创建为不变纹理对象
- （也就是不能往这个texture里输数据啦（例如glTexImage\*这种的穿数据的函数））
- 这种纹理里填数据可以用的函数
![image.png](http://pfp.ps.netease.com/kmpvt/file/5d55312168d864e3927c8f01JTkvFQyI01?sign=gEIdq9vWjblNq1urTKv-epp7weU=&expire=1599638740)
### Pixel Unpack Buffer Objects
- buffer对象的作用是在GPU显存里存数据（而不是在CPU的内存里存数据）
- 本节的这个对象的作用就是直接把texture的数据存在显存里
![image.png](http://pfp.ps.netease.com/kmpvt/file/5d5531ca68d864719eef617a2YJB65jQ01?sign=qBNAU-643UttIcn7ZnLnieO8fmk=&expire=1599638740)
- 简单来说就是当CPU-GPU之间的纹理数据传输造成瓶颈的时候推荐使用这个对象存纹理数据

-------

# Chpt 12 FrameBuffer Objects
## 为什么使用FBO
> opengl es 3.0默认draw的是视窗系统提供的framebuffer（0号framebuffer），它也可以draw到一个不在屏幕上的平面，例如pbuffer等等
> 利用这个特性，它提供了render to texture的方法来支持shadow mapping、dynamic reflections、environment mapping等等渲染效果

**render to texture**的两种技术
- 渲染到framebuffer，然后copy到texture
- 使用一个attach到texture的pbuffer，但这种方法在某些需要让每一个pbuffer和window surface位于不同context的情况下会很低效

**由于这两种技术都不够理想，opengl es提供了framebuffer objects和renderbuffer objects来达成render to texture**

**FBO支持的操作**
1. 创建fbo
2. 在单个context中创建多个fbo
3. 创建不出现在屏幕上的color、depth等等的renderbuffers和textures，然后把这些objects attach到fbo
4. 在多个fbo之间共享上面的东西
5. 直接attach texture到fbo，不用copy
6. fbo之间的copy和fbo内容的invalidating

## fbo和rbo
- fbo可以用来存color、depth等等，可以作为fbo的attachment，类似pbuffer，但不能被直接当作texture使用
- fbo是color、depth、stencil texture或者render targets的集合，但是每样只能保留一个
![image.png](http://pfp.ps.netease.com/kmpvt/file/5d5f49dd68d864acb8cf5be1Rm44LgBs01?sign=TJD3LwoqWdzS_9Uxfh5w11uuvSM=&expire=1599638740)
### 用rbo而不是texture作为fbo的attachment的好处
- 支持multisampling
- 当image不用作纹理时，rbo效率更高
### fbo和egl surface
- fbo的pixel ownership testing总是成功的，而后者可能不成功（例如窗口遮盖）
- 后者支持双buffer surface，但fbo只支持但buffer attachment
- stencil、depth buffer可以在fbo内共享，但是后者不行
### 创建fbo和rbo
![image.png](http://pfp.ps.netease.com/kmpvt/file/5d5f4b1e2dcadedaeec9c935MelHt9A101?sign=JUvQLXwoX6FySNONs9Q7-59mxCM=&expire=1599638740)
![image.png](http://pfp.ps.netease.com/kmpvt/file/5d5f4b2668d864acb8cf5be37WWTovYG01?sign=SEVdFHxvXfJGg-EUqufocZO3UaA=&expire=1599638740)
![image.png](http://pfp.ps.netease.com/kmpvt/file/5d5f4b3d68d864719eef61fdEyjd7RVt01?sign=xL9qQO4kRhKKJHN-lPLsI-uX5m0=&expire=1599638740)
## 使用rbo
- bind
- 直接bind也能生成一个rbo，但是推荐先gen再bind
- bind一个已有的rbo将不会改变它的状态
- `glRenderBufferStorage`类似`glTexImage2D`，但不会应用image data
![image.png](http://pfp.ps.netease.com/kmpvt/file/5d5f4bf268d864acb8cf5be4ypgxUqKk01?sign=nqkEaeWvzstfw8kxOdhX3h8R7og=&expire=1599638740)
### multisample renderbuffers
- 使用这个可以在render to off-screen rbo的时候也应用anti-aliasing的技术

## 使用fbo
- 直接bind也能生成一个fbo，但是推荐先gen再bind
- fbo的状态
    - color attachment point
    - depth
    - stencil
    - fbo completeness 状态
### attaching rbo到fbo的attachment point
`glFramebufferRenderbuffer`
### attaching texture到fbo的attachment point
`glFramebufferTexture2D`
### 3D texture
`glFramebufferTextureLayer`

> 注意：render to texture和应用该texture进行draw不能同时进行，行为不确定

### fbo completeness检查
- 至少一个valid attachment
- valid attachment必须有相同的w和h
- depth和stencil必须是同一个
- 所有rbo的GL_RENDERBUFFER_SAMPLES的值都必须相同

`glCheckFramebufferStatus`
- 当被检查的对象不是fbo的时候返回0
- 当检查成功的时候返回GL_FRAMEBUFFER_COMPLETE

## Framebuffer Blits
- fbo支持的高效copy一个rectangle形式的pixel数据的方式
## FBO invalidation
- 这一机制用于通知driver哪些fbo无效，然后GPU就可以优化管线
> TBR GPU (Tile-based rendering)
> - 常用于移动设备，通过最小化CPU和主存的数据传递来省电和节省内存带宽
> - 感觉可以理解成，render过程可能会对同一个pixel写入多次，如果不使用这个on-chip memory，那么这些写入都会写入到主存，占用主存带宽，而先写入on-chip memory，写完后再copy到主存，每个pixel就只有一个数据，所以节省了主存带宽

## FBO和RBO的删除
- 对应的delete语句
### 删除fbo attachment的rbo
- 删除一个当前bind的fbo的attached rbo，会自动把attachment point set 0
- 删除一个当前未bind的，**不会自动 set 0！！！需要应用手动set 0**
### 读取pixels和rbo数据
- 使用rbo的时候，应该把pixel data设置为NULL
- 迷惑：
![image.png](http://pfp.ps.netease.com/kmpvt/file/5d5f4fe568d864a6e1830a21vpQOqm4A01?sign=9RacL36CqwKiYBAzABIhUSiQGEE=&expire=1599638740)

## 性能相关tips
- 避免0号fbo和其他fbo之间的频繁切换
- 不要每帧create和destroy fbo
- 尽量不要修改attach到fbo，用作rendering targets的texture
- 当texture不会被render的时候，把pixel数据设置为NULL，并在draw之前invaliding它
- 共享depth和stencil rbo

---------