# webpack4 的生产环境优化

本文主要介绍了 webpack4 生产环境我都做了哪些优化

文章的结构如下：

-   1.静态资源路径。
-   2.hash 缓存。
-   3.代码分割。
-   4.压缩混淆代码。
-   5.开启 gzip 压缩。
-   6.关闭 devtool。
-   7.Tree Shaking

## 1.静态资源路径。对静态资源路径的处理和验证

静态资源包括: js css image，

静态资源的路径的组成: 前缀 + 自己设置的路径

**前缀 ：静态资源的路径配置的前缀通过 output.publicPath 设置**

关于publicPath的解释 https://juejin.im/post/5ae9ae5e518825672f19b094

**自己设置的路径**

> js 的路径配置

js 的路径配置在 output.filename

```js
output: {
    path: path.resolve(__dirname, '../dist'),
    filename: 'static/js/[name].[contenthash:8].js'
}
```

> css 路径配置

css 的路径配置在分离 css 的插件里。例如之前的 extract-text-webpack-plugin 或者 mini-css-extract-plugin

```js
plugins: [
    new MiniCssExtractPlugin({
        filename: "static/css/[name].[contenthash:8].css"
    })
];
```

> image 路径配置

img 的路径配置在 url-loader 里配置

```js
modules: {
    rules: [
        {
            test: /\.(png|jpe?g|gif|svg)(\?.*)?$/,
            use: [
                {
                    loader: "url-loader",
                    options: {
                        limit: 10000,
                        name: "static/images/[name].[hash:7].[ext]"
                    }
                }
            ]
        }
    ];
}
```

**ps: 有的前端同学，可能像我一样不太清楚如何验证打包压缩后的的文件内路径配置的是否正确**

推荐一个简单的方法。下载 express 项目脚手架生成一个 node 项目，将打包后的 dist 扔到 public，启动 node 服务器

```bash
npm install express-generator -D
```

当然也可以直接去打开 dist 文件夹里的路径，去确实路径是否正确。

## 2.hash 缓存。

将业务代码、第三方类库、runtime 代码、css 单独打包，给他们不同 hash，来最大化利用缓存

webpack3 中分离业务代码、第三方类库需要用 CommonChunksPlugin。
webpack4 的新增 optimization,可以方便的分离代码，而且 hash 的稳定性的问题也有改进。

**单独打包业务代码、第三方类库、runtime**

```js
    optimization: {
        splitChunks: {      // 打包 node_modules里的代码
            chunks: 'all'
        },
        runtimeChunk: true,  // 打包 runtime 代码
    }
```

**单独打包 css 代码**

webpack4 推荐 mini-css-extract-plugin

> 注意: 之前的 extract-text-webpack-plugin 需要 beta 版本才支持，而且 contenthash 无法使用。

**分配不同的 hash**

关于 hash 稳定性的坑

注意区分

-   [hash] ： 整个项目有变动时，hash 变化。
-   [chunkhash] ： chunk 有变动，chunkhash 变化
-   [contenthash] ： 目前文档没有明确定义和说明，但是和文件内容的变化相关

在分离 js 和 css 时，都用设置 contenthash.

```js
  output: {
    path: path.resolve(__dirname, '../dist'),
    filename: 'static/js/[name].[contenthash:8].js',
    publicPath: '/'
  },
```

```js
    new MiniCssExtractPlugin({
      filename: 'static/css/[name].[contenthash:8].css'
    }),
```

```
配置js的文件名时，之前webpack3都是用chunkhash也没问题，但是实践后发现webpack4中用chunkhash，会导致，修改css时引发js的chunkhash变化，从而缓存失效。
```

经测试这样的设置，的确可以分离打包，并且各自的 hash 值互相不会干扰，如果有问题的话，可以共同讨论

## 3.代码分割

代码分割或者说懒加载，是 webpack 从诞生就一直标榜的功能吧。

它的作用就是把 js 分割成几份，在用户需要加载时才加载，这样不用一次性加载所有 js。

那么在 webpack 里实现代码分割并不是用配置的方式，而是通过我们写代码的方式来告诉 webpack 哪些代码要分割

webpack 里有 2 种 webpack 分割方法

-   webpack 内置方法 : require.ensure() 和 require.include()

-   es2015 定义的 动态 import,import 返回 promise

require.ensure 使用 demo

```js
// require.ensure()的 4 个参数 ： []依赖 、 callback 、 errorCallback 、 chunkName
require.ensure(
    ["./subPageA"],
    () => {
        var subPageA = require("./subPageA");
    },
    "subPageA"
);

// require.include('./moduleA.js')可以提取异步模块中的公共部分到父chunk
```

