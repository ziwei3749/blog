# webpack4 的开发环境配置说明

开发环境的搭建，总体而言就比较轻松，因为用户就是开发者们。

你不需要考虑 hash 是否稳定、代码提取是否正常。

只要你自己能够开发、并且觉得开发体验 ok，那么你这个配置就是靠谱的。

这篇文章主要是以我自己搭建 vue + webpack 的开发为例子，记录自己从 0 搭建时的思路和遇到的坑。

所以可能不会对于配置的细节，做太详细的说明

**那么，重点说明的是思路和一些注意点**

-   1.从 0 搭建一个项目的思路
-   2.搭建过程中我遇到的坑，给大家分享一下。

下面是我们要配置的功能，也会体现从 0 搭建项目的思路和注意点。

-   1.处理 vue 文件
-   2.使用 devServer
-   3.处理静态资源的插件和 loader
    -   3.1 处理 css/css 预处理器
    -   3.2 处理图片路径
    -   3.3 处理 ES6 语法
    -   3.4 postcss 自动添加 css 前缀
    -   3.5 在 vue 里开启 css module
-   4.指定 devtool
-   5.代码规范约束配置
    -   .eslint
    -   .editorconfig
    -   pre-commit
-   6.resolve 别名设置
-   7.webpack 日志仪表盘 webpack-dashboard

## 一.处理 vue 文件

从零搭建，就是从空文件夹开始嘛。那么首先

```bash
mkdir webpack4-project

cd webpack4-project

npm init
```

初始化项目后，下一步做什么呢？

想想，我们既然要配置 vue webpack 的前端开发环境

那肯定要下载 webpack 、vue 吧，浏览器无法处理 vue 文件，那肯定需要 vue-loader 吧

**不用担心自己做的对不对，反正做错了，终端肯定会告诉你了，所以一定要自己动手**

```bash
npm i webpack vue vue-loader --save-dev
```

-   1.注意：安装 vue-loader 后，会提示你 vue-loader 依赖 css-loader 和 vue-template-compiler

```
npm i css-loader vue-template-compiler --save-dev
```

下载之后，就可以去写 webpack 配置了。

-   2.注意：如果你直接在终端输入 webpack，会提示你安装 webpack-cli。

-   3.注意：安装后尝试直接在项目里写 vue 文件，会提示你需要 loader 来处理。于是去配置 vue-loader

-   4.注意： 所以我们在 webpack.config.js 里 module 里配置，然后提示你需要 css-loader 和 VueLoaderPlugin

```js
            {
                test: /\.vue$/,
                use: 'vue-loader'
            }
```

按照要求配置 css-loader ；
VueLoaderPlugin 是 vue-loader 内置的插件，直接在 plugins 配置就好 (这个是 vue-loader@15 的变化)

但是没有 devServer，目前我们还看不到效果，接下来是开发服务器配置

## 二.使用 devServer

使用 devServer 需要安装 webpack-dev-server

安装后，就可以去配置 devServer，devServer 可以有很多参数，

但是目前可以不用关注。因为 webpack4 是有默认配置的。

-   1.注意： 这里涉及到环境变量的区分。

因为一个项目在开发和生产环境下，有不同的需求。所以自然要有一个变量用来区分环境。

webpack3 的做法是在 package.json 的 script 字段配置 NODE_ENV

类似这样

```js
  "scripts": {
    "build": "NODE_ENV=production webpack --config webpack.config.js",
    "dev": "NODE_ENV=development webpack-dev-server --config webpack.config.js"
  },
```

这样配置后，

当运行 npm run dev 时，在 webpack.config.js 里通过 process.env.NODE_ENV 可以取到值 development

以 process.env.NODE_ENV 来做 if 判断就可以啦。

那么 webpack4 中，依旧可以这样做。也可以通过 mode 来指定环境。也是设置 process.env.NODE_ENV 的值

webpack3 中一般会设置 DefinePlugin，但是 webpack4 自动帮你设置该插件

