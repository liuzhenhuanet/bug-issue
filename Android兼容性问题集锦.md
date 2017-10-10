# Android兼容性问题集锦
Android相比iOS开发困难的地方在于Android手机厂商众多，第三方ROM众多，版本分裂严重，导致UI适配
和非UI兼容问题多而繁杂。本文记录了作者在Android开发中遇到的兼容性问题，包括UI的和非UI的。

作者邮箱：<liuzhenhuanet@gmail.com>  github：[https://github.com/liuzhenhuanet](https://github.com/liuzhenhuanet)  个人网站： [http://blog.liuzhenhua.net](http://blog.liuzhenhua.net)

### Application中判断进程名字
在`Application`的`onCreate`方法里，一般会进行很多初始化工作，包括自己的代码以及
第三方库代码的初始化工作。我们知道`onCreate`方法会被运行多次，每次启动一个进程都会
被调用一次，那么在不同进程中初始化工作肯定不一样，所以就需要在哪`onCreate`中判断进程，
例如是否是主进程，是否是推送进程，是否是后台其他进程。每个进程中只做必要的初始化工作可以
避免资源的浪费。那么如何在`onCreate`判断进程呢？调用`ActivityManager.getRunningAppProcesses()`
方法即可。

但是这是有问题的，在5.1.1以及以上版本中，改方法只会返回自己APP的进程信息，这不是重点，
重点是某些机型可能返回`null`，所以保险的方法是使用下面的代码获取当前进程名字：
```java
@Nullable
public static String currentProcess(Context context) {
    int pid = android.os.Process.myPid();
    String processName = null;
    ActivityManager mActivityManager = (ActivityManager) context.getSystemService(Context.ACTIVITY_SERVICE);
    List<ActivityManager.RunningAppProcessInfo> runningAppProcesses = mActivityManager.getRunningAppProcesses();
    // 某些机型调用ActivityManager.getRunningAppProcesses()会返回null
    if (CollectionUtil.isEmpty(runningAppProcesses)) {
        return null;
    }
    for (ActivityManager.RunningAppProcessInfo appProcess : runningAppProcesses) {
        if (appProcess.pid == pid) {
            processName = appProcess.processName;
            break;
        }
    }
    return processName;
}
```
如果返回null，那么不管是应该在主进程中需要初始化的工作还是其他进程中需要初始化的工作都做一遍，
一方万一，避免某些机型不能正常初始化。

### API 25中的WindowManager.LayoutParams.TYPE_TOAST类型window
在API 25，也就是Android7.1.1中，如果同时显示多个类型为`WindowManager.LayoutParams.TYPE_TOAST`
的window，会抛出`android.view.WindowManager$BadTokenException: Unable to add window -- window android.view.ViewRootImpl$W@93d8626 has already been added`
异常导致应用崩溃。  
这个应该是API 25中的bug，因为在API 24和26中都不会出现这种情况，文档中也没有说明会这种
情况会抛出异常。  

如何发现这个问题的？  
我们的应用中的Toast使用的自定义的Toast，从屏幕顶部弹出，类似iOS系统通知。但某些代码中
依然使用了系统的Toast，这某个情景下这两种Toast会同时弹出，由于他们不知道彼此，不能彼此
取消，所以在API 25的机器上导致了崩溃。  

### 敬请期待...