import 使用 demo

```js
import("./subPageA").then(subPageA => {
    console.log(subPageA);
});

// 区别
//  import('./subPageA')会直接执行这个文件，
//  而require.ensure()不会直接执行，是在里面的回调函数里才执行的引入操作。
```

> 关于import的使用注意

```
@babel升级后，使用import的语法，需要下载插件 @babel/plugin-syntax-dynamic-import

以下的地址链接
https://babeljs.io/docs/en/next/babel-plugin-syntax-dynamic-import.html
```


## 4.压缩混淆代码

听说webpack4只需要设置mode:produciton，就自动打包混淆js代码啦！

很好，现在我们只需要压缩css了可以了呢，于是下载插件optimize-css-assets-webpack-plugin

```js
const OptimizeCssAssetsPlugin = require('optimize-css-assets-webpack-plugin')
```
```js
  optimization: {
    splitChunks: {
      chunks: 'all'
    },
    runtimeChunk: true,
    minimizer: [
      new OptimizeCssAssetsPlugin({})
    ]
  },

```

```js
    new OptimizeCssAssetsPlugin({
      assetNameRegExp: /\.optimize\.css$/g,
      cssProcessor: require('cssnano'),
      cssProcessorOptions: { safe: true, discardComments: { removeAll: true } },
      canPrint: true
    }),
```

再看一眼我们的代码，坑爹的发现js的压缩居然失效了。

为什么呢？我在问答社区看到类似的问题，并且在官方文档找到了解释。

大致意思就是。默认optimization.minimize是true，所以js可以自动帮你压缩

但是自定义minimizer后，webpack默认配置会取消掉。

文档还很皮的告诉你，如果你用了css压缩，记得自己用uglifyjs压缩js呀。。。


- https://segmentfault.com/a/1190000014904384
- https://webpack.js.org/plugins/mini-css-extract-plugin/

总之，还是要自己用uglifyjs配置后压缩js

```js
    minimizer: [
      new OptimizeCssAssetsPlugin({}), // 压缩 css,使用minimizer会自动取消webpack的默认配置，所以记得用UglifyJsPlugin
      new UglifyJsPlugin({
        // 压缩 js
        uglifyOptions: {
          ecma: 6,
          cache: true,
          parallel: true
        }
      })
    ]

```

## 5.开启 gzip 压缩。

开启gzip压缩，那么压缩的好处是什么？

可以减小文件体积，传输速度更快。

-   服务端发送 response 时可以配置 Content-Encoding：gzip，用户说明数据的压缩方式

-   浏览器接受时，就可以根据相应个格式去解码。客户端请求时，可以用 Accept-Encoding:gzip，用户说明接受哪些压缩方法

所以 gzip 格式在 http 中传输文件的话，速度更快。那么谁来压缩文件？

不是服务端，就是客户端咯。

-   服务端比如 ngix 或者 node 去做压缩，
-   也可以 webpack 打包上线时，通过插件去做压缩。

服务端响应时压缩，肯定不如应用构建时压缩更合适。因为压缩也是要有时间开销的

```js
const CompressionWebpackPlugin = require('compression-webpack-plugin');

webpackConfig.plugins.push(
    new CompressionWebpackPlugin({
      asset: '[path].gz[query]',
      algorithm: 'gzip',
      test: new RegExp('\\.(js|css)$'),
      threshold: 10240,
      minRatio: 0.8
    })
)
```

压缩之后，文件体积减少的确显著。

## 6.关闭devtool

devtool我没有做很深入的研究。

我的理解，开发环境必须要配置，否则肯定无法调试。

而生产环境可以这一项，因为我们不需要在生产环境调试代码。如果理解有误，欢迎指正哈~

## 7.Tree shaking


tree shaking 的原理

-   ES6 的模块是静态分析的。所以在编译时可以判断哪些代码没有 exports
-   分析程序流，判断哪些变量没有被使用、从而删除代码



webpack4的新增了sideEffects来指定“有副作用的文件”，

但是我在实际使用时，遇到一些坑，比如这样配置后，我意思是指定.css文件不要被“摇”掉。

但这个配置依旧导致了css文件打包后被当做冗余代码被删除。


```js
  "sideEffects": [
    "*.css"
  ],
```

关于tree shaking就贴一篇文章。关于tree shaking研究清楚后再更新吧

https://zhuanlan.zhihu.com/p/32831172


以上就是整个配置的说明啦，目前tree shaking目前还没有搞定，其他功能我自己测试是没有问题的。

贴一份，webpack4完整的配置文件的地址。

如果觉得有帮助希望star一下，如果遇到问题，也欢迎指教！
 