它可以让其他第三方类库或者你自己的 vue 代码中取到 process.env.NODE_ENV,在业务中判断环境。

```js
    new webpack.DefinePlugin({
      'process.env': {
        NODE_ENV: isDev ? '"development"' : '"production"' // 因为这个插件直接执行文本替换，给定的值必须包含字符串本身内的实际引号
      }
    }),
```

-   2.注意: devServer 常用配置项

```js
  devServer: {
    host: '0.0.0.0',   // IP地址
    port: 8888,        // 端口号
    hot: true,          // 开启热更新
    overlay: true,      // 开启报错提示显示在浏览器遮罩层
    historyApiFallback: true   // 设置vue-router的模式是histroy
  },
```

其中热更新在 webpack3 中需要配置插件，webpack4 依旧可以这样写。但是已经是默认配置了。

```js
plugins: [new webpack.HotModuleReplacementPlugin()];
```

现在运行 npm run dev 已经可以跑起来服务，但是没有 index.html

-   3.注意: 安装 html-webpack-plugin,指定 template 后，才能在开发服务器上可以看到自己的代码效果了

-   4.注意：区分 npm run dev 运行时的 webpack 和终端里直接输出 webpack 的区别

前者是项目本地的 webpack,后者是全局的 webpack

## 三、处理静态资源的插件和 loader

### 3.1 处理 css/css 预处理器

-   1.注意：处理 css，需要 style-loader 和 css-loader

```
style-loader:将css以style标签的形式插入到html中
css-loader: 能在js引入css依靠的就是css-loader解析
```

-   2.注意：css 预处理器，以 stylus 为例子，需要安装 stylus 和 stylus-loader

配置时，注意 style-loader css-loader stylus-loader 的顺序

webpack 的解析顺序是从右向左的。

### 3.2 处理图片路径

处理图片需要 url-loader，而 url-loader 依赖 file-loader，所以都需要下载

另外 url-loader 还可以处理字体、视频文件

```js
    {
        test: /\.(png|jpe?g|gif|svg)(\?.*)?$/,
        use: [
          {
            loader: 'url-loader',
            options: {
              limit: 10000,    // 小于10000kb自动转base64
              name: 'static/images/[name].[hash:7].[ext]'
            }
          }
        ]
      },
      {
        test: /\.(mp4|webm|ogg|mp3|wav|flac|aac)(\?.*)?$/,
        loader: 'url-loader',
        options: {
          limit: 10000,
          name: 'static/images/[name].[hash:7].[ext]'
        }
      },
      {
        test: /\.(woff2?|eot|ttf|otf)(\?.*)?$/,
        loader: 'url-loader',
        options: {
          limit: 10000,
          name: 'static/images/[name].[hash:7].[ext]'
        }
      },
```


## 3.3 处理 ES6 语法

处理es6的语法，需要babel

- **1.注意 npm 升级后，很多包的命名方式已经变更。类似于@babel/core，@babel/preset-env 这种**

- 2.注意：可以通过 exclude 来设置不用编译 ES6 的文件。通常 node_modules 的依赖包都不需要编译处理。

- 3.注意：babel-preset-env 、 babel-polyfill、babel-transform-runtime-plugin的区别

babel-preset-env
```
babel-preset-env: 可以babel需要兼容哪些浏览器。比如所有浏览器的最后2个版本。
babel-preset-env 是依据 Can i use 和 browserslist 来判断某一个 JS 语法是否需要用 babel 编译


但是babel-preset-env只能编译ES6中JS的语法，无法编译ES6中的api

比如let / const 会被编译 为var 

但是Array.include() Array.from() new Set()无法被编译
```

babel-polyfill和babel-transform-runtime-plugin
```
babel-polyfill是全局垫片，目的是提供一个完整的ES6环境，用于app，会污染全局变量，体积更大

babel-transform-runtime-plugin是局部垫片，主要用于开发库，不会污染全局变量，体积更小。

```

**结论就是一般做业务开发就 babel-preset-env + babel-polyfill 就可以。**

