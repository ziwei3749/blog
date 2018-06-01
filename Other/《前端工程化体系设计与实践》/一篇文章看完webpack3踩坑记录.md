# 一篇文章看完 webpack@3.6.0 的踩坑记录

## webpack@3.6.0 踩坑记录

> 1.webpack 安装

webpack 安装分为全局安装和本地项目安装

**_官方推荐: 使用本地项目安装，因为如果你用全局安装的话，等于锁定了版本，但是不同项目，可以用不同版本的 webpack 的打包_**

举一全局安装webpack后产生问题的场景:
```
你全局安装了webpack4,然后去从项目里git clone ,npm install会下载依赖，但是如果你电脑里安装了webpack他会从你的电脑里拿这个全局的webpack,

但是项目需要的是package.json里那个版本的webpack,这种情况就可能有问题
```

在终端直接输入 `webpack` 打包 和在 npm script 里设置用 npm run build 的区别就是

前者是使用全局的 webpack 打包，后者是用本地项目的 webpack 打包

```
  "scripts": {
    "server": "webpack-dev-server --open",
    "build" : "webpack"
  },
```

> 2.最简易的打包方式和最常见的打包方式

*   手动创建 dist 目录，下面有 index.html（index.html 里引入 bundle.js）
*   手动创建 src 目录，下面有 entry.js
*   终端里执行 webpack [入口 path][出口path]

也就是`webpack src/entry.js dist/bundle.js`

意思就是把 entry.js 这个文件打包，打包到 dist 下面的 bundle.js 里

以上是最简易的打包例子，实际上我们会在 webpack.config.js 里配置入口、出口，因为我们的需求是十分复杂的

> 3.我们可以配置多入口、出口，那么什么情况下需要配置多入口和多出口？

首先，项目打包策略遵循以下几点原则：

*   选择合适的打包粒度，生成的单文件大小不要超过 500KB
*   充分利用浏览器的并发请求，同时保证并发数不超过 6
*   尽可能让浏览器命中 304，频繁改动的业务代码不要与公共代码打包
*   避免加载太多用不到的代码，层级较深的页面进行异步加载

基于以上原则，我选择的打包策略如下：

*   第三方库如 vue、jquery、bootstrap 打包为一个文件
*   公共组件如弹窗、菜单等打包为一个文件
*   工具类、项目通用基类打包为一个文件
*   各个功能模块打包出自己的入口文件
*   各功能模块作用一个 SPA，子页面进行异步加载

> 4.利用 webpack 的 devServer，使用热更新服务

```
devServer: {
    contentBase: path.resolve(__dirname,'dist')   // 用绝对路径指定一个路径，你要对哪个页面开启热更新服务
    host: 'localhost' || '10.0.1.29'   // 设置服务器地址，就是我们的ip地址
    compress: true, // 配置服务器压缩
    port: 1717    // 端口号
}
```

这里需要注意的是使用 devServer 还需要下载 webpack-dev-server

```
npm i webpack-dev-server  --save-dev
```

安装后，因为是用的本地安装，所以需要在 npm script 里配置命令，这样你输入 npm run server 是就自动去 node_module/.bin/webpack-dev-server 找

```
  "scripts": {
    "server": "webpack-dev-server --open",
    "build" : "webpack"
  },
```

这里我安装了 webpack-dev-server@2.8.2,最新版本需要安装 webpack-cli

```
webpack-dev-server如果是3.x的话，webpack必须是4.x才不会报此TypeError: Cannot read property 'compile' of undefined错误, 同理如果webpack是3.x，则webpack-dev-server必须是2.x
```

> 5.打包 css 时，就是把 css 文件打包到 js 里，这样可以减少 http 请求，需要用 style-loader 和 css-loader 组合使用

*   css-loader 的作用就是解析.css 文件，遇到@import 和 url 就实现类似 require()的效果
*   style-loader 的作用就是将解析后的 css 文件，按照<style></style>的形式插入到 html 中

2 个 loader 配置时有顺序要求，style-loader 在前，css-loader 在后，因为 webpack 执行 loader 的顺序是从右往左的

> 6.uglify-webpack-plugin 是 webpack 自带的插件，但是你也单独下载其他的版本。

*   uglify-webpack-plugin: 压缩 js 代码的体积，大约是原来的 1/3

当你使用这个插件，然后运行 npm run dev 时会报错，因为开发环境不应该压缩 JS 代码。正式开发时肯定会环境

> 7.html-webpack-plugin

*   html-webpack-plugin:打包 html，就是把 src 目录下的 index.html 这个模板打包到 dist/index.html

可以用 template 配置以哪个 html 为模板,用 filename 配置生成的文件名

> 8.【图片处理】css 中的图片

