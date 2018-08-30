#创建 Swift Cocoapods 和 Framework 相关问题

Cocoapods 创建私有库网上的文章已经很多了, 但是涉及到针对 Swift 遇到的问题相关文章很少,这里针对 Swift 创建包含 Oc 代码的私有库和 Framework 等问题做个梳理



创建流程不细说, 网上相关文章很多了

[方式一]

pod lib create xxxx

根据提示选择 Swift 语言, 选择包括 Example 项目

[方式二]

或创建一个 Framework 项目, 命令行输入

pod spec create xxxx



###创建 pod 遇到的一些问题

1. pod 内部需要引用 oc 的 .framework

在 swift 项目中, 引用 oc framework 只需添加一个 Bridging-Header.h 桥接文件即可引入, 而 swift Framework 中并不支持此种方式, 为解决这种问题, 可通过创建 .moudlemap 文件解决此问题,  moudlemap  文件就是对一个框架，一个库的所有头文件的结构化描述, 我们创建的 Framework 也会自动生成一个 moudlemap 文件

需要注意的是, module 名称需和 Framework 名称相同

```swift
module MyOcFramework {
    header "myOcFramework.framework/Headers/myOcFrameworkHeader.h"
	export *
}
```

```swift
import MyOcFramework

class Person {
  ......  
}
```



如果只是某个 .h 文件也是同理

```swift
module MyOcH {
    header "MyOcH.h"
	export *
}
```



2.找不到某个 moudlemap 创建的 framework name

提示找不到 framework 说明 moudlemap 文件路径错误, 使用[方式一]创建的 pod 库需要在 .podspec 里指定 .modulemap 文件路径

```ruby
s.preserve_paths = 'xxName/***/module.modulemap'
```



3.如果在 pod 库中依赖一些静态 pod 库,编译会失败, 这种情况只能使用 Framework 方式引入, 参照问题1



### pod 制作成 Framework

1.pod 可使用 package 插件把 pod 库制作成 Framework, 但是如果是 swift 会报错

```shell
 **/**.modulemap not found
```

目前还没有发现解决方法, github 上已提 issues: [https://github.com/CocoaPods/cocoapods-packager/issues/211] 



使用手动合并的方式目前没有此问题, 具体模拟器和真机合并流程网上文章很多, 在此不再赘述

2.制作的 Framework 启动后报错

swift 制作的 Framework 添加到 General - Embedded Binaries 



3.包含 oc Framework 并创建过 .moudlemap 的 Framewrok 编译报错找不到 .moudlemap 定义的框架名称

此问题是由于封装的 Framework 未包含 .moudlemap, 因此在框架内部找不到定义的名称, 如果只是封装成 pod 引入项目并没有此问题, 而封装成 Framework 需定义一份 .moudlemap, 并在 `Build Settings` - `Swift Compiler Search Paths` -`import Paths` 中指定  .moudlemap 文件路径



