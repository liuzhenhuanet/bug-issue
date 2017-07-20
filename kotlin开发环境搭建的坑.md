## Android kotlin开发环境搭建遇到的坑

### 与注解处理器冲突
原来的工程中用到了注解处理器，引入kotlin后注解处理器不生成代码了，花了好长时间，终于找到了是由于kotlin引起的，然后就改变
搜索策略，在Google上搜索“kotlin plugin与注解处理器冲突”，有这么一条记录[使用kapt - Kotlin 语言中文站](https://www.kotlincn.net/docs/reference/kapt.html)
出现在Google搜索的第二个位置上，点开之后，终于找到了解决办法：在gradle文件上引入`apply plugin: 'kotlin-kapt'`，
然后将gradle dependencies中的`apt`或者`annotationProcessor`使用`kapt`替换，然后重新rebuild工程。