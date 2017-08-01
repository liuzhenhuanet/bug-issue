# Provider的坑
Provider作为Android四大组件之一，一般开发过程中遇到的不多，不像Activity天天见，但是
有时候还是需要用到它的，只要是代码，必定有坑，所以这篇文章记录了作者开发过程中遇到的与
Provider相关的坑。

作者邮箱：<liuzhenhuanet@gmail.com> github：[https://github.com/liuzhenhuanet](https://github.com/liuzhenhuanet) 个人网站：[http://blog.liuzhenhua.net](http://blog.liuzhenhua.net) 欢迎来找我交流。

### Provider在Manifest中声明权限
Android 6.0以后加入了动态权限，仅仅在manifest文件中申明权限已经不够了，还需要在运行时
用到某个权限时动态的向用户申请。

打开相册和拍照等也都需要动态权限。我们APP才用的是一个第三方库来进行图片选择，它提供了一个
Provider，需要在manifest中申明，其中有一个属性
```xml
android:authorities="${applicationId}.provider"
```
注意到`authorities`是复数形式，应该是支持多个权限的，刚好APP中的遗留代码有一处也使用到
了该Provider，并且需要的权限是`${applicationId}.fileProvider`，于是我去到
[Android 开发者官网](https://developer.android.com/guide/topics/manifest/provider-element.html#auth)
去看了下申明多个权限的语法是怎样的，我看到了这样一句描述
> Multiple authorities are listed by separating their names with a semicolon.

说多个权限之间使用分号分隔，于是我在manifest文件中这样写：
```xml
android:authorities="${applicationId}.provider;${applicationId}.fileProvider"
```
结果，安装启动失败。最后不得不妥协了，将遗留代码里的权限也改成了`${applicationId}.provider`，

不知道是不是我英语水平有限还是理解能力问题，按照官方文档写不能达到预料的结果，如果有哪位
能指点一下，万分感激。

### 敬请期待……