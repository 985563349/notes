### 实用的 Vue 项目打包优化方案

以下几种打包优化方案，都是实际项目中非常实用的，使用以下方案进行优化，通常都能使项目体积减小 30%左右。

#### Source Map

生产环境的代码通常都有被压缩混淆过，运行时若抛出异常，往往很难定位到具体位置，Source Map 的出现就是为了解决这个问题。

Source Map 本质上是一个信息文件，里面储存着代码转换前后的对应位置信息，通过这些信息就能准确定位到异常。

`生成Source Map会极大的降低打包速度。`

生产环境可以考虑取消 Source Map 生成。如果要生成，应将服务器配置为不允许普通用户访问 Source Map 文件！

```javascript
// vue.config.js
module.exports = {
  productionSourceMap: false, // 关闭生产环境source map生成
};
```

#### 组件懒加载

组件的代码默认会被分别打包至 app.js 和 app.css 文件中，组件越多，文件就会越大。

app.js 与 app.css 文件会在首屏完成加载，过大的文件会降低加载速度，从而导致白屏时间过长。设置懒加载可以将组件的代码抽离出去，生成独立的文件，当使用到该组件时，才会加载对应的文件。

实际项目中使用最多的是路由懒加载。

`懒加载需要抽离代码，生成文件，因此会降低打包速度。`

> webpackChunkName 可以指定生成文件的名称。

```javascript
const routes = [
    // ...route
    {
        path: '/about',
        name: 'About',
        component: () => import(/* webpackChunkName: 'about' */'./views/about.vue');
    }
];
```

#### 按需加载

实际项目中通常会依赖许多的第三方包，全局引入的方式会将包的所有代码都打包到项目中，而实际上绝大多数代码都是不需要的，按需加载可以在构建过程中将不需要的代码剔除掉。

按需加载能大幅减小文件体积，可以做为`主要的优化手段`。

##### 按需加载 ElementUI

首先，安装 babel-plugin-component：

```shell
npm install babel-plugin-component -D
```

然后，将 babel.config.js 修改为：

```javascript
module.exports = {
  presets: ['@vue/cli-plugin-babel/preset'],
  plugins: [
    // ...plugins
    [
      'component',
      {
        libraryName: 'element-ui',
        styleLibraryName: 'theme-chalk',
      },
    ],
  ],
};
```

接下来，创建一个`element-ui-plugin.js`文件，在这个文件中注册你需要使用的组件：

```javascript
import { Button, Message } from 'element-ui';

export default {
  install: (Vue) => {
    Vue.use(Button);
    Vue.prototype.$message = Message;
  },
};
```

最后，在`main.js`中引入`element-ui-plugin.js`：

```javascript
import Vue from 'vue';
import ElementUiPlugin from './element-ui-plugin';

Vue.use(ElementUiPlugin);
```

#### CDN 加速

项目中的第三方包，默认都会被打包到 chunk-vendors.js 中，因此该文件的体量往往会非常的大。

chunk-vendors.js 会在首屏幕完成加载，过大的文件会降低加载速度，导致白屏时间过长。通过配置 externals（外部扩展）可以将第三包的代码从 chunk-vendors.js 中剔除出去，运行时再从用户环境中获取这些扩展依赖。

外部扩展通常会指定为一个 CDN 链接，使用 html-webpack-plugin 可以很方便的将 CDN 链接注入到 html 文件中。

`使用CDN时，一个稳定可靠的CDN服务是非常非常重要的。`

```javascript
// vue.config.js

// 定义外部扩展
const externals = {
  vue: 'Vue',
};

// 定义CDN地址
const cdn = {
  css: ['https://unpkg.com/element-ui/lib/theme-chalk/index.css'],
  js: ['https://unpkg.com/vue@2.6.14/dist/vue.runtime.min.js'],
};

module.exports = {
  chainWebpack: (config) => {
    if (process.env.NODE_ENV === 'production') {
      config.externals(externals);
      // 使用html-webpack-plugin，输入CDN连接
      config.plugin('html').tap((args) => {
        args[0].cdn = cdn;
        return args;
      });
    }
  },
};
```

配置 index.html 文件：

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8" />
    <!-- 注入CSS -->
    <% for(let css of htmlWebpackPlugin.options.cdn.css) { %>
    <link rel="stylesheet" href="<%=css%>" />
    <% } %>

    <!-- 注入JS -->
    <% for(let js of htmlWebpackPlugin.options.cdn.js) { %>
    <script src="<%=js%>"></script>
    <% } %>
    <title><%= htmlWebpackPlugin.options.title %></title>
  </head>

  <body>
    <div id="app"></div>
  </body>
</html>
```

#### 拆分第三方模块

如果找不到稳定可靠的 CDN 服务，或者不想使用 CDN，同时又希望将第三方包从 chunk-vendors.js 中抽离出来，可以使用`splitChunks`进行拆分，将依赖生成独立的文件。

`注意：当文件过大时再考虑拆分，实际优化时需要在请求大小和请求数量之间做权衡。`

```javascript
// vue.config.js
module.exports = {
  // ...options

  chainWebpack: (config) => {
    // 生产环境配置
    if (process.env.NODE_ENV === 'production') {
      config.optimization.runtimeChunk('single').splitChunks({
        chunks: 'all', // 对同步、异步引入的模块都进行分析
        maxInitialRequests: Infinity, // 指定入口最大并行请求数，尽可能多的生成文件
        minSize: 20000, // 指定生成文件的最小体积
        cacheGroups: {
          vendor: {
            test: /[\\/]node_modules[\\/]/,
            name: (module) => {
              const packageName = module.context.match(/[\\/]node_modules[\\/](.*?)([\\/]|$)/)[1];
              // 指定第三方包输出文件名
              return `${packageName.replace('@', '')}`;
            },
          },
        },
      });
    }
  },
};
```

#### Gzip 压缩

Gzip 压缩，能极大的减轻文件的体积，加快传输效率，`可以作为主要的优化手段`。使用 Gzip 压缩需要对服务器进行配置，在不支持 Gzip 时，要将普通文件传输给客户端。

##### 打包生产 Gzip 文件

首先，安装 compression-webpack-plugin：

`注意：如果使用最新版的vue-cli，推荐安装compression-webpack-plugin 5.0.0版本，否则在打包过程中会出现异常。`

```javascript
npm install compression-webpack-plugin@5.0.0 --save-dev
```

接下来，修改 vue.config.js：

```javascript
// vue.config.js
const CompressionWebpackPlugin = require('compression-webpack-plugin');

module.exports = {
  // ...options
  chainWebpack: (config) => {
    if (process.env.NODE_ENV === 'production') {
      // 注册Plugin
      config.plugin('Compression').use(CompressionWebpackPlugin, [
        {
          include: 'src', // 指定src目录
          test: /\.js$|\.html$|\.css$/, // 匹配src目录下所有js、html、css文件
          minRatio: 1, // 只对压缩比率大于1的文件进行压缩，压缩比率 = 压缩后大小 / 原大小
          threshold: 10240, // 压缩大于1kb的文件
        },
      ]);
    }
  },
};
```
