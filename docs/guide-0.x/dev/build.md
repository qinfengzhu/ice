---
title: 工程配置
order: 2
---

前端项目普遍都需要使用 webpack 进行构建，但是 webpack 的配置相当繁琐，因此我们封装了 ice-scripts 这个工具简化工程的使用链路。

## 开发调试

安装依赖：

```bash
$ npm i --save-dev ice-scripts
```

在 package.json 中配置 `npm scripts`：

```json
{
  "scripts": {
    "start": "ice-scripts dev",
    "build": "ice-scripts build"
  }
}
```

然后执行 `npm start` 即可进行项目开发，正常情况下执行命令后自动打开浏览器 `http://localhost:4444` 进行页面预览。修改源码内容后将自动刷新页面。执行 `npm run build` 进行项目构建，构建产物默认输出到 `./build` 目录下。

## 命令行介绍

### dev

```bash
$ ice-scripts dev --help

Usage: ice-scripts dev [options]

Options:
  -p, --port <port>      服务端口号
  -h, --host <host>      服务主机名
  --https                开启 https
  --analyzer             开启构建分析
  --analyzer-port        设置分析端口号
  --disabled-reload      关闭 hot reload
  --disabled-mock      关闭 mock 功能
```

比如使用 3000 端口启动 dev server：

```bash
$ npm run start -- -p 3000
# 或者
$ ice-scripts dev -p 3000
```

比如开启 https

```bash
$ npm run start -- --https
```

### build

构建项目代码：

```bash
$ npm run build
```

构建服务支持的命令参数：

```bash
$ ice-scripts build --help

Usage: ice-scripts build [options]

Options:
  --analyzer             开启构建分析
  --analyzer-port        设置分析端口号
```

构建产物默认生成到 `./build` 目录下。

## 工程配置

`ice-scripts` 通过 `ice.config.js` 文件支持项目配置，支持基础配置、插件配置以及自定义 webpack 配置三种模式。

## 基础配置

基础配置涵盖了常见 webpack 定制场景，方便开发者快速设置项目工程配置。

### entry

* 类型：`string`
* 默认值：`src/index.js`

默认会以 `src/index.js` 文件作为入口文件，如果你需要改变默认的入口文件，可以自行修改 `ice.config.js` 即可生效。

```js
// ice.config.js
module.exports = {
  entry: 'src/index.js'
}
```

如果你的项目是多页应用，希望把 `src/pages` 的文件作为入口，那么可以这样配置：

```js
// ice.config.js
module.exports = {
  entry: {
    dashboard: 'src/pages/dashboard/index.js',
    about: 'src/pages/about/index.js',
  }
}
```

多 entry 的情况构建时会额外生成 vendor.js/css，需要自行在 html 里引入（public 目录会自动引入），也可以通过设置下面的 `vendor` 禁止生成 vendor 文件。

### alias

* 类型：`object`
* 默认值：`{}`