你在.css 里通过 url 引入一个图片，那么打包出来地址肯定是错误的，启动服务器也会提示你，需要引入适当 loader 来处理图片

其实就是你在 src 目录下写代码的路径，打包到 dist 下了，而且没有任何处理的话，肯定路径的错的。

*   file-loader: 这个 Loader 就可以专门处理这个打包后 url 路径的处理
*   url-loader: 内置 file-loader，多一个配置 limit，可以让你图片小就转化 base64，大的话就 copy 一份
    *   limit: 小于它的转化为 base64，大于它的 copy 一份到 dist 目录下
    *   name: 设置输出的文件名，默认的是文件 + hash
    *   outputPath：打包出来的文件的路径结构
    *   publicPath: 表示打包文件中 link 引用文件的路径前缀，比如你的图片在 CDN 上，上线时加上这个参数，值为 CDN 地址。

> 9.【图片处理】css 除了可以打包到 js 里，还可以分离出来。而分离出来的 css 里的图片路径也需要特别处理

*   extract-text-webpack-plugin: 可以做 css 和 js 的分离，
*   同时配置 output 里的 publicPath 可以解决分离 css 之后，里面图片路径出错的问题

css 单独打包

```
            // {
            //     test: /\.css$/,
            //     use: [
            //         {
            //             loader: "style-loader"
            //         },
            //         {
            //             loader: "css-loader"
            //         }
            //     ]
            // },

            {
                test: /\.css$/,
                use: extractTextPlugin.extract( {
                    fallback: 'style-loader',
                    use: 'css-loader'
                })
            }
```

配置 output 里的 publicPath 解决图片路径出错的问题

你可以直接配置成域名，或者用/，这样变成绝对路径后，css 单独打包后，css 内部的图片路径就也是正确的了

var website = {
publicPath: "http://192.168.9.222:1818/"
};

```
    output: {
        path: path.resolve(__dirname, "dist"),
        filename: "[name].js",
        publicPath: website.publicPath
    },
```

> 10.【图片处理】 html 里通过 img 的 src 直接引入图片，这样打包后图片路径也会有问题

*   html-withimg-loader : 用起来，我是没有做什么配置，它的效果首先 html 里图片路径是没问题，而且他把打包后的 index 还压缩了

```
 {
                test: /\.(htm|html)$/i,
                loader: 'html-withimg-loader'
            }
```

> 11.对于 less 的文件，我们需求就是可以要转 css，然后配置你是否需要分离出来，单独打包

首先你肯定是项目里先安装并使用了 less

```
npm i --save-dev less
```

然后安装 less-loader

```
npm i --save-dev less-loader
```

配置顺序
css-loader:可以让引入 css 文件，把 css 当做模块处理，同时也会遍历 css 文件，里面的@import/url，它都会解析，把@import 的内容当做依赖的模块来处理。

```
     rules: [{
            test: /\.less$/,
            use: [{
                loader: "style-loader" // creates style nodes from JS strings
            }, {
                loader: "css-loader" // translates CSS into CommonJS
            }, {
                loader: "less-loader" // compiles Less to CSS
            }]
        }]
```

这样执行 npm run build 打包之后，就成功实现了 less 编译成 css 了，但是现在是直接把 less 打包到 js 里的

那么如何把 less 编译后的 css 分离呢，还是用 extractTextPlugin

它是打包到了 index.css 里

```
{
                test: /\.less$/,
                use: extractTextPlugin.extract({
                    fallback: "style-loader",
                    use: [
                        {
                            loader: "css-loader"
                        },
                        {
                            loader: "less-loader"
                        }
                    ]
                })
            }
```

> 11.sass 的编译，以及 sass 的编译成 css 后单独打包

使用 sass 有几个注意点

*   我们除了下载 sass 和 sass-loader，还需要下载 node-sass，因为 sass-loader 依赖了这个
*   写 sass 时，注意文件名称是 scss，写正则判断时不要写错了。scss 和 sass3 的新语法
*   sass 写完之后，记得要引入到 entry.js 这个入口文件里

目前打包出来的 sass 是编译后进入到 js 里的

如果要分离，还是用 extractTextWebpackPlugin

> 12.webpack 处理 css3 属性前缀 --- postcss 自动添加 css 属性前缀

postcss 是一个 css 处理平台，可以对 css 做很多处理

*   安装 postcss-loader 安装 autoprefixer

```
npm install --save-dev postcss-loader autoprefixer
```

*   创建 postcss.config.js 配置文件

*   配置使用插件

```
module.exports = {
    plugin: [
        require('autoprefixer')
    ]
}
```

*   给.css 文件加一个 loader

```
    {
        test: /\.css$/,
        use: extractTextPlugin.extract({
            fallback: "style-loader", // 处理后，最后用style-loader处理
            use: ["css-loader", "postcss-loader"]
        })
    },
```

