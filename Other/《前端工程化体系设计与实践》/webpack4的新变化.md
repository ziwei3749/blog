# 简述webpack4的新变化

这篇文章也只是简答的记录一下webpack4的主要变化。

实际上，和自己目前的配置相比究竟有哪些变化，还是需要自己实践，一些细节不太容易被注意

比如新增的[contenthash]替换[chunkhash]，但是目前文档还没有更新说明。


## 1、新增mode字段，默认处于production

设置mode字段后，就类似以前的process.env.NODE_ENV设置环境变量，会针对不同环境自动做优化处理

mode : development模式下的webpack会优化你的构件速度、错误日志的提示更友好、不过devtool还是要设置
mode : production模式下的话,自动调用uglify打包


## 2、取消CommonsChunkPlugin,用optimaztion替代

- 总体来说配置起来更简洁一点。比如提取manifest，需要另外再new CommonsChunksPlugin()，现在只需要optimization.runtimeChunk: true

## 3.extract-text-webpack-plugin，推荐用mini-css-extract-plugin。

关于这个我自己也有一推踩坑经验。

因为到写这篇文章7月份为止，extract-text-webpack-plugin依旧没有支持webpack@4，但是extract-text-webpack-plugin@next是支持的

但是如果你用了测试版本的话，会使用发现[content hash]会报错，可能是因为和webpack4内置的content hash产生了冲突

https://github.com/webpack-contrib/extract-text-webpack-plugin/issues/763


```
css使用[content hash]，而不像js一样用[chunkhash]，是以为他们的chunkhash相同，这样的话修改js或者css，都会影响js/css的文件名，从而导致缓存失效
```

所以这里用mini-css-extract-plugin就可以实现，修改js时，不会改变css的文件名。

**但是修改css时，js的文件名会被改变。解决方法是在output.filename设置js的文件名时也用[contenthash]  **

```
这个问题在webpack3中是不存在的。webpack3一般的做法就是css设置contenthash，js设置为chunkhash就出现互相缓存影响的问题。

但是在webpack4中出现了，而是用contenthash来解决也不完全确定是否正确。因为英文官方文档目前也搜索不到contenthash的定义。

可以推测的理解为，css和js分离后，确实都是还是同一个chunk，那么css也是chunk的一部分，修改css导致chunkhash变化也是可以理解。

```

## 4.剩下就是一些一些包需要升级

- webpack-cli是需要新增的一个包，把一些命名放到webpack-cli，总之会提醒必须安装
- webpack-merge/webpack-dev-server都是要升级的
- css-loader都是要升级，按照提示升级就行了
- NoEmitOnErrorsPlugin- > optimization.noEmitOnErrors（默认情况下处于生产模式）
- ModuleConcatenationPlugin- > optimization.concatenateModules（默认情况下处于生产模式）
- NamedModulesPlugin- > optimization.namedModules（在开发模式下默认开启）

## 5.吐槽一下

webpack的升级实在是够频繁的，很多时候小版本的变化也蛮明显。

比如我在用@3.10.0时，必须要提取manifest，否则会导致业务代码，第三方类库的文件hash也变化
而我在@3.12.0时，也就是webpack3的最高版本，就不是这种情况了。不用提取第三方类库的hash也不受影响。
