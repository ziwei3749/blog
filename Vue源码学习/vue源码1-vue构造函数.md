# vue源码1-Vue构造函数

我们使用Vue,在main.js去new Vue()

所以基本上实例化vue基本是一个项目的开始，所以学习源码也可以先研究Vue这个构造函数

## 顺藤摸瓜，找到Vue构造函数

- 查看package.json里的script里的dev，运行这个构建命令，是执行rollup的配置文件
- 这个rollup里的配置文件的入口是 web/entery-runtime-with-compiler
- 那么web来自哪里呢？是在scriptt/alias里的一个别名配置
- 找到这个 web/entery-runtime-with-compiler.js后，查看Vue是从哪来的 

```js
import Vue from './runtime/index'
```

- 这说明这个entry-runtime-with-compiler.js并不是vue的出生地，我们要一路追查下去

- runtime/index.js里的Vue来自  

```js 
import Vue from 'core/index'
```

- core/index.js里的Vue来自 

```js 
import Vue from './instance/index'
```

- 进入instance/index后，发现Vue这个构造函数在源码里定义的地方


```js
import { initMixin } from './init'
import { stateMixin } from './state'
import { renderMixin } from './render'
import { eventsMixin } from './events'
import { lifecycleMixin } from './lifecycle'
import { warn } from '../util/index'

function Vue (options) {
  if (process.env.NODE_ENV !== 'production' &&
    !(this instanceof Vue)
  ) {
    warn('Vue is a constructor and should be called with the `new` keyword')
  }
  this._init(options)
}

initMixin(Vue)
stateMixin(Vue)
eventsMixin(Vue)
lifecycleMixin(Vue)
renderMixin(Vue)

export default Vue

```


我们发现在最初定义Vue的地方,代码十分清晰

- 从5个文件，引入了5个方法 initMixin / stateMixin / renderMxiin / eventsMixin / lifecycleMixin

- 将Vue作为参数，传递给这5个方法

- 构造函数内部核心就一句话，调用了vm._init(options)

- 导出Vue


## Vue构造函数的prototype

> 首先, 看一下./init.js

这个方法接受Vue作为参数，并且在Vue.prototype上挂在了_init()

这就是我们刚刚看到的，在Vue构造函数里的 _init

```js
this._init(options)
```

> 再看一下，./state.js

找到里面stateMixin方法


重点关注这一部分
```js
  Object.defineProperty(Vue.prototype, '$data', dataDef)
  Object.defineProperty(Vue.prototype, '$props', propsDef)

  Vue.prototype.$set = set
  Vue.prototype.$delete = del

  Vue.prototype.$watch = function (
```

可以看到stateMixin和initMixin一样，再拿到Vue之后，都往Vue.prototype上挂了一些东西

其中的

```js
Object.defineProperty(Vue.prototype, '$data', dataDef)
Object.defineProperty(Vue.prototype, '$props', propsDef)
```

可以看一下dateDef 和 propsDef的是干嘛的

他们的get就是this._data和this._props

他们的set如果不是生产环境，就直接报警高了，其实就是提醒就data和props是只读的。

```js
  const dataDef = {}
  dataDef.get = function () { return this._data }
  const propsDef = {}
  propsDef.get = function () { return this._props }
  if (process.env.NODE_ENV !== 'production') {
    dataDef.set = function (newData: Object) {
      warn(
        'Avoid replacing instance root $data. ' +
        'Use nested data properties instead.',
        this
      )
    }
    propsDef.set = function () {
      warn(`$props is readonly.`, this)
    }
  }
```

> 再看一下eventsMixin方法

你会发现思路是类似，eventsMixin做的事情也是类似的

拿到Vue构造函数，往Vue构造函数上挂在$emit/$on/$off/$once这些方法

```js
export function eventsMixin (Vue: Class<Component>) {
  const hookRE = /^hook:/
  Vue.prototype.$on = function (event: string | Array<string>, fn: Function): Component {}
  Vue.prototype.$once = function (event: string, fn: Function): Component {}
  Vue.prototype.$off = function (event?: string | Array<string>, fn?: Function): Component {}
  Vue.prototype.$emit = function (event: string): Component {}
}
```

> 看看lifecycleMixin往Vue上挂载了什么

```js
Vue.prototype._update = function (vnode: VNode, hydrating?: boolean) {}
Vue.prototype.$forceUpdate = function () {}
Vue.prototype.$destroy = function () {}
```

> 最后是renderMixin


不出意外的话，还是在Vue.prototype上挂在东西吧？

可以看到这里挂在了_render和$nextTick
```js
installRenderHelpers(Vue.prototype)
Vue.prototype.$nextTick = function (fn: Function) {}
Vue.prototype._render = function (): VNode {}
```

注意installRenderHelpers把Vue.prototype作为参数传递了进去，这个方法的源码如下

其实也是往vue.prototype上挂在一系列的render helper方法

```js
/* @flow */

import { toNumber, toString, looseEqual, looseIndexOf } from 'shared/util'
import { createTextVNode, createEmptyVNode } from 'core/vdom/vnode'
import { renderList } from './render-list'
import { renderSlot } from './render-slot'
import { resolveFilter } from './resolve-filter'
import { checkKeyCodes } from './check-keycodes'
import { bindObjectProps } from './bind-object-props'
import { renderStatic, markOnce } from './render-static'
import { bindObjectListeners } from './bind-object-listeners'
import { resolveScopedSlots } from './resolve-slots'

export function installRenderHelpers (target: any) {
  target._o = markOnce
  target._n = toNumber
  target._s = toString
  target._l = renderList
  target._t = renderSlot
  target._q = looseEqual
  target._i = looseIndexOf
  target._m = renderStatic
  target._f = resolveFilter
  target._k = checkKeyCodes
  target._b = bindObjectProps
  target._v = createTextVNode
  target._e = createEmptyVNode
  target._u = resolveScopedSlots
  target._g = bindObjectListeners
}

```

