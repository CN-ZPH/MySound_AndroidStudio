Fork修改版  新增v7平台下可用性 升级gradle配置 5.6.4


> 版权声明：转载必须注明本文转自张鹏辉的博客: http://blog.csdn.net/qingtiangg
> <br>
> 大家好，距离上一篇博客半年过去了，关于上一篇博客很多人加我QQ留言问了几个问题，最近不忙决定解决一下，并写几个文档总结下这半年的经验，这篇文档主要是记录下移植的过程。


# 统计


1：因为半年前代码用Eclipse写的，有人问我能不能移植到Android Studio 上，可以

2：变声后的文件在哪里？这么保存？问这个问题的肯定没有认真阅读我上一篇博客和看源码。（我这里提供一种解决方案，在最下面认真往下看）

<br>
<br>


#开始


Android Studio NDK目前有两种玩法:
1： ndk-build 、Android.mk、 Application.mk 
2： CMake

我也不介绍他俩对比了，第二个是android studio2.2之后主推的，新建ndk项目直接勾选上可以玩了。
既然要移植到android studio上我们也用第二种

开撸：

##1.创建新项目（Create New Project）
  勾选上 <font color=gray size=2>Include C++ Support</font>

![这里写图片描述](http://img.blog.csdn.net/20170803150811282?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvUWluZ1RpYW5HRw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

接下来和普通创建一样一路 ---->Next到下图这个页面
##2.配置C++（Customize C++ Support）

你们 C++ Standard 这里应该是默认的<font color=gray size=2>Toolchain Default </font>默认是CMake环境，这里我们用C++11没有为什么，任性。
 勾选上 <font color=gray size=2>Exceptions Support 、Exceptions Support</font>让其支持C++异常处理然后<font color=gray size=2>Finish</font>如下图：

![这里写图片描述](http://img.blog.csdn.net/20170803152517457?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvUWluZ1RpYW5HRw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

>项目创建好，把他帮我们生成的文件删掉，布局以及MainActivity里的不用代码，还有cpp目录下的.cpp文件不需要，之后我们开始做移植吧。

##3.拷贝资源文件到 Android Studio

![这里写图片描述](http://img.blog.csdn.net/20170803154711051?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvUWluZ1RpYW5HRw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

首先看下Eclipse工程下的目录结构：

 - src 目录下的java代码
 - assets 目录下的音频资源
 - jni 目录下的fmod的动态库和c++代码
 - lib 目录下的fmod包
 - res 目录下的图片及布局xml代码和一些资源文件


这些复制黏贴的活我就不贴出来了，主要看下jni目录
刚才创建完成项目后，AS自动帮创建了cpp目录，原jni目录下的inc直接拷过去就好其他不要看图：

![这里写图片描述](http://img.blog.csdn.net/20170803160708862?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvUWluZ1RpYW5HRw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

在Android studio 里app->libs目录下创建armeabi平台文件夹把libfmod.so和libfmodL.so这俩包放进去看下AS的现在的结构：

![这里写图片描述](http://img.blog.csdn.net/20170803161129712?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvUWluZ1RpYW5HRw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

>fmod.jar放到libs下右键add进去、assets这个目录as没有我们需要自己去创建把音频文件放进去。



##4.Android Studio 下生成.h头文件

现在剩下头文件和supersound.cpp这俩关键文件了，因为我包结构不一样所以需要重新生成头文件。
我看了几篇文档AS下有配置直接生成头文件的，我懒不配置了，直接命令行玩吧！

 1. 从Android Studio的Terminal里进入到， <Project>/src/main/java 目录下，一定要刀这个文件夹下来执行命令操作。
 2. 执行javah XXX.XX.XXX 即可，   XXX.XX.XXX是要生成.h文件的完整路径名，包名+文件名

>cd 到目录下 执行javah命令会在当前文件夹下生成.h文件F5刷新目录就有了,把他拷贝到cpp目录下，把原来的supersound.cpp也拷过来，如下图 

![这里写图片描述](http://img.blog.csdn.net/20170803162848739?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvUWluZ1RpYW5HRw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
---
![这里写图片描述](http://img.blog.csdn.net/20170803163014257?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvUWluZ1RpYW5HRw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

>把.h文件里的这一行代码拷贝到我们原来supersound.cpp文件里替换标红这段代码，以及修改引用头文件的目录地址#include  那里地址变了，如下图：
>

![这里写图片描述](http://img.blog.csdn.net/20170803163244689?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvUWluZ1RpYW5HRw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

>至此我们资源文件拷贝完毕

##5.修改CMakeList.txt构建ndk
>看下CMakeList.txt的我直接写好了，简单介绍下
```
# Sets the minimum version of CMake required to build the native
# library. You should either keep the default value or only pass a
# value of 3.4.0 or lower.

cmake_minimum_required(VERSION 3.4.1)

find_library( # Sets the name of the path variable.
              log-lib

              log )

set(distribution_DIR ${CMAKE_SOURCE_DIR}/../../../../libs)

add_library( fmod
             SHARED
             IMPORTED )
set_target_properties( fmod
                       PROPERTIES IMPORTED_LOCATION
                       ../../../../libs/armeabi/libfmod.so )
add_library( fmodL
             SHARED
             IMPORTED )
set_target_properties( fmodL
                       PROPERTIES IMPORTED_LOCATION
                       ../../../../libs/armeabi/libfmodL.so )
add_library( sound
             SHARED
             src/main/cpp/supersound.cpp )

include_directories(src/main/cpp/inc)

target_link_libraries( sound fmod fmodL
                       ${log-lib} )
```

这里不介绍Cmake的使用了附上学习地址官方文档

>https://cmake.org/documentation/

>中文翻译的简易的 CMake手册

>https://www.zybuluo.com/khan-lau/note/254724


最后一步配置build.gradle直接上图了：

![这里写图片描述](http://img.blog.csdn.net/20170803165431429?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvUWluZ1RpYW5HRw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

>可以编译安装玩一玩了
>哈哈哈真是愉快的一下午啊

##6.结束语-变声后保存解决思路
关于你们说的要保存变声后的声音
不知道你们为什么要保存下来哈哈哈
因为是在播放的时候加入的音轨就像唱歌的时候加入伴奏加入一段钢琴的声音达到变声效果，加了特效。

目前我们是在本机播放变声后的效果，有个哥们他的需求是这边发出去语音对方听到的是变声后效果。然后问我怎么保存说找不到解决方法所以来问我。

>我这里给的思路就是我们本地不保存，就把原声发送给接收方，在接收方调fmod进行变声，这么说理解了吗？
>发送的时候加上tag标示，这个标示代表用那种变声效果，然后对方点击播放的时候你把他变声了就好了啊！
有时候换一种思路你会发现真的很简单，没必要一直纠结
关键还是在于懒，懒得查资料，那么我们就用简单的方法
>这只是一次移植的记录，所以资料我写的不是很详细，如果需要学习哪一方面可以找我，我写教程或者帮你找相关资料都可以。

>边移植边查资料还要记录所以文档很乱，有问题来找我QQ：1344670918或者下面留言

##项目源码下载地址：

https://github.com/CN-ZPH/
 觉得不错请点一个star蛤！
 有问题下面留言评论，我看到会回复。


