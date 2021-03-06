<p align="center">
  <img src="ART/android_god_eye_logo.png" width="256" height="256" />
</p>

<h1 align="center">AndroidGodEye</h1>
<p align="center">
<a href="https://travis-ci.org/Kyson/AndroidGodEye" target="_blank"><img src="https://travis-ci.org/Kyson/AndroidGodEye.svg?branch=master"></img></a>
<a href="http://androidweekly.net/issues/issue-293" target="_blank"><img src="https://img.shields.io/badge/Android%20Weekly-%23293-blue.svg"></img></a>
<a href="https://android-arsenal.com/details/1/6561" target="_blank"><img src="https://img.shields.io/badge/Android%20Arsenal-AndroidGodEye-brightgreen.svg?style=flat"></img></a>
<a href="LICENSE" target="_blank"><img src="http://img.shields.io/badge/license-Apache2.0-brightgreen.svg?style=flat"></img></a>
</p>
<br/>

<p>
<a href="README.md">English README.md</a>&nbsp;&nbsp;&nbsp;
<a href="README_zh.md">中文 README_zh.md</a>
</p>

> Android开发者在性能检测方面的工具一直比较匮乏，仅有的一些工具，比如Android Device Monitor，使用起来也有些繁琐，使用起来对开发者有一定的要求。而线上的App监控更无从谈起。所以需要有一个系统能够提供Debug和Release阶段全方位的监控，更深入地了解对App运行时的状态。

## 概览

![android_godeye_connect](ART/android_god_eye_connect.jpg)

AndroidGodEye是一个可以在PC浏览器中实时监控Android数据指标（比如性能指标，但是不局限于性能）的工具，你可以通过wifi/usb连接手机和pc，通过pc浏览器实时监控手机性能。

系统分为三部分：

1. Core 核心部分，提供所有模块
2. Debug Monitor部分，提供Debug阶段开发者面板
3. Toolbox 快速接入工具集，给开发者提供各种便捷接入的工具

AndroidGodEye提供了多种监控模块，比如cpu、内存、卡顿、内存泄漏等等，并且提供了Debug阶段的Monitor看板实时展示这
些数据。而且提供了api供开发者在release阶段进行数据上报。

## 快速开始

