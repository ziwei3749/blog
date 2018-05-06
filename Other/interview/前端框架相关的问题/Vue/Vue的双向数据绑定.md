# 双向数据绑定

双向数据绑定，是2个方向的绑定。

- 1是视图层面变化，对应的JS数据需要发生变化。
- 2是JS数据进行操作变化，对应的视图也要发生变化，而且这些变化都是自动。

那其实视图层的变化 ==> js数据自动就变化，是很好实现的。比如监听input事件，当动态的获取用户输入的值，

所以双向绑定关键是，实现JS数据做一些操作时，对应的视图能自动发生变化。因为没有专门监听JS变量的事件。

但是ES5中提供了Object.defienProperty，就可以让我们实现当JS变量的值变化时，触发setter，在setter中做一些赋值操作。


## 使用Object.defineProproty实现双向数据绑定

```
    <input type="text" id="inputDom">
    <span id="spanDom"></span>

```


```
    var inputDom = document.querySelector( '#inputDom' )
    var spanDom = document.querySelector( '#spanDom' )
    var obj = {}

    Object.defineProperty( obj, 'val', {
        set( value ) {
            console.log( value )
            spanDom.innerText = value
            inputDom.value = value
        }
    } )

    inputDom.addEventListener( 'input', () => {
        obj.val = inputDom.value
        console.log( inputDom.value )

    } )

```

> 如何给一个对象的多个属性,实现双向绑定

- 用Object.defineProperties,区别就是参数不同，第一个参数是对象，第二个参数是{},可以写键值对，键是属性名，值是用来描述的对象
- 或者遍历对象，每一个都用Object.defineProperty绑定
- 对于嵌套的对象的情况，可以用递归绑定


## 使用Proxy来实现双向数据绑定

使用Proxy里的set方法，它的触发时机跟Object.defienProperty不同。

在要变化之前，可以做一些你希望做的动作。

```
        var inputDom = document.querySelector( '#inputDom' )
        var spanDom = document.querySelector( '#spanDom' )
        var obj = {}

        obj = new Proxy( {}, {
            set( obj, prop, val ) {
                console.log( val )
                spanDom.innerText = val
                inputDom.value = val
                // console.log(obj)
                obj[ prop ] = val
                // console.log(obj)
            }
        } )

        inputDom.addEventListener( 'input', () => {
            obj.val = inputDom.value
        } )

```