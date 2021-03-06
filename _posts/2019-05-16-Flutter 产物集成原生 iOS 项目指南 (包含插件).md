# Flutter 产物集成原生 iOS 项目指南 (包含插件)

随着 Flutter 1.5 版本的发布, 越来越多的原生项目开发者开始学习 Flutter, 通过在项目中引入 Flutter  混合开发的方式尝试 Flutter 是一个不错的选择

### 混合方案

目前混合开发方案有两种集成方式:

##### 源码集成: 

谷歌官方提供的方案  https://github.com/flutter/flutter/wiki/Add-Flutter-to-existing-apps,  

开发调试方便, 但会产生编译依赖, `所有项目开发者都需要安装 flutter 环境` ,` 每次编译都会重新生成 flutter 产物, 编译速度会有一定影响 `

##### 产物集成:

不影响原生项目, 把编译生成的 flutter 产物集成到项目中, Flutter 所有相关内容就只是一个 pod , 添加或移除方便

鉴于源码集成方式依赖性强, 使用产物集成方式是原生项目尝试接入最好的选择, 对原生项目没有任何侵害, 我这边目前的做法是 flutter 开发者通过源码集成方式开一个分支开发相关功能, 而其他成员通过产物集成方式拉取 Pod 即可集成 flutter

这里主要讲讲产物集成的过程包含插件的集成

### 创建 Flutter 项目

创建 Flutter项目和源码集成,谷歌的教程是相同的

```
$ flutter create -t modult my_flutter
```

源码集成和产物集成主要差别是 flutter 生成的产物, 源码集成通过 pod 和 编译脚本在每次编译时生成 flutter 产物自动添加到项目中, 而产物集成是把生成的产物拿出来直接使用

### 生成 Flutter 产物

在 flutter 项目路径,终端输入 flutter build ios 即可生成产物

```
$ my_flutter: flutter build ios --debug
$ my_flutter: flutter build ios --release
```

iOS 项目路径(隐藏文件):

```
/my_flutter/.ios
```

如果执行 flutter build ios 失败, 一般是证书问题, 可打开 `.ios/Runner.xcworkspace` 修复问题, 确保 Runner 可编译通过

执行成功后可看到在 `.ios/Flutter` 文件中会生成 App.framework 和 Flutter.framework, 这就是生成的 Flutter 产物了

我们可以直接把这两个 framework 拖进原生项目中使用, 无任何侵害的使用 flutter, 但是制作成 pod  是最好的选择, 打包和团队开发都方便, flutter 开发者通过 my_flutter 开发完成后, 执行 flutter build ios 生成产物, 发布到 pod , 其它成员拉取 pod 即可更新

```
MyFlutterSDK
    /MyFlutterSDK
        /App.framework
        /Flutter.framework
    MyFlutterSDK.podspec
```

### 集成插件

产物集成的方式集成插件的原生代码是比较麻烦的事情, 源码集成通过脚本会自动添加插件生成 pod, 而产物集成我们需要手动集成插件, 插件的文件目录

```
/my_flutteer/.ios/Flutter/.symlinks/{插件名称}/ios
```

插件文件是一个路径映射, 它真正所在的路径在 flutter 库中

```
$ flutter/.pub-cache/hosted/pub.flutter-io.cn
```

在 `插件文件/ios` 中一般会看到 `Classes` 和 `xxx.podspec` 两个文件, 我们只需要把它们拿出来单独制作成 pod 即可, 需要注意的是在插件 podspec 中一般会依赖 Flutter

```
s.dependency 'Flutter'
```

这里需要把它改成我们自己生成的 pod 名称, 比如上面生成的 MyFlutterSDK

由于包含插件就涉及到插件注册我们还需要把插件注册文件制作成 pod , 路径在

```
/my_flutteer/.ios/Flutter/FlutterPluginRegistrant
```

记得更改里面引用的 Flutter 名称

### 完成制作

至此我们需要的相关文件都制作完成了,包含的 pod 有:

```
MyFlutterSDK
FlutterPluginRegistrant
插件
插件
...
```

在原生项目中集成 pod 即可, 需要注意的是 flutter build ios —release 生成的 framework 需要使用真机测试, 因为 debug 和 release  flutter 的编译方式是不同的, 我们可以制作两个 pod 针对不同的环境, 比如:

```
MyFlutterSDK_Debug
MyFlutterSDK_Release
```

最后放上 Demo, 里面包含了一个插件, 用来演示插件的集成

地址: <https://github.com/ZeroJian/FlutterHybrid>