创建 `import` 或 `require` 的别名，使模块应用变得更加简单。
配置 webpack 的 [resolve.alias](https://webpack.js.org/configuration/resolve/#resolve-alias) 属性。

```js
// ice.config.js
const path = require('path');

module.exports = {
  alias: {
    '@components': path.resolve(__dirname, 'src/components/')
  }
}
```

现在，替换「在导入时使用相对路径」这种方式，就像这样：

```diff
-import CustomTips from '../../../components/CustomTips';
+import CustomTips from '@components/CustomTips';
```

使用了 alias 能力之后会发现 IDE 无法识别&跳转 import 的文件路径了，此时推荐在项目里创建 `jsconfig.json` 文件，并配置以下内容：

```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "jsx": "react",
    "paths": {
      "@/*": ["./src/*"]
    }
  }
}
```

通过上述的配置来指定 `@/` 引用相对于 `baseUrl` 的路径映射，可以快速定位引用模块，从而提升项目开发的体验。

### define

* 类型：`object`
* 默认值：`{}`

创建一个在编译是可以配置的全局变量，针对开发环境和发布环境的构建配置不同的行为非常有用。

用法：

```js
module.exports = {
  define: {
    // 此处不能省略 JSON.stringify，否则构建过程会出现语法问题
    ASSETS_VERSION: JSON.stringify('0.0.1'),
  }
}
```

在代码里使用该变量（当做全局变量使用）：

```javascript
console.log(ASSETS_VERSION);
```

### publicPath

* 类型：`string`
* 默认值：`/`

配置 webpack 的 [output.publicPath](https://webpack.js.org/configuration/output/#output-publicpath) 属性。
仅在运行 `ice-scripts build` 时生效。

```js
// ice.config.js
module.exports = {
  publicPath: 'https://cdn.example.com/assets/'
}
```

### devPublicPath

* 类型：`string`
* 默认值：`/`

同 `publicPath` 仅在运行 `ice-scripts dev` 时生效。

```js
// ice.config.js
module.exports = {
  devPublicPath: 'http://127.0.0.1/'
}
```

### hash

* 类型：`boolean` | `string`
* 默认值：`false`

如果希望构建后的资源带 hash 版本，可以将 `hash` 设置为 `true`

```js
// ice.config.js
module.exports = {
  hash: true,
  // hash: 'contenthash'
}
```

### outputDir

* 类型：`string`
* 默认值：`build`

修改构建后的文件目录

```js
// ice.config.js
module.exports = {
  // 构建后输出到dist目录
  outputDir: 'dist'
}
```

### vendor

* 类型：`boolean`
* 默认值：`true`

配置是否生成 vendor，如果希望禁用：

```js
// ice.config.js
module.exports = {
  vendor: false
}
```

### externals

* 类型：`object`
* 默认值：`{}`

将某些 `import` 的包排除在 bundle 之外，在运行时再去外部获取这些依赖。
比如，从 CDN 引入 React 资源，而不是将它打包

详细配置同 webpack 的 [externals](https://webpack.js.org/configuration/externals/#externals)

例如通过配置 `externals` 减少图表资源大小：

在使用到图表（Bizcharts）的时候，会发现打包后的文件特别大。是由于图表库本身比较大，这样会影响页面的加载效率。可以通过 CDN 的方式加载图表库，在打包时排除掉对应的图标库。

配置 `ice.config.js` 的 `externals`

```js
module.exports = {
  externals: {
    'bizcharts': 'BizCharts',
  }
}
```

说明：key 表示依赖包名，如： `bizcharts`。 value 表示引用 CDN 后的全局变量名，如: `BizCharts`

> 参考：[https://github.com/alibaba/BizCharts](https://github.com/alibaba/BizCharts)

将 CDN 文件添加到

```html
<!DOCTYPE html>
<html>

<head>
  <meta charset="utf-8" />
  <meta http-equiv="x-ua-compatible" content="ie=edge,chrome=1">
  <meta name="viewport" content="width=device-width">
  <title>ICE Design Lite</title>
</head>

<body>
  <div id="ice-container"></div>
+  <script src="https://cdn.jsdelivr.net/npm/bizcharts/umd/BizCharts.min.js"></script>
</body>

</html>
```

### devServer

* 类型：`object`

配置 webpack 的 [devServer](https://webpack.js.org/configuration/dev-server/#devserver) 属性。

不如通过配置 `devServer` 使项目开发使用 BrowserRouter

```js
// ice.config.js
module.exports = {
  // 修改 devServer 配置
  devServer: {
    historyApiFallback: true,
  }
}
```

将 `HashRouter` 修改为 `BrowserRouter`

```diff
-import { HashRouter as Router } from 'react-router-dom';
+import { BrowserRouter as Router } from 'react-router-dom';
```

完成上述修改后，本地开发就可以使用 BrowserRouter 了。

### proxy

* 类型：`object`
* 默认值：`{}`

配置 webpack 的 [devServer.proxy](https://webpack.js.org/configuration/dev-server/#devserverproxy) 属性。

> 建议使用 `proxy` 来设置代理而不要修改 webpack 的 `devServer.proxy`

```js
// ice.config.js
module.exports = {
  proxy: {
    '/**': {
      // 通过 enable 字段快速开关代理配置
      enable: true,
      target: 'http://127.0.0.1:6001'
    }
  }
}
```

更多接口代理，详见[接口代理](/docs/guide-0.x/dev/proxy.md)

### injectBabel

* 类型：`string`
* 默认值：`polyfill`

默认情况下会注入 core-js/stable 和 regenerator-runtime/runtime，根据 `targets` 配置的兼容浏览器进行 polyfill，实现按需添加。
开发类库项目，可以将配置设置为 `runtime`。
如果想手动 polyfill，可以将配置设置为 `false`，工程将不会进行自动的 polyfill。

```js
// ice.config.js
module.exports = {
  injectBabel: 'runtime'
}
```

### minify

* 类型：`boolean`
* 默认值：`true`

构建后的资源将进行压缩，如果不希望资源被压缩可以修改为 `false`

```js
// ice.config.js
module.exports = {
  minify: false
}
```

### outputAssetsPath

* 类型：`object`
* 默认值：`{ js: 'js', css: 'css' }`

修改构建后的 css/js 文件目录，默认情况下 css 在 `build/css/` 下，js 在 `build/js/` 下，可以通过 `outputAssetsPath` 配置修改：

```js
// ice.config.js
module.exports = {
  outputAssetsPath: {
    // 示例1：修改为 build/css-dist/ build/js-dist/
    js: 'js-dist',
    css: 'css-dist',
  }
}
```

### targets

* 类型：`object`
* 默认值：`last 2 versions, Firefox ESR, > 1%, ie >= 9, iOS >= 8, Android >= 4`

配置 @babel/preset-env 的 [targets](https://babeljs.io/docs/en/babel-preset-env#targets)，配置浏览器最低版本，新配置的 `targets` 会覆盖默认值。

```js
// ice.config.js
module.exports = {
  targets: {
    chrome: 49,
    ie: 11,
  }
}
```

## 插件配置

插件用于封装一些常见场景，通过 `plugins` 字段来配置插件列表：

* 类型：`Array`
* 默认值：`[]`

插件数组项每一项代表一个插件，ice-scripts 将按顺序执行插件列表，插件配置形式如下：

```js
// ice.config.js

module.exports = {
  plugins: [
    // npm依赖包
    ['ice-plugin-fusion', {
      themePackages: []
    }]
    // 相对路径
    './plugins/custom-plugin',
  ]
}
```

ice-scripts 有丰富的插件生态，业务可以按需选择，[插件列表](/docs/guide-0.x/build/plugin-list)。

## 自定义 webpack 配置

ice-scripts 内部的基础 webpack 配置都是通过 [webpack-chain](https://github.com/neutrinojs/webpack-chain) 生成的，它通过 webpack 配置链式操作的 API，并可以定义具体 loader 规则和 webpack 插件的名称，可以让开发者更加细粒度修改 webpack 配置。

`ice.config.js` 中提供的 `chainWebpack` 提供自定义 webpack 修改的接口。`chainWebpack` 接收两个参数：

- `config`: 链式的 webpack 配置，基于 `webpack-chain` 的 API 进行修改
- `options`: `{ command }` 命令执行参数，`command` 为当前执行命令 `dev | build`

```js
// ice.config.js
module.exports = {
  chainWebpack: (config) => {
    // 修改 webpack devServer.hot
    config.devServer.hot('dist');

    // 修改 webpack output.path
    config.output.path('dist');
  }
}
```

更复杂的场景请参考 [修改 webpack 配置](/docs/guide-0.x/build/plugin-dev#修改 webpack 配置)。