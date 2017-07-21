## Activity的坑——不为人知的故事
activity作为Android四大组件之一，四大组件中平时我们接触的最多的也就是Activity了，然而我们真的了解Activity吗？本为
就是收集Activity不为人知的一面。

### 转场动画之`overridePendingTransition`
好的交互体验应该缺少不了页面之间的切换动画，Android中设置页面切换动画有多种方式，其中之一就是调用`overridePendingTransition`
方法。
根据官方文档，这个方法必须在`Activity`的`startActivity`或者`finish`方法调用之后立刻调用。但是在实际开发过程中发现，
这样的限制给我们带来了诸多不便，例如应用中的第一个启动的Activity就不能使用`overridePendingTransition`方法设置启动
动画，还有其他不方便的地方。
`overridePendingTransition`方法有两个参数，第一个参数表示页面进入时的动画，第二个参数表示页面退出时的动画，一般情况下
Android屏幕上每次只能展示一个页面，所以每次一个页面进入总伴随着一个页面退出，之前我对`overridePendingTransition`方法
的理解还停留在使用`startActivity`启动页面时只有通过`overridePendingTransition`设置的第一个参数，即启动动画有效，
第二个参数，即退出动画会被忽略，但其实不是这样的，这两个参数都会生效，当我们使用`startActivity`启动页面时，两个参数设置
的动画会分别应用在将要启动的页面和当前页面。
既然这样，我发现在Activity的`onCreate`方法中，Activity实际上还没显示出来，上一个Activity的生命周期也还没有走到onDestroy，
也就是说，当前页面还没有显示，上一个页面还没有退出，那么这个时候调用`overridePendingTransition`设置页面切换动画是不是
应该有效呢，实践是检验真理的唯一标准，我试了一下，果然成功了。我在网上查了一下，还真有这样做的。
既然可以放在onCreate中调用，那么Google为什么在文档里说必须紧接着`startActivity`或者`finish`方法后面调用呢？直到今天，
我才明白了Google的良苦用心——我的`onCreate`方法里调用`overridePendingTransition`之前添加了友盟推送的`PushAgent.getInstance(this).onAppStart();`
这段代码，结果我的页面切换动画失效了。最后总结原因：应该是友盟的这个方法调用相对耗时，而在Android上启动页面是需要进程间通信的，
等友盟的这段代码执行完，执行`overridePendingTransition`之前，页面切换动画参数已经传给了ActivityManager了，并没有
使用我通过`overridePendingTransition`方法设置的动画。
所以，想要在`onCreate`里调用`overridePendingTransition`需要尽可能保证`overridePendingTransition`方法之前没有
耗时操作，最好把`overridePendingTransition`放在第一行，最好同时在`super.onCreate`之前。