---
layout: post
title:  "UE Knowledge Base"
date:   2023-09-27
categories: blogs
author: 2bdenny
---

---
### Actor
- [怎么检查对象的类型](https://forums.unrealengine.com/t/how-to-check-if-an-actor-is-from-a-certain-class/278582/5?u=2bdenny)
- [UE里面动态创建Actor然后逐个add component跟Unity里面new GameObject不一样](https://blog.csdn.net/qq_16756235/article/details/53157914)

### Audio - 音频
- [怎么在切换关卡时保留Actor, 以及怎么让WWise的Listener Actor一直存在于关卡中](https://www.cnblogs.com/lingchuL/p/14751703.html#5184301)
- [怎么监听event结束事件?]()
```
class ATest : public AActor
{
public:
    FOnAkPostEventCallback OnAkPostEventDelegate;

    virtual void BeginPlay() override;
    virtual void EndPlay(const EEndPlayReason::Type EndPlayReason) override;
    
    UFUNCTION()
    void OnAkPostEventCallback(EAkCallbackType CallbackType, UAkCallbackInfo* CallbackInfo);

    void LetWwiseSpeakSomething(AActor* Actor, UAkAudioEvent* AkEvent);
};

void ATest::BeginPlay()
{
    OnAkPostEventDelegate.BindDynamic(this, &ATest::OnAkPostEventCallback);
}

void ATest::EndPlay(const EEndPlayReason::Type EndPlayReason)
{
    OnAkPostEventDelegate.Clear();
}

void ATest::LetWwiseSpeakSomething(AActor* Actor, UAkAudioEvent* AkEvent)
{
    UAkGameplayStatics::PostEvent(AkEvent, Actor, AK_EndOfEvent, OnAkPostEventDelegate);
}
```
- [怎么设置Wwise工程的ignore文件](https://blog.audiokinetic.com/en/version-control-a-wwise-project/)

### Blueprint - 蓝图
- [怎么定义多output pin的蓝图节点](https://forums.unrealengine.com/t/how-to-define-a-function-with-multiple-return-values-as-blueprint-in-c/16934/3?u=2bdenny)
- [蓝图怎么检测null](https://forums.unrealengine.com/t/blueprints-null-actor/277986/2?u=2bdenny)

### Camera - 相机
- [UE相机基础和使用](https://zhuanlan.zhihu.com/p/564571102)
- [一种推荐的自建CameraManager模块方案](https://forums.unrealengine.com/t/setting-the-active-camera-on-an-aplayercontroller/417122/3?u=2bdenny)
- [相机的Damping问题及其解决方案](https://www.rorydriscoll.com/2016/03/07/frame-rate-independent-damping-using-lerp/)
  - [中文版本](https://zhuanlan.zhihu.com/p/630930384)

### CDO
- [怎样可以让继承UObject的类不创建CDO](https://forums.unrealengine.com/t/how-do-i-prevent-instances-of-uobjects-from-being-created-automatically/345850/2?u=2bdenny)
- [怎样让Subsystem不要Tick两次](https://forums.unrealengine.com/t/subsystem-tick-always-called-twice-per-frame-what-am-i-doing-wrong/695649/3?u=2bdenny)

### Compileing
- [VS编译]()
  - 应该选择 "Development Editor"
- [一直弹Error C4430 :默认为int之类的编译报错]()
  - https://forums.unrealengine.com/t/error-c4430-missing-type-specifier-int-assumed-note-c-does-not-support-default-int/487372/3
- [C4458: declaration of "InitialLocation" hides class member 声明隐藏了类成员](https://stackoverflow.com/a/67473926/4291968)
  - 函数参数名不可以与class的成员名相同 F**k c++
- [怎么把FString转换为std::string](https://stackoverflow.com/a/62973867/4291968)

### Collision
- 有时两个体型差别巨大的Pawn之间因为避障而出现奇怪的碰撞（小个子不能从大个子缝隙间通过等等），可以通过关闭RVOAvoidance来解决
  - 更好的方法应该是给大家分到合理的Group里面

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

### Debug
- [关于Rider通过Attach的方式Debug打不到断点](https://youtrack.jetbrains.com/issue/RIDER-81241/Cannot-set-breakpoint-after-attaching-debugger-on-UE5?_gl=1*5aoigu*_ga*NzA1NzgxNTU2LjE2ODM2MjMxNDk.*_ga_9J976DJZ68*MTY4Njg4MTI5MC4xNS4xLjE2ODY4ODE0OTIuMC4wLjA.&_ga=2.191218026.1586200720.1686799260-705781556.1683623149#focus=Comments-27-6982283.0-0)
  - [windows的说明](https://social.msdn.microsoft.com/Forums/en-US/0de597fb-9496-4297-8e23-15fd24b998f5/quotattach-to-processquot-is-not-loading-more-than-500-dlls?forum=vsdebug)
- [关于exe一打开就crash]
  - 方案1，命令行启动exe时添加参数`-WaitForDebugger`
  - 方案2
```
在main函数里面，然后给b++这个打个断点
static int b =0;
while(true)
{
     b++;
}
```

### Input
- [怎么让BindAction支持带参数的Handler]()
```
EnhancedInputComponent->BindAction<AMyClass, bool>(IA_A, ETriggerEvent::Triggered, this, &AMyClass::DoIt, true);
void AMyClass::DoIt(const FInputActionValue& Value, bool bMyCustomParameter) {}
```
- [怎么实现InputAction的覆盖]()
  - 将新的InputAction放到新的IMC里面，IMC注册的时候设置更高的Priority

### Interface
- [UE怎么实现Interface](https://docs.unrealengine.com/5.3/en-US/interfaces-in-unreal-engine/)
    - 太怪了

### Json
- [Json解析字符串到FJsonObject失败]()
  - `FJsonSerializer::Deserialize`这个函数遇到尾","就会报错
- [怎么把Json字符串转化为对应的struct](https://stackoverflow.com/a/32962292/4291968)
  - struct必须是ustruct
  - struct的field必须是uproperty
  - struct的field name必须跟json的key name完全一致
- [JsonObject怎么转换为字符串]()
```
FString OutputString;
TSharedRef< TJsonWriter<> > Writer = TJsonWriterFactory<>::Create(&OutputString);
FJsonSerializer::Serialize(JsonObject.ToSharedRef(), Writer);

UE_LOG(LogTemp, Warning, TEXT("resulting jsonString -> %s"), *OutputString);
```

### Level
- [怎么给地图设置自己的gamemode](https://forums.unrealengine.com/t/change-a-map-game-mode/144690/2?u=shen_yuju)

### Localization
- [UE里怎么使用本地化文本对UI进行本地化](https://www.youtube.com/watch?v=QOjSnqVVVV8)

### Log
- [怎么定义自己的Log分类](https://forums.unrealengine.com/t/where-do-i-put-declare-log-category-extern/302371/3)

### Math
- [Easing曲线](https://github.com/noisecrime/Unity-EasingLibraryVisualisation)

### Sequence
- [官方示例Slay Animation](https://www.unrealengine.com/en-US/spotlights/mold3d-studio-to-share-slay-animated-content-sample-project-with-unreal-engine-community)
- [黑客帝国的追逐玩法](https://youtu.be/A1Nvl2MD30U?t=98&si=U728DMyuvIzvmNy8)

### Streaming
- [怎么在切换关卡时保留Actor](https://forums.unrealengine.com/t/keeping-actors-between-levels/3085?u=2bdenny)
- [使用OpenLevel蓝图节点切换关卡导致蓝图Trigger失效](https://forums.unrealengine.com/t/opening-level-with-trigger-not-working-on-overlap/508767/2?u=2bdenny)
- [Level切换的回调](https://forums.unrealengine.com/t/how-can-i-execute-code-immediately-after-loading-a-non-streaming-level/351320?u=2bdenny)

### Tools
- [怎么用代码清空、写入、保存StringTable数据](https://pafuhana1213.hatenablog.com/entry/2023/01/14/153800)
- [一个开源的icon库](https://github.com/MahApps/MahApps.Metro.IconPacks)
- [怎么把UE的蓝图做成网页](https://blueprintue.com/)
- [C#程序怎么设置运行程序权限](https://philippsen.wordpress.com/tag/requestedexecutionlevel-manifest/)
- [让Confluence支持自己插入代码](https://marketplace.atlassian.com/apps/1215813/iframes-for-confluence?hosting=server&tab=support)
- [怎么使用7zr.exe压缩和解压缩文件，以及解压缩并保留目录结构]()
  - 7zr.exe的，一个是压缩时文件名跟压缩算法要对应，应该用7z，一个是解压参数e -o是解压并不保留文件结构，x -o才是解压并保留文件结构
- [bat脚本新建快捷方式添加icon](https://stackoverflow.com/a/59440537/4291968)
- [VS报错:MSB3821 无法处理文件 resx，因为它位于 Internet 或受限区域中，或者文件上具有 Web 标记。要想处理这些文件，请删除 Web 标记。]()
  - `Get-ChildItem -Path '<YOUR-SOLUTION-PATH>' -Recurse | Unblock-File`
- [UE怎么用脚本创建asset](https://udn.unrealengine.com/s/article/How-To-Create-New-Assets-in-C)
- [WPF怎么使用TreeView](https://wpf.2000things.com/2017/09/28/1219-expanding-all-nodes-in-a-treeview-by-default/)
- [怎么在运行时获取某个文件夹下所有某种类型的资源文件]()
    - 使用GetAssetRegistry的GetAssets节点，然后MakeARFilter，Package Paths填UE路径/Game/Levels，ClassName填写Copy Reference粘贴出来的类型名称（Blueprint或者World等等）

### UI
- [LogSlate: Warning: Slate: Had to block on waiting for a draw buffer.](https://forums.unrealengine.com/t/slate-waiting-on-draw-buffer/1295986/2?u=2bdenny)
