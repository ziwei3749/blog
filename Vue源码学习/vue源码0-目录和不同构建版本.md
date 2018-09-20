# Vue源码目录 和 不同的构建版本

## Vue源码目录

- dist : 8个vue版本
- scripts : 构建相关的rollup配置文件
- flow : vue使用flow做静态类型校验
- src : 源码
    + compiler : 编译器代码
    + core : 和平台无关的核心代码
        * instance : vue构造实例化
        * global-api : 全局api
        * observer : 响应式代码
        * vdom 
        * components
        * util
    + platforms : 和平台相关的代码
    + server : 服务端渲染代码
    + sfr
    + shared
- test


> flow

flow做静态类型分析

- 1.类型判断: 根据变量上下文，自动判断你type是否异常
- 2.类型注释: 你自己写你的期望类型

## 不同的构建版本

一共8个版本

我们不要从1到8去记忆，这8个版本的区别

从2个角度是区分他们。

- 1是根据是否带编译器，分为运行时和完整版
- 2是根据模块规范，可以分为cmd / umd / esm

那么umd模块规范又叫通用模块规范，它的2个版本，还分dev和production，所以一共是8个版本


> 运行时和完整版的区别

就是首先，我们需要知道vue2中我们是引入虚拟dom嘛，最终vue源码是认识render方法的。

所以template模板语法是必须compiler的。

运行时和完整版的区别就是，是否带编译器。

那么这个怎么选呢？ 在实际场景中，我们一般会选择「运行时版本」 + 「vue-loader」来开发

因为运行时的话，体积小30%。而且vue-loader在构建时做编译，这样体积也小，我们也可以用template提供的方便地语法。


> 模块规范分为cmd / umd / esm

查看vue源码的rollup的配置的出口、 或者package.json里的script标签

都可以发现vue通过不同的命令行，可以构建出支持各种模块规范的版本。

支持各种模块规范的目的，当然就是为了满足尽可能多的用户呗,

- 比如你用webpack1的话，就需要cmd规范的版本
- 高版本webpack或者rollup的话就需要esm模块规范的版本
- 如果是script的src的方式引入的话，可以用umd模块规范，这样就不同配合各种构建工具了。

**最终效果就是，不管你用什么构建工具都可以方便的引入vue，甚至你不用构建工具也可以使用vue。**




