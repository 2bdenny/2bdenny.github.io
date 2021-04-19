# Solution Database
> 是解决方案仓库哦

## [UE4的pak资源导出方法](https://www.bilibili.com/s/video/BV1qE411A79D)
- 希望有效

## [多台电脑使用同一套键鼠](http://home.netease.com/#/forum/post/14666)
- 首先，Microsoft早就发现了这个问题，并且对应开发出了[Mouse Without BOrders](https://www.microsoft.com/en-us/download/details.aspx?id=35460) 软件（巨硬！）
- 或者synergy

## vs2017的fbxsdk的cmakelist配置说明
- 每次重新加载项目都需要重新配置真的太麻烦了，所以折腾了一下cmakelist的配置
- 在`include_directories`下添加条目`D:/Program\ Files/Autodesk/FBX/FBX\ SDK/2019.5/include/`（自己对一下fbx目录不要直接复制）
- 增加条目`link_directories(D:/Program\ Files/Autodesk/FBX/FBX\ SDK/2019.5/lib/vs2017/x64/Debug/)`（同样自己对一下fbx目录不要直接复制）[参考链接](https://stackoverflow.com/a/28606916/4291968)
- 注意**空格**
- 把下面这个配置改成这样
```
set(ALL_LIBS
	${OPENGL_LIBRARY}
	glfw
	GLEW_1130
	libfbxsdk-md
	libxml2-md
	zlib-md
)
```
- 以上3步配置完后可以免去手动添加编译和链接选项的操作

## [glsl的语法高亮](https://marketplace.visualstudio.com/items?itemName=DanielScherzer.GLSL)

## [MSB6006 cmd.exe exit 1](https://stackoverflow.com/questions/13118947/error-msb6006-cmd-exe-exited-with-code-1)
- 一开始找不到错误代码行很奇怪
- 其实这是个cmake错误，比如cmake里的某个文件丢了等等
- 通过output的log就可以找到错误原因

## 骨骼动画动作幅度过大
- 原因1：在获取点的坐标的时候，把最后一个w变量设置为了0，正确的应该是1
- 原因2：矩阵乘法问题，先把矩阵相乘后相加了，正确的乘法顺序应该是：（骨骼在t时刻的变形矩阵 * 骨骼空间映射矩阵 * 初始位置）* 权重，然后向量相加

## [骨骼动画的模型空间和世界空间转换问题](https://stackoverflow.com/questions/13566608/loading-skinning-information-from-fbx)
- 简单来说就是直接获取mesh里的顶点位置，获取到的不是世界空间的位置，需要进行上面链接里的转换才能获取到真正世界空间的位置

## 同一段代码，画一个模型的时候很正常，画另一个的时候出现很多三角形空洞，还有奇奇怪怪的线
- 要检查mesh是否是triangle mesh
```
	if (!mesh_node->GetMesh()->IsTriangleMesh()) {
		printf("Not triangle mesh, convert it\n");
		FbxGeometryConverter triangulator(mesh_node->GetFbxManager());
		triangulator.Triangulate(mesh_node->GetMesh(), true);
	}
	this->mesh = mesh_node->GetMesh();
```

## 在传uniform数组的时候遇到编译错误
> 具体表现是，只要不使用某些uniform变量，就不会出现编译错误，一旦使用就编译错，然后一般报offset XXXX的位置出现编译错
- [关于uniform数组位置的计算方法](https://computergraphics.stackexchange.com/a/5326)
- 检查是否uniform变量size过大：[uniform limitation](https://www.khronos.org/opengl/wiki/Uniform_%28GLSL%29#Implementation_limits)

## 模型失去了大部分的颜色和所有的贴图
- 检查in-out变量是否对应、以及in-out关键字是否正确，当把所有的in都写成out的时候会遇到这个问题

## 在计算切线的时候，部分切线值是NAN导致模型出现黑色空洞
- vector不能取地址！！！
- 原因是通过指针指向vector里的元素，这样一旦vector进行扩容，指针就会指向不可达的位置或者错误的位置！！！

## 个人SVN配置和使用
1. 去软件中心下载安装svn
2. 最好创建一个跟远程文件夹一样的本地空文件夹，然后右键svn checkout，输入url
3. svn的账号和密码是邮箱前缀/邮箱密码
4. 然后svn add
5. svn commit

## OpenGL的各种库的依赖关系调整
最重要的提示来自这里：
![image.png](http://pfp.ps.netease.com/kmpvt/file/5d53cc792dcade7d7e0454f0Bil0dPB901?sign=01hHtxsy-l-YWAC-WTtIlo-bk8c=&expire=1599638535)

> **遇到的坑：glad几乎与所有的使用到opengl的库（例如各种gui库）的默认编译选项不兼容，会遇到gl和glew的循环依赖问题导致编译不过**
> 
> **推荐的解决方案**
> **1. 不使用glad**
> 2. 使用glfw+glew的组合，其中glew自己去[官网](http://glew.sourceforge.net/install.html)下载然后按照官网教程进行配置
    > 这个配置很简单，就是把默认的dll文件扔到system32文件夹下，然后把include目录和lib目录加到项目属性中，然后把glew32.lib和glew32s.lib加到链接属性中

## CEGUI的编译和样例运行方法
- 编译教程参考[这个](https://blog.csdn.net/beyond_ray/article/details/22477095)
- 不需要配置boost和ogre这些东西，直接编译，然后用opengl3运行也是可以的
- 遇到报dll找不到的错误肯定是文件夹合并的时候出问题了

## 将CEGUI与OpenGL代码进行合并的方法
- 在上面的代码运行成功后，直接把build/bin目录里所有文件拷贝到OpenGL工程的工程目录下
- 也不要花时间去配dll路径啊环境变量啊这些东西了
- 搞了一整天，吐血

## 遇到一个特别奇怪的情况，CEGUI的窗口和按钮的图形被其他模型的图形覆盖了！
- 最终发现问题出在texture上。
- 如果shader包含过多的texture变量，并且其中部分**没有**active然后bind使用，那么这些多设置的变量继承之前active的值
- 后续GUI如果仅仅通过active（不配套bind）来使用它自己的texture，那么就会导致active的texture带有之前继承的值，而不是gui自己的texture
- 简单来说就是locate和active和bind的texture一定要配套！

## 在添加了泛光效果之后CEGUI消失的问题
- 原因是泛光效果的最后一步画的只有一张深度为0的texture，而CEGUI也是在深度为0的平面上画的，两者互相叠加导致先画的那张被后画的遮挡（CEGUI被quad遮挡）
- 所以解决方法就是让CEGUI在最后一步画，并且在画之前清空深度buffer
> 另外中间不知道为什么又出现了贴图混乱的问题，然后又不知道调了什么就调好了，脑阔疼

## 天空盒的问题（天空盒不显示、天空盒出现漏洞、模型走到天空盒外面、天空盒是个小立方体）
1. 检查天空盒数据的读取问题（顶点数据、贴图数据）
2. 检查天空盒的渲染函数调用顺序是否正确
3. 检查深度测试相关设置是否正确
4. 检查模型大小是否过大（理论上没有影响）
5. 检查是否设置了gl_Position那边的z=w
6. 我特么也很抓狂啊，不知道改了什么天空盒就正常显示了

## 互相遮挡时阴影出现空洞
1. 再开始画深度map之前设置为`glCullFace(GL_FRONT);`画完深度map之后设置回来`glCullFace(GL_BACK);`

## 骨骼动画的过渡动画插值时的骨骼缩放问题
- [远古网易大佬关于“在世界坐标系进行骨骼动画插值导致骨骼缩放问题的解决方案”的博客](https://blog.codingnow.com/2009/11/skeletal_animation.html#comment-43078)
- 对于fbxsdk来说，就是需要对evaluationlocaltransform的矩阵进行插值后乘起来获取模型空间的变换矩阵，而不是通过evaluationglobaltransform获取矩阵后插值来获取模型空间的变换矩阵

## 骨骼动画中的法线贴图问题
- 从法线贴图中读到的法线是在切线空间中的，所以需要转到世界坐标系中进行运算（或者把所有东西都转到切线空间中运算）
- 首先包含骨骼动画的TBN矩阵中的TBN三个值，应该是fbx文件中读到的模型空间的值先乘骨骼矩阵，然后乘model矩阵后的值
- 世界空间的法线=TBN * 读到的法线贴图里的切线空间的法线
- [TB和N如何随动画变化](https://community.khronos.org/t/bones-and-normals-binormals-tangent/40598/2?u=2bdenny)

## 贴图出现网格样条纹或者贴图出现奇怪纹路完全不对的情况
- 美术资源里的贴图的通道数目是**完全随机的**
- 如果使用stbi image来加载图片，推荐使用一劳永逸的加载方法：
```
GLuint Texture;
int width, height, nrChannels;
stbi_set_flip_vertically_on_load(flip);
unsigned char *texture_data = stbi_load(texture_file_name.c_str(), &width, &height, &nrChannels, 4); // 这里用4，强制加载4通道
glGenTextures(1, &Texture);
glBindTexture(GL_TEXTURE_2D, Texture);
if (texture_data) {
    glTexImage2D(GL_TEXTURE_2D, 0, GL_RGBA, width, height, 0, GL_RGBA, GL_UNSIGNED_BYTE, texture_data); // 这里强制使用RGBA
    if (mipmap) glGenerateMipmap(GL_TEXTURE_2D);
}
else std::cout << "Error in load texture file: " << texture_file_name.c_str() << std::endl;
stbi_image_free(texture_data);
glBindTexture(GL_TEXTURE_2D, 0);
```

## 骨骼重定向
- [文档一](https://zhuanlan.zhihu.com/p/25064011)

## PBR渲染得到的画面过白的问题
- 可能是太早进行hdr和gamma校正，把所有的hdr和gamma校正都放到最后一步做就可以了

## 关于添加音频
- 用[irrKlang](https://www.ambiera.com/irrklang/downloads.html)这个库就可以，超级好用

# Unity
## Toggle的文字不显示
- [ContentSizeFitter有时会刷新失败](https://forum.unity.com/threads/content-size-fitter-refresh-problem.498536/)
- 所以能不用就不用吧
```
// 一种可以保证正常的刷新方式
    Canvas.ForceUpdateCanvases();
    GridLayoutGroup[] grids = Panel.GetComponentsInChildren<GridLayoutGroup>();
    ContentSizeFitter[] fitters = Panel.GetComponentsInChildren<ContentSizeFitter>();

    foreach (ContentSizeFitter fit in fitters)
    {
        fit.enabled = false;
    }
    foreach (GridLayoutGroup grid in grids)
    {
        grid.spacing = new Vector2(26, 7);
        grid.padding.left = 10;
        grid.padding.right = 10;
        grid.padding.top = 0;
        grid.padding.bottom = 20;
        LayoutRebuilder.ForceRebuildLayoutImmediate(grid.gameObject.GetComponent<RectTransform>());
    }
    foreach (ContentSizeFitter fit in fitters)
    {
        fit.enabled = true;
    }
```

## onScroll事件无法穿透
- 直接在一个Button上添加一个EventTrigger, 会导致所有事件被这个Button截获, 如果这个Button放在一个ScrollRect里, 那么当鼠标停留在这个Button上的时候, ScrollRect将不能滚动
- [参考资料](https://forum.unity.com/threads/solved-how-to-detect-double-tap-on-ui-button-and-change-interval.289672/)
- [参考资料](https://forum.unity.com/threads/child-objects-blocking-scrollrect-from-scrolling.311555/)

## Unity 2018.4的VideoPlayer在安卓10上无法播放本地视频
- 是Unity本身的bug, 安卓10更新了权限管理策略导致的, 无法修复
- [Issue&Patch](https://issuetracker.unity3d.com/issues/android-video-player-cannot-play-files-located-in-the-persistent-data-directory-on-android-10?_ga=2.168257129.998468620.1593483418-868846815.1563444103)
- [讨论](https://forum.unity.com/threads/error-videoplayer-on-android.742451/)
- 顺便学习了一下adb打unity应用的log
```
adb connect ip # 连接设备
adb -s ip:port logcat -s Unity # 查看Unity输出
```

## 美术资源复制到程序资源文件夹, prefab中引用丢失
- 传资源的时候需要把meta文件也一起传, meta文件里有guid, 然后unity是根据guid确定的引用关系, 但是猎奇的是, 对于相同MD5的同名文件, unity两次生成的meta文件里的guid不一样!!!

## [一篇讲Unity音效基础的很好的资料](https://gameinstitute.qq.com/community/detail/128887)

## [RuntimeNavMeshBuilder: Source mesh xxxxxxxxxxxxxxx does not allow read access]
- [原因是对应的fbx文件没有开启**读写权限**](https://forum.unity.com/threads/runtimenavmeshbuilder-source-mesh-xxxxxxxxxxxxxxx-does-not-allow-read-access.512563/)

## [Unity加载png文件为贴图]
1. [读取png文件为`byte[]`](https://stackoverflow.com/a/23657493/4291968)
2. [加载`byte[]`到texture2d.LoadImage函数](https://docs.unity3d.com/ScriptReference/ImageConversion.html)

## [Unity IMGUI的TreeView怎么override KeyEvent获取Key](https://raw.githubusercontent.com/Unity-Technologies/UnityCsReference/master/Editor/Mono/BuildPlayerSceneTreeView.cs)

## [Maya在新场景中自动创建模型并导出为fbx文件]
```
# -*- coding: utf-8 -*-
import os
import random
import sys

def GenRandomFbx(id):
    # Launch Env
    import maya.standalone
    maya.standalone.initialize("Python")
    import maya.cmds as cmds
    import maya.mel as mel
    cmds.loadPlugin('fbxmaya')

    # Gen Model
    x = 0
    y = 0
    cmds.polyCube(name = 'cube')
    for i in range(5):
        cmds.polyCube(name = 'cube' + str(i))
        x += random.randint(5, 10)
        y += random.randint(1, 5)
        cmds.move(x, y, 0)
    cmds.select(all = True)
    cmds.scale(random.randint(1, 7), random.randint(1, 7), random.randint(1, 7))
    
    # Save to File
    cmd = 'FBXExport -f "D:/workspace/TestMayaScript/test' + str(id) + '.fbx"'
    mel.eval(cmd)

# Generate Random Fbx
currentId = len(os.listdir('D:/workspace/TestMayaScript/'))
GenRandomFbx(currentId)
```

## [Unity编辑器报错`Dimensions of color surface does not match dimensions of depth surface`](https://forum.unity.com/threads/bug-dimensions-of-color-surface-does-not-match-dimensions-of-depth-surface.849676/#post-6187809)
- 解决方案: 把SceneView里面的灯光关了

## [Python里读取不同编码的文件](https://docs.python.org/3/library/codecs.html)
```
def ReadFile(filePath):
    encodings = ['utf_8', 'ascii', 'big5', 'big5hkscs', 'gb2312', 'gbk', 'gb18030', 'hz', 'utf_16', 'utf_16_be', 'utf_16_le', 'utf_7', 'cp037', 'cp424', 'cp437', 'cp500', 'cp737', 'cp775', 'cp850', 'cp852', 'cp855', 'cp856', 'cp857', 'cp860', 'cp861', 'cp862', 'cp863', 'cp864', 'cp865', 'cp866', 'cp869', 'cp874', 'cp875', 'cp932', 'cp949', 'cp950', 'cp1006', 'cp1026', 'cp1140', 'cp1250', 'cp1251', 'cp1252', 'cp1253', 'cp1254', 'cp1255', 'cp1256', 'cp1257', 'cp1258', 'euc_jp', 'euc_jis_2004', 'euc_jisx0213', 'euc_kr', 'iso2022_jp', 'iso2022_jp_1', 'iso2022_jp_2', 'iso2022_jp_2004', 'iso2022_jp_3', 'iso2022_jp_ext', 'iso2022_kr', 'latin_1', 'iso8859_2', 'iso8859_3', 'iso8859_4', 'iso8859_5', 'iso8859_6', 'iso8859_7', 'iso8859_8', 'iso8859_9', 'iso8859_10', 'iso8859_13', 'iso8859_14', 'iso8859_15', 'johab', 'koi8_r', 'koi8_u', 'mac_cyrillic', 'mac_greek', 'mac_iceland', 'mac_latin2', 'mac_roman', 'mac_turkish', 'ptcp154', 'shift_jis', 'shift_jis_2004', 'shift_jisx0213']
    
    count = 0
    for e in encodings:
        with open(filePath, 'r', encoding = e) as fh:
            try:
                content = fh.read()
            except UnicodeDecodeError:
                # Nothing
                count += 1
            except UnicodeError:
                # Nothing
                count += 1
            else:
                return content, e
```

## [关于Unity的MenuItem的Priority的排列](http://answers.unity.com/answers/810239/view.html)

## [UI画线]
- UILineRenderer(Unity UI Extension库的)