## Vue构造函数的静态属性 （global-api）

那么回顾一下 顺藤摸瓜的线索

- package.json
-  -->  web/entery-runtime-with-compiler.js
-  -->  ./runtime/index
-  -->  core/index
-  -->  ./instance/index

那么看完 instance/index.js 了，下一步就是看 core/index了

看一下core/index,

引入了initGlobalAPI函数，把Vue传递进去，做的事情很好理解

就是给Vue挂在了各种静态方法和属性

文档上见过的全局方法，都在在这个initGlobalAPI里做的挂载
```js
// 从 Vue 的出生文件导入 Vue
import Vue from './instance/index'
import { initGlobalAPI } from './global-api/index'
import { isServerRendering } from 'core/util/env'
import { FunctionalRenderContext } from 'core/vdom/create-functional-component'

// 将 Vue 构造函数作为参数，传递给 initGlobalAPI 方法，该方法来自 ./global-api/index.js 文件
initGlobalAPI(Vue)

// 在 Vue.prototype 上添加 $isServer 属性，该属性代理了来自 core/util/env.js 文件的 isServerRendering 方法
Object.defineProperty(Vue.prototype, '$isServer', {
  get: isServerRendering
})

// 在 Vue.prototype 上添加 $ssrContext 属性
Object.defineProperty(Vue.prototype, '$ssrContext', {
  get () {
    /* istanbul ignore next */
    return this.$vnode && this.$vnode.ssrContext
  }
})

// expose FunctionalRenderContext for ssr runtime helper installation
Object.defineProperty(Vue, 'FunctionalRenderContext', {
  value: FunctionalRenderContext
})

// Vue.version 存储了当前 Vue 的版本号
Vue.version = '__VERSION__'

// 导出 Vue
export default Vue
```

## Vue的平台化包装

那么回顾一下 顺藤摸瓜的线索，我们已经看了2个文件了

- package.json
-  -->  web/entery-runtime-with-compiler.js
-  -->  ./runtime/index
-  -->  core/index
-  -->  ./instance/index


因为Vue支持多平台，所以在不同平台上的Vue构造函数有一些差异化的属性

这些属性肯定不会定义在core文件，因为core文件是和平台无关的核心代码

这些属性，是定义在platforms下的代码，接下来，我们会看platforms/web/runtime/index.js


```js
// install platform specific utils
Vue.config.mustUseProp = mustUseProp
Vue.config.isReservedTag = isReservedTag
Vue.config.isReservedAttr = isReservedAttr
Vue.config.getTagNamespace = getTagNamespace
Vue.config.isUnknownElement = isUnknownElement

// install platform runtime directives & components
extend(Vue.options.directives, platformDirectives)
extend(Vue.options.components, platformComponents)

// install platform patch function
Vue.prototype.__patch__ = inBrowser ? patch : noop

// public mount method
Vue.prototype.$mount = function (
  el?: string | Element,
  hydrating?: boolean
): Component {
  el = el && inBrowser ? query(el) : undefined
  return mountComponent(this, el, hydrating)
}
```

这里核心逻辑就是

设置平台化的Vue.config

在Vue.options上混合了2个指令，分别是model和show
在Vue.options上混合了2个组件，分别是transition和transiton-group
在Vue.prototype上添加了2个方法 _patch_ 和 $mount

## with compiler

那么回顾一下 顺藤摸瓜的线索，我们已经看了3个文件了

- package.json
-  -->  web/entery-runtime-with-compiler.js
-  -->  ./runtime/index
-  -->  core/index
-  -->  ./instance/index

目前为止运行时版本的Vue函数已经成型了

不信，你看entry-runtime.js这个入口文件，这有2句话

就是把runtime/index.js引入，再导出
```js
import Vue from './runtime/index'

export default Vue
```

按照套路，我们还没需要看一下 web/entery-runtime-with-compiler.js

这个是带编译器的Vue,应该比我们现在runtime版本，多一些编译相关的代码

那么，其实我们已经知道它做了啥事了，就看一下它是如何做的编译吧

```js

import Vue from './runtime/index'
import { compileToFunctions } from './compiler/index'


// 使用 mount 变量缓存 Vue.prototype.$mount 方法
const mount = Vue.prototype.$mount
// 重写 Vue.prototype.$mount 方法
Vue.prototype.$mount = function (
  el?: string | Element,
  hydrating?: boolean
): Component {
  // ... 函数体省略
}


// 在 Vue 上添加一个全局API `Vue.compile` 其值为上面导入进来的 compileToFunctions
Vue.compile = compileToFunctions

```

引入了运行时的vue，还引入了compileToFunctions

缓存了mount，并且重写了mount

这个文件运行下来，对 Vue 的影响有两个，

- 第一个影响是它重写了 Vue.prototype.$mount 方法；
- 第二个影响是添加了 Vue.compile 全局API，目前我们只需要获取这些信息就足够了，我们把这些影响同样更新到 附录 对应的文件中，也都可以查看的到。



中间还有一段值得注意的代码被省略了，但是最好记住

那就是,他会把判断你是否有render，有的话就转化成render方法，它是通过compileToFunctions来转化的

也就是说，Vue最终只认识render方法