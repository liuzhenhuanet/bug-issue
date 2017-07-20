## weex的坑
随着react-native的兴起，国内的互联网大厂阿里巴巴也出了个类似的东西：weex，采用js语言，使用vue.js框架进行跨平台开发，
Android和ios共用一套代码。这篇文章记录作者使用weex开发Android应用中遇到的各种问题。欢迎大家讨论，也可以联系作者，作者的个人网站：
[http://blog.liuzhenhua.net](http://blog.liuzhenhua.net)，github地址：[https://github.com/liuzhenhuanet](https://github.com/liuzhenhuanet)，
邮箱[liuzhenhuanet@gmail.com](mailto://liuzhenhuanet@gmail.com)

> 作者所参考的weex文档地址：[http://weex-project.io/cn/](http://weex-project.io/cn/)，以下所有讨论均针对该文档。

### switch组件设置宽高
官方文档[http://weex-project.io/cn/references/components/switch.html](http://weex-project.io/cn/references/components/switch.html)上告诉我`switch`组件
上有些样式不能使用，分别是：
- `width`
- `height`
- `min-width`
- `min-height`
- `margin`
- `padding`
- `border`
所以我在使用`switch`组件时没有设置宽高，结果是死活都显示不出来，当我开始怀疑人生的时候，同事说你试试设置以下`width`和`height`样式，
结果神奇的事情发生了，终于显示出来了。我想唱一句：我要这铁棒有何用（我要这文档有何用）

### getComponentRect(ref, callback)
weex的内建模块dom下有一个`getComponentRect(ref, callback)`方法，文档地址[https://weex-project.io/cn/references/modules/dom.html#getComponentRect-ref-callback-v0-9-4]
(https://weex-project.io/cn/references/modules/dom.html#getComponentRect-ref-callback-v0-9-4)
,是用来获取元素的布局信息的，例如`width`，`height`，`top`等，文档里还特地举了个例子，`如果想要获取到 Weex 容器的布局信息，
可以指定 ref='viewport'`，这个例子是可以正常运行的，所以我抱着试一试的心态，给一个`div`元素设置了`ref`属性为`test`，
然后就调用`getComponentRect`方法，把`test`作为字符串传入第一个参数，结果报错了，告诉我`errMsg: "Component does not exist`。

我检查了一遍又一遍，确认我的ref属性没有写错，而且没有同名ref属性被设置，我只好设置断点去检查代码运行的上下文，结果什么信息
都没有找到，不过发现了`this.$refs.test`的属性列表里有一个属性叫做`ref`，它的值是一个字符串，但字符串是一个整数。

这时候我想起了Android里的id，Android里的id最终引用的时候其实就是一个整数，我想这里可能也是需要一个整数，最后我把`this.$refs.test.ref`
作为`getComponentRect`第一个参数传入，终于获取到了我想要的信息。

真是浪费了我好多时间，还浪费时间在这里记录一下。真的不知道为什么不直接告诉我传什么值进去，还要特地举例用字符串`viewport`作为参数，所以我天真的以为只要把我设置的`ref`属性值作为
字符串传入就能得到我想要的，fuck weex。

### scroller组件bug
有一个页面需要下拉刷新功能，所以用到了`scroller`组件以及它的子组件`refresh`，`scroller`里的内容是可以动态添加的，遇到
的问题就是页面几乎不能滚动和下拉，只有将手指放到没有子元素的位置才能使用下拉刷新功能，我猜测可能是touch事件被拦截了，最后的
解决办法也十分粗暴，就桑把`scroller`组件换成了`list`组件，另外说一句，之前的`scroller`写法在iOS上表现正常，所以这个
应该是weex在Android上特有的bug。

### boolean值引发的问题
这个问题不知道是应该归咎weex还是vue，又或者是js。具体情况是这样子的，我在一个标签上绑定了一个变量，
```
<tag :variable="value"></tag>
<other-tag :variable="!value"></other-tag>
```
value这个变量的值是从持久化存储获取的，所以读取出来是一个字符串，为"true"或者"false"，当value="false"时，第一个标签tag
的variable属性获取到的值自动转换为`false`了，而第二个标签的variable属性由于有`!`符号，首先将字符串"false"转换为
Boolean值`true`，最终variable得到的值同样是`false`，按照代码的意思来，应该期盼的是tag和other-tag的variable属性
值使用相反，但是当value='false'时却得到了相同的结果。 既然找到了原因，那修改方法也很简单，就是value获取值的时候不要直接
赋值字符串，而是先手动转换为Boolean类型的值。