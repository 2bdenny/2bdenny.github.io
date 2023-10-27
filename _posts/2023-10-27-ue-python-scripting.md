---
layout: post
title:  "How to use python speed up tool development in UE - 用Python加速UE工具开发"
date:   2023-10-27
categories: blogs
author: 2bdenny
---

---
# 用Python加速UE工具开发

> 用C++写各种资源处理代码实在是太蛋疼了（参考fbx相关的c++代码，动不动就#include几十个头文件）
> 因此Epic自带`Python 3.9`环境，也开放了unreal package的一大堆接口，方便大家进行工具开发
> 很多操作搜索通过c++怎么做是搜不到的！这个时候带上unreal python前缀，问chatgpt会有意想不到的收获

### C++怎么调用Python
- 因此这个函数的存在，就可以考虑用C++写Slate代码，然后用Python写功能，再通过它调用一下脚本

```
const FFilePath ScriptPath; // 这里的ScriptPath可以设置为一个setting的变量，然后直接通过File Selector获取
const FString ScriptArguments;
FString CommandArg;

if (!ScriptPath.FilePath.IsEmpty())
{
  CommandArg=FString::Printf(TEXT("py \"%s\""),*ScriptPath.FilePath);
  if (!ScriptArguments.IsEmpty())
  {
    CommandArg+= " " + ScriptArguments;
  }
}
else
{
  if (!ScriptArguments.IsEmpty())
  {
    CommandArg+= "py " + ScriptArguments;
  }	
}

GEngine->Exec(NULL, *CommandArg);
```

### Python怎么调用UE的引擎接口
- [配置UE的Python开发环境](https://docs.unrealengine.com/5.2/en-US/scripting-the-unreal-editor-using-python/)
  - 只需要由一个人配置上传就好了，其他人可以直接用
- [配置Python自动补全](https://docs.unrealengine.com/5.2/en-US/setting-up-autocomplete-for-unreal-editor-python-scripting/)
  - 推荐vscode，因为全免费

### Python接口查询推荐步骤
- 当不知道自己想做的功能用python怎么做的时候推荐按照以下步骤进行查询
  1. 问chatgpt: unreal python how to xxx?
  2. 尝试chatgpt给的代码，如果不行，带上其中的关键词，从[UE python doc](https://docs.unrealengine.com/5.2/en-US/PythonAPI/)查询
  3. UDN上也有少许Python开发的[文档](https://udn.unrealengine.com/s/article/UE4-Sequencer-Python-Cookbook)

### Python函数参数类型声明
- 为了获得自动补全，推荐在定义函数时，显示声明每个参数的类型

### Python的类型转换
- 每个unreal名空间下的class类型，都提供了cast函数进行转换，语法如下

```
unreal.Skeleton.cast(one_loaded_asset)
```