参考Demo:[https://github.com/Kyson/AndroidGodEyeDemo](https://github.com/Kyson/AndroidGodEyeDemo)

### STEP1

引入依赖，使用gradle

```
dependencies {
  implementation 'cn.hikyson.godeye:godeye-core:VERSION_NAME'
  debugImplementation 'cn.hikyson.godeye:godeye-monitor:VERSION_NAME'
  releaseImplementation 'cn.hikyson.godeye:godeye-monitor-no-op:VERSION_NAME'
  implementation 'cn.hikyson.godeye:godeye-toolbox:VERSION_NAME'
}
```

> VERSION_NAME可以看github的release名称

### STEP2

Application中初始化:

```java
GodEye.instance().init(this);
```

模块安装，GodEye类是AndroidGodEye的核心类，所有模块由它提供。

在应用入口安装所有模块：

```java
// v1.7以下
// GodEye.instance().installAll(getApplication(),new CrashFileProvider(context))
// v1.7.0以上installAll api删除，使用如下：
if (isMainProcess(this)) {//安装只能在主进程
        GodEye.instance().install(Cpu.class, new CpuContextImpl())
                .install(Battery.class, new BatteryContextImpl(this))
                .install(Fps.class, new FpsContextImpl(this))
                .install(Heap.class, Long.valueOf(2000))
                .install(Pss.class, new PssContextImpl(this))
                .install(Ram.class, new RamContextImpl(this))
                .install(Sm.class, new SmContextImpl(this, 1000, 300, 800))
                .install(Traffic.class, new TrafficContextImpl())
                .install(Crash.class, new CrashFileProvider(this))
                .install(ThreadDump.class, new ThreadContextImpl())
                .install(DeadLock.class, new DeadLockContextImpl(GodEye.instance().getModule(ThreadDump.class).subject(), new DeadlockDefaultThreadFilter()))
                .install(Pageload.class, new PageloadContextImpl(this))
                .install(LeakDetector.class,new LeakContextImpl2(this,new RxPermissionRequest()));
}


/**
* 是否主进程
*/
    private static boolean isMainProcess(Application application) {
        int pid = android.os.Process.myPid();
        String processName = "";
        ActivityManager manager = (ActivityManager) application.getSystemService
                (Context.ACTIVITY_SERVICE);
        for (ActivityManager.RunningAppProcessInfo process : manager.getRunningAppProcesses()) {
            if (process.pid == pid) {
                processName = process.processName;
            }
        }
        return application.getPackageName().equals(processName);
    }
```

> 推荐在application中进行安装，否则部分模块可能工作异常

#### 可选部分

不需要的时候卸载模块(不推荐)：

```java
// v1.7以下
// GodEye.instance().uninstallAll();
// v1.7.0以上uninstallAll api删除，不提供便捷的卸载方法，如果非要卸载：
GodEye.instance().getModule(Cpu.class).uninstall();
```

> 注意：network和startup模块不需要安装和卸载

安装完之后相应的模块就开始输出数据了，一般来说可以使用模块的consume方法进行消费，比如cpu模块：

```java
// v1.7以下
// GodEye.instance().cpu().subject().subscribe()
// v1.7.0以上使用class获取对应模块
GodEye.instance().getModule(Cpu.class).subject().subscribe();
```

> 就像我们之后会提到的Debug Monitor，也是通过消费这些数据进行展示的

### STEP3

Debug面板安装，GodEyeMonitor类是AndroidGodEye的Debug监控面板的主要类，用来开始或者停止Debug面板的监控。

开始消费GodEye各个模块数据并输出到Debug面板：

```java
GodEyeMonitor.work(context)
```

结束消费，关闭Debug面板：

```java
GodEyeMonitor.shutDown()
```

### STEP4

完成！开始使用：

手机与pc连接同一网段，在pc浏览器中访问`手机ip+端口+/index.html`。或者如果你是用USB连接的话，执行`adb forward tcp:5390 tcp:5390`，然后pc浏览器中访问`http://localhost:5390/index.html`。

即可看到Debug面板!

> 端口默认是5390，也可以在`GodEyeMonitor.work(context)`中指定，一般在开发者在调用`GodEyeMonitor.work(context)`之后可以看到日志输出 'Open AndroidGodEye dashboard [ http://xxx.xxx.xxx.xxx:5390/index.html" ] in your browser...' 中包含了访问地址。

**好吧，如果你懒得自己编译这个项目的话，你也可以先下载 [APK](https://fir.im/5k67) 看看效果。**

**注意：/index.html 是必须的!!!**

## Debug开发者面板

###### 点击下面预览↓

<p>
<a href="https://player.youku.com/embed/XMzIwMTgyOTI5Mg==" target:"_blank">
<img border="0" src="ART/android_god_eye_play.png" width="128" height="128" />
</a>
</p>

### Base info

![android_godeye_summary](ART/android_god_eye_summary.png)

### 卡顿检测

![android_god_eye_block](ART/android_god_eye_block.gif)

### 内存泄漏检测

![android_god_eye_leak](ART/android_god_eye_leak.gif)

### 更多模块

![android_god_eye_cpuheaptraffic](ART/android_god_eye_cpuheaptraffic.gif)

还有更多...

## 模块详情

|模块名|需要安装|数据引擎|数据生产时机|权限|
|-----|------|-------|----------|---|
|cpu|是|内置|定时|无|
|battery|是|内置|定时|无|
|fps|是|内置|定时|无|
|leakDetector|是|内置|发生时|WRITE_EXTERNAL_STORAGE|
|heap|是|内置|定时|无|
|pss|是|内置|定时|无|
|ram|是|内置|定时|无|
|network|否|外部驱动|-|无|
|sm|是|内置|发生时|无|
|startup|否|外部驱动|-|无|
|traffic|是|外部驱动|定时|无|
|crash|是|外部驱动|安装后，一次性|无|
|thread dump|是|内置|定时|无|
|deadlock|是|内置|定时并发生时|无|
|pageload|yes|internal|happen|无|

## 框架

下图可以更清楚地解释AndroidGodEye是如何工作的：

![android_god_eye_framework](ART/android_god_eye_framework.jpeg)

## 许可协议

AndroidGodEye使用 Apache2.0 许可协议。

## 关于我

- Github: [Kyson](https://github.com/Kyson)
- Weibo: [hikyson](https://weibo.com/hikyson)
- Blog: [tech.hikyson.cn](https://tech.hikyson.cn/)









