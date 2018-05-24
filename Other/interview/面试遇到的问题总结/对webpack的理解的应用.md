# 简历

- webpack是如何配置的，说说你常用的配置以及踩过哪些坑
- 你是如何利用webpack优化性能，提升构建项目时的开发速度的


配置

> 1.修改devtool的source-map。要注意生产环境和开发环境要分开。比如我用inline-source-map打包过，打出来的包就特别的大，inline-source-map会给被一个打包前的文件添加dataUrl,base64转码后的字符串


- eval: 每一个模块转化为字符串，在尾部添加 souceURL

- source-map : 生成一个.map文件,打包文件末尾会添加souceURL

- inline-source-map: 给每一个打包前的文件添加dataURL,dataURL包括完成的soucemap信息，做base64转码后的字符串

- cheap-source-map : 跟sourcemap相同，但是不包含列信息，不包含Loader的sourcemap



最终的配置是

生产环境用： cheap-module-eval-source-map
开发环境用： cheap-module-source-map


相关解释：

- 1.用cheap是因为就是sourcemap没有列信息，很多浏览器也是会给出列信息的，使用cheap模式提高soucemap生成的效率
- 2.使用Module可支持的babel这种预编译工具
- 3.使用eval的话就是把模块转化为字符串，可以大幅度提高持续构建效率。webpack官网上的速度对比表。对于前端比较重要


> 2.使用extract-text-webpack-plugin，可以把css和js单独打包。

这样免得以后只修改css，导致浏览器端所有缓存都失效

> 3.开启gizp压缩，用CompressWebpackPlugin。

用法就是new CompressWebpackPlugin（{
    test : js html,
    threshold : 10240  只有大小大于改值的资源会被处理 
}）


> 4.purifyCssPlugin将冗余的css代码，主要包括UI库里无效的css代码删除

> 5.webpack可以做代码分割和按需加载  require.ensure()

另一个写法就是可以用CommonChunkPlugin合并文件