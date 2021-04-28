### 认识 react-native-web

react-native-web 是由 Twitter 工程师(目前已加入 Facebook React.js 部门) Nicolas Gallagher 实现并维护的开源项目，目前已用在 Twitter、Flipkart、Uber 等项目中，它支持把 React Native 编写的项目转换成 Web 应用。

为 React Native 项目接入 react-native-web 成本极低：react-native-web 对原项目没有侵入性，无需改动原来的代码，只需在项目中加入一些 webpack 构建配置即可构建运行出和 React Native 应用一致效果的 Web 应用。

[react-native-web github](https://github.com/necolas/react-native-web)



### react-native-web 实现原理

react-native-web 实现了在不修改 React Native 代码的情况下渲染在浏览器里的功能，其实现原理如下：

在用 webpack 构建用于运行在浏览器里的代码时，会把 React Native 的导入路径替换为 react-native-web 的导入路径，在 react-native-web 内部则会以和 React Native 目录结构一致的方式实现了一致的 React Native 组件。在 react-native-web 组件的内部，则把 React Native 的 API 映射成了浏览器支持的 API。react-native-web 架构如图所示：

### ![截屏2020-11-22 上午12.32.07.png](https://cdn.nlark.com/yuque/0/2020/png/591836/1605976335659-59ae0a24-198e-439b-bcbb-2e2a821a8c9e.png)

我们来看看 react-native-web 的源码:

![截屏2020-11-22 上午11.51.48.png](https://cdn.nlark.com/yuque/0/2020/png/591836/1606017113311-c8db4859-6c05-40e9-ba0f-0287172db6f9.png)![截屏2020-11-22 下午12.01.34.png](https://cdn.nlark.com/yuque/0/2020/png/591836/1606017698004-2436d81c-4a6d-4f9f-83b0-4726e28927b0.png)

它的方案其实挺简单粗暴，把 RN 的组件和 API 都用 H5 实现适配一遍，适配其行为和默认样式，在打包的时候使用 webpack 的别名机制将用到的组件替换成 react-native-web 里的对应组件。

```
#项目中正常使用 'react-native' 组件
import { component } from 'react-native'

# 使用 webpack 打包时会自动替换为
import { component } from ''react-native-web'
```

### 项目创建

- **创建 ReactNative 项目**

项目以 RN 为主体,会涉及到原生自定义组件和通信,所以需要使用 React Natvie CLI 脚手架工具创建 ReactNative 项目

[项目创建-官方文档](https://reactnative.dev/docs/environment-setup)


> 注意：本次采用了将现有 ReactNative 应用程序转换为 web 的方法。另一种等效方法是启动一个新的 React 项目 npx create-react-app 并转换为 ReactNative 应用. 



- **添加 Web 支持**

通过添加 react-native-web, 和 expo 编译配置即可让 RN 项目支持 web

安装相关依赖

```
// 全局安装 expo 工具
npm i -g expo-cli

// 项目增加 expo
yarn add expo

// 项目增加 react-native-web 和 web 运行环境 react-dom
yarn add react-native-web react-dom
```

配置 expo

```
// app.json
{
    "name": "YourApp",
    "displayName": "React Native Project",
+    "expo": {
+        "platforms": ["web"]
+    }
}
  
// index.js  
import {
    AppRegistry,
+    Platform
  } from 'react-native';
import App from './App';
import {name as appName} from './app.json';

AppRegistry.registerComponent(appName, () => App);

+  if (Platform.OS === 'web') {
+      AppRegistry.runApplication(appName, {
+          rootTag: document.getElementById('root'),
+      });
+  }
```

通过 expo start 运行后, 输入 w 运行 web

[expo 文档](https://docs.expo.io/guides/running-in-the-browser/?redirected)

### expo

expo 工具内部实现了一套 web 运行配置, 包含  `index.html`, `serve.json`, `favicon.ico` , `webpack` 等配置.

项目根目录创建 `web/` 文件夹可自定义实现  `index.html`, `serve.json`, `favicon.ico`

如需要自定义 webpack 配置: 

- 安装 yarn add @expo/webpack-config
- 创建 webpack.config.js 文件到根目录

```
const createExpoWebpackConfigAsync = require('@expo/webpack-config');

// Expo CLI will await this method so you can optionally return a promise.
module.exports = async function(env, argv) {
  const config = await createExpoWebpackConfigAsync(env, argv);
  // If you want to add a new alias to the config.
  config.resolve.alias['moduleA'] = 'moduleB';

  // Maybe you want to turn off compression in dev mode.
  if (config.mode === 'development') {
    config.devServer.compress = false;
  }

  // Or prevent minimizing the bundle when you build.
  if (config.mode === 'production') {
    config.optimization.minimize = false;
  }

  // Finally return the new config for the CLI to use.
  return config;
};
```

几个 webpack 配置: 

```
 // 本地调试解决跨域问题  
 config.devServer = {
    ...config.devServer,
    port: 80,
    proxy: {
      '/api': {
        target: 'http://xxx.com',
        changeOrigin: true,
        secure: false, //不校验
      },
    },
  }

    // 部署到子路径配置
    if (config.mode === 'production') {
    config.output.publicPath = '/subPath/'
  }
```

通过 webpack 可自定义相关配置,并设置别名替换依赖包, 比如我们使用的 @ant-design/react-native 不支持 web, 可替换为 api 相同的 antd-mobile

```
 config.resolve.alias['@ant-design/react-native'] = 'antd-mobile';
```

通过打印 `createExpoWebpackConfigAsync` 创建的配置,可查看 expo 的默认 webpack 配置:

```
{
  mode: 'production',
  entry: { app: [ '{your project path}/index.web.js' ] },
  bail: true,
  devtool: 'source-map',
  context: '{your project path}/node_modules/@expo/webpack-config/webpack',
  output: {
    publicPath: '/',
    path: '{your project path}/web-build',
    globalObject: 'this',
    filename: 'static/js/[name].[contenthash:8].js',
    chunkFilename: 'static/js/[name].[contenthash:8].chunk.js',
    devtoolModuleFilenameTemplate: [Function (anonymous)]
  },
  plugins: [
    CleanWebpackPlugin {
      dangerouslyAllowCleanPatternsOutsideProject: false,
      dry: false,
      verbose: false,
      cleanStaleWebpackAssets: true,
      protectWebpackAssets: true,
      cleanAfterEveryBuildPatterns: [],
      cleanOnceBeforeBuildPatterns: [Array],
      currentAssets: [],
      initialClean: false,
      outputPath: '',
      apply: [Function: bound apply],
      handleInitial: [Function: bound handleInitial],
      handleDone: [Function: bound handleDone],
      removeFiles: [Function: bound removeFiles]
    },
    CopyPlugin { patterns: [Array], options: {} },
    HtmlWebpackPlugin {
      options: [Object],
      childCompilerHash: undefined,
      assetJson: undefined,
      hash: undefined,
      version: 4,
      platform: 'web'
    },
    InterpolateHtmlPlugin {
      htmlWebpackPlugin: [class HtmlWebpackPlugin extends HtmlWebpackPlugin],
      replacements: [Object]
    },
    ExpoPwaManifestWebpackPlugin {
      options: [Object],
      writeObject: [AsyncFunction (anonymous)],
      pwaOptions: [Object],
      rel: 'manifest'
    },
    FaviconWebpackPlugin {
      modifyOptions: {},
      pwaOptions: [Object],
      favicon: null
    },
    ApplePwaWebpackPlugin {
      modifyOptions: {},
      pwaOptions: [Object],
      meta: [Object],
      icon: null,
      startupImage: null
    },
    ChromeIconsWebpackPlugin { options: [Object], icon: null },
    ModuleNotFoundPlugin {
      appPath: '{your app path}',
      yarnLockFile: undefined,
      useYarnCommand: [Function: bound useYarnCommand],
      getRelativePath: [Function: bound getRelativePath],
      prettierError: [Function: bound prettierError]
    },
    DefinePlugin { definitions: [Object] },
    MiniCssExtractPlugin { options: [Object] },
    ManifestPlugin { opts: [Object] },
    WebpackBar {
      profile: false,
      handler: [Function (anonymous)],
      modulesCount: 500,
      showEntries: false,
      showModules: true,
      showActiveModules: true,
      options: [Object],
      reporters: [Array]
    }
  ],
  module: { strictExportPresence: false, rules: [ [Object], [Object] ] },
  resolveLoader: { plugins: [ [Object] ] },
  resolve: {
    extensions: [
      '.web.ts',  '.web.tsx',
      '.web.mjs', '.web.js',
      '.web.jsx', '.ts',
      '.tsx',     '.mjs',
      '.js',      '.jsx',
      '.json',    '.wasm'
    ],
    plugins: [ [Object] ],
    symlinks: false,
    alias: {
      'react-native$': 'react-native-web',
      'react-native/Libraries/Components/View/ViewStylePropTypes$': 'react-native-web/dist/exports/View/ViewStylePropTypes',
      'react-native/Libraries/EventEmitter/RCTDeviceEventEmitter$': 'react-native-web/dist/vendor/react-native/NativeEventEmitter/RCTDeviceEventEmitter',
      'react-native/Libraries/vendor/emitter/EventEmitter$': 'react-native-web/dist/vendor/react-native/emitter/EventEmitter',
      'react-native/Libraries/vendor/emitter/EventSubscriptionVendor$': 'react-native-web/dist/vendor/react-native/emitter/EventSubscriptionVendor',
      'react-native/Libraries/EventEmitter/NativeEventEmitter$': 'react-native-web/dist/vendor/react-native/NativeEventEmitter',
      'react-native/Libraries/Image/AssetSourceResolver$': 'expo-asset/build/AssetSourceResolver',
      'react-native/Libraries/Image/assetPathUtils$': 'expo-asset/build/Image/assetPathUtils',
      'react-native/Libraries/Image/resolveAssetSource$': 'expo-asset/build/resolveAssetSource'
    }
  },
  performance: { maxAssetSize: 600000, maxEntrypointSize: 600000 },
  optimization: {
    nodeEnv: false,
    minimize: true,
    minimizer: [ [TerserPlugin], [OptimizeCssAssetsWebpackPlugin] ],
    splitChunks: { chunks: 'all', name: false },
    runtimeChunk: true,
    noEmitOnErrors: true
  },
  node: {
    module: 'empty',
    dgram: 'empty',
    dns: 'mock',
    fs: 'empty',
    http2: 'empty',
    net: 'empty',
    tls: 'empty',
    child_process: 'empty'
  }
}
```

### 相关工具适配

- **React Navigation - 路由组件**

路由组件支持 web, 可直接添加运行

[React Navigation 官方文档](https://reactnavigation.org/docs/getting-started)

web 路由可通过配置 link 实现不同页面显示不同的 url

```
/// 设置深度链接, 设置过的屏幕可作为程序启动的根地址

// 如果返回一个空对象, 使用导航默认名称, 并且在地址栏输入任何子路径都会自动跳转到首页
const config = {}

// 如果设置了路径名称, 可在地址栏定位到此页面,未设置的页面会自动跳转到首页
const config = {
  // 此方法设置后, 所有屏幕都会成为深度地址
  // enabled: true,
  
  // 如果启动的根地址不是当前设置的根地址, 导航栏可显示一个后退按钮回退到 initialRoute
  initialRouteName: 'home',
  
  screens: {
    Home: 'home'
  },
}

// 多层嵌套的页面
// 页面对象有一个 path 和 screens 属性
const config = {
  screens: {
    Tabs: {
      path: 'Tabs',  // 设置过 path,页面路径 host/Tabs/..
      screens: {
        UserStack: {
          // path: 'user',   // 没有设置 path, 当前这一层不在地址显示 host/Tabs/..
          screens: {
            UserCenterScreen: 'center',  // 直接定义名称, 地址有显示 host/Tabs/center
          },
        },
      },
    },
  },
}

const linking = {
  prefixes: ['https://mychat.com', 'mychat://'], // 当前网站域名 和 app scheme, 针对 app 的 deeplink
  config,
}

<NavigationContainer linking={linking}>
```

**[ReactNative 配置 Link 文档](https://reactnavigation.org/docs/configuring-links)**

- **Async-Storage - 数据持久化**

已支持 web, web 端数据通过 Local Storage 存储

[Async-Storage 官方文档](https://react-native-async-storage.github.io/async-storage/docs/install/)



- **React Native UI Kitten - UI 组件库**

组件支持 web, 不过当前版本 expo webpack 运行会报错

web failed to compile

...

this.textInputRef.current?.focus();

...

解决方法:

1. 安装 yarn add @expo/webpack-config
2. 创建 webpack.config.js 文件到根目录,添加以下内容

```
const createExpoWebpackConfigAsync = require('@expo/webpack-config');

module.exports = async function(env, argv) {
    const config = await createExpoWebpackConfigAsync({
        ...env,
        babel: {
            dangerouslyAddModulePathsToTranspile: ['@ui-kitten/components']
        }
    }, argv);
    return config;
};
```

[相关 issue](https://github.com/akveo/react-native-ui-kitten/issues/996)



### 编写特定平台代码

为了让 ReactNative 三端同构能正常的运行,在有些情况下你不得不编写平台特定的代码,因为有些代码只能在特定平台下才能运行,编写特定的 web 平台代码有以下几种方式:

**文件命名方式:**

MyComponent.android.js 

MyComponent.ios.js 

MyComponent.web.js



如需要共享代码给 web 使用, 也可生成:

MyComponent.native.js 

MyComponent.js

或

MyComponent.js 

MyComponent.web.js



在使用的位置:

import MyComponent from './MyComponent'



一个语音播报的例子:

```
// SpeechTool.web.js
// web 端语音播报
export const speechSpeak = (text) => {
  var msg = new SpeechSynthesisUtterance(text)
  msg.volume = 100
  msg.rate = 1
  msg.pitch = 1.5
  window.speechSynthesis.speak(msg)
}

// SpeechTool.js
// 移动端语音播报, 通过原生 modules 实现, RN 调用
import {NativeModules} from 'react-native'
const SpeechToolModule = NativeModules.SpeechTool
export const speechSpeak = (text) => {
  SpeechToolModule.speak(text)
}

// 调用位置
import {speechSpeak} from './SpeechTool'
speechSpeak('你好')
```

**所有端的代码都在一个文件中,通过平台判断编写 web 专属代码:**

```
import { Platform } from 'react-native';

if(Platform.OS==='web'){
  // web 平台专属代码
}
```

**不同平台区分样式:**

```
import { Platform, StyleSheet } from 'react-native';

const styles = StyleSheet.create({
  container: {
    flex: 1,
    ...Platform.select({
      ios: {
        backgroundColor: 'red'
      },
      android: {
        backgroundColor: 'green'
      },
      default: {
        // other platforms, web for example
        backgroundColor: 'blue'
      }
    })
  }
});
```

[ReactNative 文档相关介绍](https://reactnative.dev/docs/platform-specific-code)

### 其它

- **react-native-web 暂不支持 RefreshControl 导致无法触发列表下拉事件问题**

```
// 相关 Issues: https://github.com/necolas/react-native-web/issues/1027
// 修复 web 暂不支持 RefreshControl 导致无法触发下拉事件问题
// https://github.com/NiciusB/react-native-web-refresh-control
import { patchFlatListProps } from 'react-native-web-refresh-control'
patchFlatListProps()
```

- **static-container 报错**:

https://stackoverflow.com/questions/59949008/im-trying-to-run-expo-project



- **[Text] fontSize not work when lower than 12 on web #1642  [safari 没此问题]**

https://github.com/necolas/react-native-web/issues/1642