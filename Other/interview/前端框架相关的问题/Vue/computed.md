# computed

> 模板里不适合写太长的js逻辑，因为不易读而且不易复用，所以复杂一点的逻辑，应用计算属性，而不是堆积着在模板里写

> 计算属性默认只有getter,如果需要也可以设置setter,但是并不建议这么做，如果让逻辑变得混乱。

## 计算属性缓存 vs method

- computed是基于它依赖的属性进行缓存的，也就是只要依赖的属性没有变化，多次访问computed，computed是直接返回数据的，也不影响性能。

- method是每次触发重新渲染时，调用的方法总会执行

## 计算属性 vs watch

- computed是当我这个计算属性，需要依赖多个变量时，用computed可以方便的计算，而且有缓存机制
- watch观察的意思。就是监听对象的某一个属性，一旦变化就触发一个动作。这个动作可以是异步的、也可以复杂。比如监听页码的变化，根据页码请求不同的数据。


## computed的实现原理

- 1.对data属性做处理，初始化他们的getter和setter
- 2.对computed计算属性初始化，提供的函数将作为this.reverseMessage的getter
- 3.当首次获取reverseMessage计算属性的值时，Dep开始依赖收集
- 4.在执行message的getter方法时，如果Dep处于依赖收集状态，则判定message为reversedMessage的依赖，并建立依赖关系
- 5.当message发生变化时，根据依赖关系，触发reverseMessage的重新计算。

步骤1：

```
首先要对我们在data里定义的对象，用Object.defineProperty重新定义，这样才是响应式的嘛。

通过data是一个函数嘛，调用一个data()就可以得到这些数据。得到这个对象后，通过Object.definePropety初始化getter和setter，里面分别做了什么先不提，后面会说
```

步骤2：

```
对computed计算属性初始化。提供的函数会作为this.reverseMessage的getter来初始化

因为计算属性默认就是getter，特殊需求也可以显示的给计算属性写setter，因为可能让逻辑变得混乱，所以不太建议。

初始化方法就是initComputed，这个方法里，这个方法，

还会调用一个defineComputed(),先不纠结这个方法的实现，先说这个方法做的事情，就是刚说的创建this.reverseMessage，并且把我们提供的函数，作为this.reverseMessage的getter来初始化。

最终computed和data会混合到this下,所以当computed和data如果重名，Vue也会抛出警告

会创建watcher实例

```

步骤3：

```
其实无论是属性还是计算属性，都会生成一个对应的watcher实例

当通过this.reversedMessage获取属性时，Dep就会开始依赖收集。

具体就是，获取this.reversedMessage，会将它对应的wather实例暂存Dep.target里,这就代表开启了依赖收集任务

```

步骤4:
```
在执行message的getter时，会判断是否处于依赖收集状态，根据Dep.target，如果依赖收集状态，就建立依赖关系

 get: function reactiveGetter () {
    const value = getter ? getter.call(obj) : val
    // 判断是否处于依赖收集状态
    if (Dep.target) {
      // 建立依赖关系
      dep.depend()
      ...
    }
    return value
  },
```

步骤5:
```
在执行message的setter时，也就是被依赖的属性发生变化时，会执行dep.notify()来通知计算属性变化

  set: function reactiveSetter (newVal) {
    ...
    // 依赖发生变化，通知到计算属性重新计算
    dep.notify()
  }

```

关于Dep的代码短小精悍，作用就是依赖收集,用了订阅发布模式的设计方式。

```
export default class Dep{
    static target   // 用了flow，类型检查

    constructor(){
        this.subs = []
    }

    addSub(sub){
        this.subs.push(sub)
    }

    removeSub(sub){
        remove(this.subs,sub)
    }

    depend(){
        if(Dep.target){
            Dep.target.addDep(this)
        }
    }

    notify(){
         const subs = this.subs.slice()
        for (let i = 0, l = subs.length; i < l; i++) {
        // 更新 watcher 的值，与 watcher.evaluate() 类似，
        // 但 update 是给依赖变化时使用的，包含对 watch 的处理
        subs[i].update()
        }
    }

}

```


参考： https://segmentfault.com/a/1190000010408657

### 面试题： computed的实现原理是什么？如何实现一个？

- 首先手写一个computed使用的基本例子，结合例子谈谈源码实现细节
- 步骤1:用过Object.defineProperty来初始化getter和setter
    + 提到data()是函数  
    + 提一下getter里通过Dep.target判断是否是依赖收集状态，是的话就dep.depend()建立依赖关系
    + 提一下setter里通过dep.notify()来通知计算属性变化。
- 步骤2: 初始化Computed计算属性，遍历computed对象，执行initComputed()和创建watcher实例
    + initComputed的作用就是创建this.reversedMessage属性，并且将提供的函数作为它的getter
    + 最终computed和data属性会一起混合到this下，所有当computed和data重名的话会爆出警告
- 步骤3: 无论是data属性还是计算属性，都会生成对应的watcher实例
    + 比如这个例子里的通过this.reversedMessage获取计算属性时，就会进入到一个方法里
    + 这个方法，调用pushTarget(this)把watcher存到Dep.target里，push进去就表示开启了依赖收集
- 步骤4： message的getter
- 步骤5： message的setter


- 这里的Dep类的作用就是依赖收集的作用，看源码里的代码组织方式，是用了订阅发布模式。（可能会让你手写订阅发布模式）


### 面试题： 实现一个EventEmitter

完成 EventEmitter 模块，它是一个类，它的实例具有以下几个方法：on、emit、off：

- on(eventName, func)：监听 eventName 事件，事件触发的时候调用 func 函数。
- emit(eventName, arg1, arg2, arg3...)：触发 eventName 事件，并且把参数 arg1, arg2,arg3... 传给事件处理函数。
- off(eventName, func)：停止监听某个事件。

使用例子:

```
const emitter = new EventEmitter()
const sayHi = (name) => console.log(`Hello ${name}`)
const sayHi2 = (name) => console.log(`Good night, ${name}`)

emitter.on('hi', sayHi)
emitter.on('hi', sayHi2)
emitter.emit('hi', 'ScriptOJ')
// => Hello ScriptOJ
// => Good night, ScriptOJ

emitter.off('hi', sayHi)
emitter.emit('hi', 'ScriptOJ')
// => Good night, ScriptOJ

const emitter2 = new EventEmitter()
emitter2.on('hi', (name, age) => {
  console.log(`I am ${name}, and I am ${age} years old`)
})
emitter2.emit('hi', 'Jerry', 12)
// => I am Jerry, and I am 12 years old

```

答案：

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