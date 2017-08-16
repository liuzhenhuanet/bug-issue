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
```html
<tag :variable="value"></tag>
<other-tag :variable="!value"></other-tag>
```
value这个变量的值是从持久化存储获取的，所以读取出来是一个字符串，为"true"或者"false"，当value="false"时，第一个标签tag
的variable属性获取到的值自动转换为`false`了，而第二个标签的variable属性由于有`!`符号，首先将字符串"false"转换为
Boolean值`true`，最终variable得到的值同样是`false`，按照代码的意思来，应该期盼的是tag和other-tag的variable属性
值使用相反，但是当value='false'时却得到了相同的结果。 既然找到了原因，那修改方法也很简单，就是value获取值的时候不要直接
赋值字符串，而是先手动转换为Boolean类型的值。

### 不支持Android View的`View.INVISIBLE`和`View.GONE`
背景是这样的，我之前在做一个搜索页，搜索结果分为好几类，需要在同一个页面内使用标签页分别
展示“综合搜索结果”、“类型1搜索结果”和“类型2搜索结果”。最初使用了3个list标签分别存放不
同类别的结果，为每个list设置`v-if`属性控制显示当前标签下的结果。最终的效果就是每个标签
页第一次加载的时候表现还算正常，但是加载完成后在不同标签页之间切换就表现得很卡，下一个标
签页是一个一个item的加载出来，倒是有种动画的效果（哈哈哈...）。 分析原因是因为`v-if`
设置为`false`会被remove掉，当设置为`true`时又重新构建，所以切换时上一个页面已经remove，
但是下一个页面正在构建，所以就出现了这种`动画`效果。

除了`v-if`之外，`vue.js`还提供了`v-show`还控制标签是否显示，很可惜，weex并没有提供
`v-show`的实现，所以我就想能不能自己覆盖weex的`div`组件，让其在Android平台上能够实现
`View.VISIBLE`、`View.INVISIBLE`和`View.GONE`之间的切换。

说干就干，实现起来其实很简单，我之前已经覆盖了`div`组件，所以这次只要增加一个属性就可以了，
在组件中，我新增了一个属性，当属性值为`false`时通过组件的`getRealView()`获取`View`并
设置`visible`为`View.GONE`，反之设置为`View.VISIBLE`，这样就实现了只隐藏`View`
但是不会移除，而且不占用视图布局空间。

结果总是出人意料，除了第一个list显示了之外，切换到其他list都不显示，调试组件java端的实现，
设置visible属性确实生效了，但是为什么不显示呢？遇到这种问题首先就是检查`View`的宽高是
否正常，是否都大于0，经过检查，一切正常。既然`View`的宽高正常，visible属性也设置为了
`View.VISIBLE`，那可能是`View`不在正确的位置上，但是不可能啊，没人移动它的位置啊，它
处于正常的标签位置处，也没有设置`position`属性。怀着疑惑以及十分懒散的心情，去查看了
`View`的位置，一看吓我一跳，不显示的list的`top`居然是3000多，这早超出了我的屏幕高度了，
难怪我看不到。是什么导致了`top`值这么大？肯定有原因的。很容易就发现是因为`marginTop`
值非常大导致的，但是问题又来了，我没有设置它的`marginTop`属性啊。随后我在weex代码里面
设置了它的`marginTop`为20px，再去java代码检查一下`marginTop`属性，发现这个属性值
稍微变大了一些，刚好增加了20px转换成Android px的值，这是为何？

`marginTop`这个属性到底是谁设置的？在哪里设置的？最有可能就是它的`parent`设置的。weex
的`div`标签能够竖向布局、横向布局的原理是什么？它本身是一个`FrameLayout`，不像Android
里的`LinearLayout`自带竖向和横向布局能力，那么它肯定是使用了其他手段实现的，经过查看
源代码以及之前的调试结果，判断出它就是通过增加子`View`的`marginTop`来实现竖向布局，通过
增加子`View`的`marginLeft`来实现横向布局。总算弄清楚所有问题的起因了，就是第一个list
占据了巨大空间，导致第二个list计算时`marginTop`特别大，从而挤出了屏幕。虽然知道了原因，
但是想要自己实现visible控制并不容易，因为weex在计算`marginTop`时把`View.GONE`的`View`
也计算在内了。希望weex官方早日提供`v-show`实现

### 神奇的`position`属性
`position`是一个枚举属性，取值为`static` `absolute` `fixed` `relative`,在`html`
里`position`的默认值是`static`，在`weex`里它的默认值是`relative`。

虽然weex提供了`position`，但是在某些情况下表现的十分诡异，会出现一些具有默认`position`
属性的元素位置出现异常。一般来说，我不会使用这个属性，正常的布局能满足要求，之所以使用它
是因为我上文提到的需要实现“搜索列表”展示，我想通过设置`position`属性为`absolute`让list
脱离正常的标签流，实现三个list在同一个位置叠加显示。

我们的weex页面根标签具有`absolute`的`position`属性，最终的页面结构：
```html
<div style="position: absolute">
    <div>搜索框</div>
    <div>3个搜索结果标签选择按钮</div>

    <div>
        <list style="position: absolute">搜索结果1</list>
        <list style="position: absolute">搜索结果2</list>
        <list style="position: absolute">搜索结果3</list>
    </div>

    <!-- 之前就有的 -->
    <div style="position: absolute">搜索历史</div>
    <div style="position: absolute">搜索提示，热门搜索，标签等</div>

</div>
```
最初这样的写的时候，无论我如何设置`top`，`left`，`margin`这些属性，所有的列表就是一
个都不显示，如果把`absolute`修改成`fixed`就显示了，至今不知道是什么原因导致的，后来
我观察到“搜素历史”这个`div`，同样的是`absolute`为什么它就可以显示，于是我找出了他们
唯一不同点就是list外面还有个`div`包裹，我把外层的`div`去掉之后就能正常显示了，这不科学，
weex里的`position`属性实现有bug。

正当我被自己的机智折服时，突然发现“3个搜索结果标签选择按钮”这个`div`看不见了，但是还是
占着它原本的位置的，嗯，一定是幻觉，再试试。哈哈，我就说是幻觉，这不出现了吗？不对，它
怎么跑到list底下去了，也就是页面的最底部，这不符合布局结构啊。后来分析，因为这个`div`
设置了v-if，会根据条件切换是否显示状态，从而将它从父组件中去除了，当再次出现时因为list
这些标签是`absolute`的，所以weex将其重新View tree时将这个div放置的`children position`
出错了，weex又给我留坑。