这是一种 css 后处理器
Autoprefixer 会分析 CSS 代码，并且根据 Can I Use 所提供的资料来决定要加上哪些浏览器前缀

编译前

```
transform: rotate(45deg);
transform-origin: left;
user-select: none;
```

编译后

```
-webkit-transform: rotate(45deg);
        transform: rotate(45deg);
-webkit-transform-origin: left;
        transform-origin: left;
-webkit-user-select: none;
    -moz-user-select: none;
    -ms-user-select: none;
        user-select: none;
```


> 13.使用插件purifycss  消除未使用的css，冗余的css

2个场景下需要去掉冗余的css代码

- 使用ui库时
- 自己的写的一些无用的css，没有被使用的css代码

需要安装2个包

```
npm i --save-dev purifycss-webpack purify-css
```

```
const glob = require("glob"); // 一起来扫描文件时用
const purifycssWebpackPlugin = require("purifycss-webpack");

new purifycssWebpackPlugin({
    paths: glob.sync(path.join(__dirname, "src/*.html"))
})

```

这个插件你需要配置一个paths,会去扫描src下的html文件。

去跟你的css去碰，看看是否用了这个css，如果没用到的css就是冗余的，打包的时候就不会打包进来


> 14.Babel的核心功能就是转换ES6/7，还有就是把JSX转换成JS

```
npm i --save-dev babel-core 
npm i --save-dev babel-loader 
npm i --save-dev babel-preset-es2015

```

如果用babel转JSX的话

```
npm i --save-dev babel-preset-react
```


最简易的loader配置，转换ES6

```
{
                test: /\.(js|jsx)$/,
                exclude: /node_modules/,
                use: {
                    loader: "babel-loader",
                    options: {
                        presets: ["es2015", "react"]
                    }
                }
            }

```


另外因为babel有非常多的配置，所以一般babel的配置会写在.babelrc里

.babelrc里主要对plugins和preset预设做配置。

转换器分3种

- 语法转换器 ：针对ES7核心语法转换，但是新增的API比如，Object.assign/includes (babel-preset-env、babel-preset-es2015、babel-preset-es2016)
- 补丁转换器 : babel-plugin-transform-runtime,转换一些新增API
- jsx的转换插件 : jsx


官方推荐babel-preset-env来替代一些插件包的安装
```
{
  "presets": [
    ["env", options],
    "stage-2"
  ]
}
```

插件的话，其实就是补丁，扩展功能，早起会有一些插件，专门用来转换箭头函数语法的插件

现在官方推荐babel-preset-env来替代一些插件包的安装

这里主要介绍两款常用插件，分别是babel-plugin-transform-runtime，babel-plugin-syntax-dynamic-import。

Babel默认只转换新的JavaScript语法，而不转换新的API，比如Iterator、Generator、Set、Maps、Promise等等全局对象，以及一些定义在全局对象上的方法（比如Object.assign）都不会转码

一般会使用transform-runtime和babel-polyfill配合使用，对于后者只需要在项目入口文件require引入即可。


> 15.打包后如何调试

devtool : 关注一下速度快慢、打包出来的文件大小、打出来包的大小


> 16.开发环境和生产环境，分开配置

- --save : 生产环境的依赖
- --save-dev : 生产和开发都依赖，写在package.json就是devDepend和depends
- --production   只安装生产环境的依赖

终端执行不同的命令，打包时的前缀进行对应的修改

export type=xx传值
```
  "scripts": {
    "server": "webpack-dev-server --open",
    "build": "export type=build&&webpack",
    "dev": "export type=dev&&webpack"
  },

```


根据这个字段process.env.type判断是什么环境，不同环境换不同的前缀
```
console.log(encodeURIComponent(process.env.type));
if (process.env.type == "build") {
    var website = {
        publicPath: "http://jspang:1818/"
    };
} else {
    var website = {
        publicPath: "http://192.168.9.222:1818/"
    };
}

```

> 17.webpack的模块化配置，所有配置写在webpack.config.js太冗余了，所以利用模块化语法拆开来配置，这样职责更加清晰

> 18.优雅的第三方类库

- 普通的引入 : 在main.js这一类入口文件里，引入第三方类库，全局可用
- 通过webpack.ProvidePlugin插件的方式引入

```
   var webpack = require('webpack')   // 因为是自带的，所以必须先引用webpack 
   new webpack.ProvidePlugin( {
            $ : 'jquery'
        })

```

> watch的正确使用方法

webpack --watch : 可以做到代码改变后，保存就自动打包 （这里不太懂）


常见的有这些配置
```
    watchOptions: {
        poll: 1000, //检测1s内的修改，
        aggregeateTimeout: 500, // 防止练习按键保存，重复打包的操作
        ignored: /node_modules/
    }

```