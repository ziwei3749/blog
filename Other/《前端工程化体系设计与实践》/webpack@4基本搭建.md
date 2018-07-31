# webpack@4基本搭建

## webpack的开发环境的搭建

- 1.处理.vue文件
- 2.处理devServer
- 3.处理静态资源
    + 处理css/css预处理器
    + 处理图片路径
    + 处理es6
    + 处理html里的img和自动生成html
- 4.处理devtool

**1 处理.vue文件**

```js
mkdir webpack4
cd webpack4
npm init

cnpm i webpack vue-loader
```
安装的是webpack@4 和vue-loader@15,终端提示需要安装css-loader和vue-template-compiler

打包会提示，使用webpack要求webpack-cli

会报出warning,提示使用mode

配置 npm run build 脚本命令

运行 ```npm run build``` 提示VueWebpackPlugin，如果.vue里用到了css，会提示css-loader

**2 处理devServer**

启动服务，需要安装webpack-dev-server,安装出来是@3的版本

这里要区分isDev,安装cross-env，因为mac和window上设置NODE_ENV的方式不同。通过插件兼容

安装cnpm i webpack-merge --save-dev

生成html-webpack-plugin显示模板

生产环境需要CleanWebpackPlugin


其实webpack4里设置了mode后，按说应该可以不配置package.json里，还有webpack.Define了，

因为文档说是，设置mode后会把process.env.NODE_ENV修改。 我发现在.vue里可以取到值，但是在webpack配置文件里没法取到值，所以还是2种写法都用

**3 处理静态资源**

- 使用css和css预处理器
- 单独打包css
- 处理图片路径、安装url-loader和file-loader
- 处理html里的img路径

安装stylus stulus-loader

生产环境，需要用extract-text-webpack-plugin单独提取css

但是目前还没有不支持webpack4，所以用 cnpm i extract-text-webpack-plugin@next  --save-dev






**4 处理devtool**


**4 处理eslint代码规范**

https://github.com/standard/eslint-config-standard

我们使用eslint-standard，下面就是官方文档推荐安装的

```js
cnpm i eslint eslint-config-standard eslint-plugin-standard eslint-plugin-promise eslint-plugin-import eslint-plugin-node --save-dev
```


解析不了.vue里的js代码，需要用这个
```js
cnpm i eslint-plugin-html -D

```

利用这个lint命令可以检查错误
```
"lint": "eslint --ext .js --ext .vue src/"
```
加一个--fix可以自动帮我们修复


安装，因为我们用了babel,babel会帮我们处理JS语法，然后eslint去检查babel编译后的代码的话，可能就会有问题
```js
cnpm i eslint-loader babel-eslint -D
```

所以使用eslint和babel的话，就需要这个插件设置parser : "babel-eslint"

配置.editorconfi


保证不符合eslint的代码不能通过
```
安装npm i husky -D
```
```
 "precommit": "npm run lint-fix"
 ```

 必须要有git仓库