>文档资料

实践过程中，查阅过的文档留在这里。如果 API 忘记或者变化了，方便查阅。
https://babeljs.io/


关于@babel/preset-env的一篇文章，解释useBuiltIns的三个选项
https://www.jianshu.com/p/0dc3bddb6da8


### 3.4 postcss 自动添加 css 前缀

安装postcss-loader / autoprefixer

配置loader后，还需要创建postcss.config.js作为配置文件
```js
  module: {
    rules: [
      {
        test: /\.styl(us)?$/,
        loader: [
          'vue-style-loader',
          'css-loader',
          'postcss-loader',
          'stylus-loader'
        ]
      }
    ]
  },

```


postcss.config.js
```js
module.exports = {
    plugins: [
        require('autoprefixer')
    ]
}

```


### 3.5 在 vue 里开启 css module

参考vue-loader@15就可以
https://vue-loader.vuejs.org/zh/guide/css-modules.html

简单说就是在css-loader里增加一个配置项 module:true

这里要注意，如果你是有的vue文件用css module，有的不用的话。

需要参考文档中的【可选用法】，用oneOf来配置



## 四、指定 devtool

如果不指定devtool，你会发现代码调试起来十分不便，出错的行数也看不出来。

```js
  devtool: 'cheap-module-eval-source-map', // webpack4在开发阶段可以不写devtool
```

devtool的配置我没详细研究，我是开发阶段用cheap-module-eval-source-map，生产阶段去掉devtool


## 五、代码规范约束配置

slint的配置,我用的stardard风格的，

官方文档地址 ： https://github.com/standard/eslint-config-standard

需要依赖比较多的插件

```
npm i --save-dev eslint
```

```bash
npm install --save-dev eslint-config-standard eslint-plugin-standard 
eslint-plugin-promise eslint-plugin-import eslint-plugin-node

```

配置eslint可能遇到的问题：

- vue无法被eslint识别，配置plugin:['html']
- 希望用eslint-loader检查vue文件，但是又不能只用eslint-loader解析vue，可以设置enforce:'pre'。这样在处理.vue之前，先用eslint-loader预处理
- 记得用exclude: /node_modules/.关闭对第三方库代码风格的检查
- eslint很多报错的话，可以配置命令一次性修正
- 还可以通过一个包husky保证在git commit时，eslint报错的情况下无法提交，从而保证提交到git仓库上代码的规范性。

```js
  "scripts": {
    "lint": "eslint --ext .js --ext .vue src/ build",
    "lint-fix": "eslint --fix --ext .js --ext .vue src/ build",
    "precommit": "npm run lint"
  },

```


分享一下.editorconfi的配置,用来统一编辑器风格的

```
root = true

[*]
charset = utf-8
end_of_line = lf
indent_size = 2
indent_style = space
insert_final_newline = true
trim_trailing_whitespace = true

```

## 六.resolve 别名设置

```js
  resolve: {
    extensions: ['.js', '.vue', '.json'],
    alias: {
      vue$: 'vue/dist/vue.esm.js',
      '@src': path.resolve(__dirname, '../src')
    }
  },

```

别名的设置可能遇到的坑是，在html里可以使用别名@src

但是在css是无法识别别名。解决方案是使用 ~@src,或者在css-loader的配置项里再次设置别名


##  七.webpack 日志仪表盘 webpack-dashboard

这个是优打包出来的终端界面的一个插件

https://www.jianshu.com/p/46bdacb4c7fd
https://github.com/FormidableLabs/webpack-dashboard

使用时，需要注意的的坑是，需要配置script脚本命令

package.json配置方法像这样，其他的参考文档就可以了，像普通插件一样使用就好

```js
  "scripts": {
    "build": "NODE_ENV=production webpack-dashboard -- webpack --config build/webpack.prod.conf.js",
    "dev": "NODE_ENV=development webpack-dashboard -- webpack-dev-server --config build/webpack.dev.conf.js",
    
  },
```