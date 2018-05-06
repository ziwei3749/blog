# Vue

- Vue不同版本之间的区别
- Vue里的Data为什么必须是一个函数，构造时可以是对象
- watch和computed的区别
- nextTick的作用
- $set的作用，为什么一开始没有定义的就检测不到？
- created和mounted的区别
- Computed如何实现的，如何写set
- $on和$emit如何实现的
- 了解slot和provide/inject
- 了解render和createElement的写法
- vue动画
- vue和react的区别

> 1.Vue不同版本之间的区别

官方文档上，一共提供了不同构建版本的Vue，一共有8种

从2个维度上进行划分：

- 根据是否需要编译器，分为运行时版本和完整版
- 根据这个Vue代码用在什么地方，分为UMD / CommonJS / ES Module


注意点1： 用运行时版本还是完整版？
```
如果你需要用Vue的template语法，就需要完整版。但是完整版比运行时版本的文件体积要大30%，
所以我们的需求是，既想用模板语法，又希望体积小。所以用vue-loader就可以帮你把template语法构建成js
```

注意点2：用UMD / CommonJS / ES Module的区别

- 用script标签引用的话，可以用UMD
- 用低版本的打包工具的话CommonJS
- 用高版本的打包工具的话ES Module

> 2.Vue里的Data为什么必须是一个函数，构造时可以是对象

为了保证一个Vue组件实例化2次，那这个组件应该各自有一份自己的数据。
而不是2个组件里的data是指向同一个内存空间的对象。

> 3.watch和computed的区别, method和computed的区别

- computed只要依赖项不变化，就不用重新计算，但是method没有缓存。
- computed和watch的适用场景不一样。某个属性依赖多个属性的变化用计算属性。观察一个obj.name，name变化就执行某个动作用watch，比如watch页码变化，动态请求不同数据

> 4.nextTick的作用

nextTick()接受一个回调函数。当下次DOM渲染完整后执行里面的回调。

如果我们有某些代码，希望确保执行时Dom渲染完成了的，就用nextTick。因为mounted虽然是在dom渲染完整时执行，但是它不能保证当前组件的所有子组件也都dom渲染完成

> 5.$set的作用，为什么一开始没有定义的就检测不到？

因为Vue会对data里定义的数据用Object.defineProperty保证，给他们初始化getter和setter，才能实现响应式的数据。

> 6.created和mounted的区别

mounted是dom加载完成,created是data、方法已经加载，但是还没有挂在

> 7.Computed如何实现的，如何写set

- 初始化data，用Object.defineProperty去初始化getter/setter
    + getter里根据Dep.target判断是否开启依赖收集状态，开启就调用dep.depned()建立依赖关系
    + setter里，调用dep.notify()，根据建立的依赖关系，去通知计算属性变化
- 初始化computed。源码里会遍历computed对象，遍历时执行initComputed()和创建watch实例
    + initComputed的作用是把data和计算属性都混合在this下，访问计算属性就跟访问属性一样,重名报错
    + 初始化计算属性的getter,把你提供的函数作为this.reveredMessage的getter
- 首次通过this.reveredMessage获取值时,开始依赖收集
    + 每一个data/computed都有watch实例
    + 获取this.reversedMessage，会将它对应的wather实例暂存Dep.target里,之后就根据它来判断
- 然后这个Dep是订阅发布模式组织的代码


> 8.$on和$emit如何实现的

```

        class EventEmitter {
            constructor() {
                this.eventMap = {}
            }
            on( eventName, fn ) {
                this.eventMap[ eventName ] = this.eventMap[ eventName ] || []
                this.eventMap[ eventName ].push( fn )
            }

            emit( eventName, ...params ) { // 一个事件有多个fn，循环都触发掉

                this.eventMap[ eventName ].forEach( fn => {
                    fn( ...params )
                } )
            }

            off( eventName, fn ) { // 解除绑定，可以只解绑制定的fn
                var fnIndex = this.eventMap[ eventName ].indexOf( fn )
                this.eventMap[ eventName ].splice( fnIndex, 1 )

            }
        }


```
