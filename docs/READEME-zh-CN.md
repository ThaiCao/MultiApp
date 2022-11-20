# MultiApp

![](https://img.shields.io/badge/license-AGPL3.0-brightgreen.svg?style=flat)
![](https://img.shields.io/badge/Android-7.0%20--%2013-blue.svg?style=flat)
![](https://img.shields.io/badge/arch-armeabi--v7a%20%7C%20arm64--v8a-blue.svg?style=flat)

## 项目简介
MultiApp是一款虚拟安卓容器，可实现app多开，该工程提供了一个简易的UI供您体验，您也可以下载[官网安装包](https://play.google.com/store/apps/details?id=com.waxmoon.ma.gp)享受更流畅的体验。如果您是一个android开发者，也可以自已定制UI，您不用关注底层的实现细节，使用[opensdk](https://github.com/WaxMoon/opensdk)提供的api即可实现app多开。如果您在体验过程中有任何问题，可直接咨询微信账号。

另外，团队会持续修复问题以及更新opensdk，为您提供最佳的体验。

### **您可以观看如下视频了解我们的功能**

[免安装google play](https://github.com/WaxMoon/MultiApp/blob/5fc33308ca9fd651ce7be2a5bab53160d5303426/docs/res/github_gp.mp4) <----> [多开facebook](https://github.com/WaxMoon/MultiApp/blob/5fc33308ca9fd651ce7be2a5bab53160d5303426/docs/res/github_fb.mp4)
<div align=center>
    <video width="320" height="320" controls>
        <source src="res/github_gp.mp4" type="video/mp4">
    </video>
    <video width="320" height="320" controls>
        <source src="res/github_fb.mp4" type="video/mp4">
    </video>
</div>

## 方案简介
传统的多开方案依赖于java动态代理、inline hook、代理转发等手段保证虚拟进程的正常运行。如果三方app同样使用了java动态代理，此时会存在代理相互覆盖的问题，该情况会导致三方app运行时的逻辑发生变化。逻辑‘被’发生变化是极大多数厂家不愿意看到的，可能在一定程度上影响其收益。所以从某种意义上来讲，传统方案并不能定义为容器。

MultiApp技术选型之初就抛弃了动态代理，service、receiver、provider等binder组件也均由MultiApp engine自行维护。很遗憾的是Activity组件必须通过代理维护其生命周期，但是我们使用了更底层的机制确保不会影响app的运行时逻辑。另外，在native hook技术上，我们基于seccomp/bpf自研了更为有效的svc hook方案，并在某些场景下启用，综合来看更接近沙盒容器的概念。

### 下面我做了两个简单的实验

**1) 在Activity.onCreate函数中打印代码栈，并分别用传统多开软件、MultiApp、本机打开运行**

```Java
public class MainActivity extends ButtonActivity {

    final static String TAG = "MainActivity";

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        new Exception().printStackTrace();
    }
}
```

第一张图片为传统多开软件的异常代码栈，第二张为MultiApp运行的代码栈，第三张为系统本机运行的代码栈。

MultiApp与本机的代码栈一致。

![image](res/stack_other.png)

![image](res/stack_MultiApp.png)

![image](res/stack_system.png)

**2) 判断ActivityManager的binder接口是否被动态代理，并分别在传统多开软件、MultiApp中运行**

```Java
public class MainActivity extends ButtonActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        new Exception().printStackTrace();

        {
            try {
                Class<?> class_ActivityManagerNative = Class.forName("android.app.ActivityManagerNative");

                Method method_getDefault = class_ActivityManagerNative.getDeclaredMethod("getDefault");

                IInterface remote_ActivityManager = (IInterface) method_getDefault.invoke(null);

                boolean isProxy = remote_ActivityManager instanceof Proxy;

                Log.d("WaxMoon", String.format("ActivityManager(%s) is proxy: %s", remote_ActivityManager, isProxy));

            } catch (Exception ignore) {

            }
        }
    }
}
```

传统多开日志
```Text
11-25 17:56:38.823  5153  5153 D WaxMoon : ActivityManager(android.app.IActivityManager$Stub$Proxy@8abaec7) is proxy: true
```

MultiApp日志
```Texx
11-25 17:59:13.804  8197  8197 D WaxMoon : ActivityManager(android.app.IActivityManager$Stub$Proxy@79f3e55) is proxy: false
```

## 功能特性
*  支持android7-android13(android7-8还在测试中，敬请谅解)
*  支持armv7-32, armv8-64
*  主包/辅包
*  支持google play
*  app多开
*  支持app免安装(私密安装)

## 商业版功能
*  功能定制
*  授权
*  其他...


## 开源协议须知
该仓库包括[opensdk](https://github.com/WaxMoon/opensdk)均使用AGPL-3.0协议，您必须遵守该协议。您在对外发布软件之前，请务必联系微信告知您的想法，在某些情况下您可以自由免费使用。

## 安全须知
出于代码安全以及行业安全的角度，我们暂时**禁用了软件调试**功能。如果您有相关的合法需求，可以通过下述方式咨询合作。

## 联系方式
微信账号:WaxMoon2018

新浪邮箱:cocos_sh@sina.com