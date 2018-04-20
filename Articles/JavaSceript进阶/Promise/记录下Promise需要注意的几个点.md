# Promise使用细节


使用promise需要注意的几点:

> 1.如何用promise实现并行的异步 (Promise.all配合.map)

- Promise.all()参数 : 一个很多promise组成的数组
- Promise.all()返回值 : 当所有异步都执行完后,promise.all的状态才变成fulfilled,返回一个各个结果组成的数组

```
       var arr = []
        for ( let i = 0; i < 5; i++ ) {
            arr.push( new Promise( ( resolve ) => {
                setTimeout( () => {
                    console.log( i )
                    resolve('结果：' + i)
                }, i * 1000 )
            } ) )
        }

        
        Promise.all(arr).then(val => {
            console.log(val)
        })

```

> 2.用Promise串行的异步 

- then的链式调用
- 配合forEach或者reduce

> 3.Promise.race的用法

Promise.race()和all类似,也可以并行,但是它是只要有一个子promise完成了,race()的状态也就完成了。

应用: 把一个异步操作和定时器放在一起。如果定时器先触发就提示用户超时

```
        let ajaxData = ''
        const ajax = new Promise( ( resolve ) => {
            setTimeout( () => {
                console.log( 'ajax' )
                ajaxData = 'ajax'
                resolve()
            }, 3000 )
        } )

        const timer = new Promise( ( resolve ) => {
            setTimeout( () => {
                if(ajaxData== ''){
                    console.log( '用户超时' )
                }else{

                }
                
                resolve()
            }, 2000 )
        } )
        Promise.race( [ timer, ajax ] ).then( (data) => {
            console.log(data)
        } )

```

 
> 4.什么是值穿透?

.then或者.catch期望传入一个函数,如果不是函数,会发生值穿透

```
      Promise.resolve(1)
            .then(2)
            .then(3)
            .then(val => {
                console.log(val)
            })

```

> 5.catch和then的第二个参数的区别?

- 比较类似,catch是一个语法糖,相当于then(null,() => {})
- 还有一点区别就是,如果第一个then报错,第二个then无法捕获。catch可以

> 6.如果catch或者then的第二个参数也抛出异常了,该如何处理?

通过全局添加一个unhandledrejection捕获Promise异常

```
window.addEventListener("unhandledrejection", (e) =>{
    console.log(e.reason)
})    

let promise = new Promise((resolve, reject) => {
    reject('Hello World')
});

promise.catch((err) => {
    throw('Unexpected Error');     // Unexpected Error
})


```


> 7.为什么then返回的都是Promise对象?

- then如果你return的不是promise对象,自动用Promise.resolve包装一下
- then的链式调用,并不是通过return this,而是通过每次创造一个新的promise实例

> 8.一道关于Promise应用的面试题 ：红灯三秒亮一次，绿灯一秒亮一次，黄灯2秒亮一次；如何让三个灯不断交替重复亮灯？（用Promse实现）

```
        function tip( timer, fn ) {
            return new Promise( resolve => {
                setTimeout( () => {
                    fn()
                    resolve()
                }, timer )
            } )
        }
        
        function step() {
            var d = Promise.resolve()
            d.then( () => {
                    return tip( 3000, () => {
                        console.log( 'red' )
                    } )
                } )
                .then( () => {
                    return tip( 1000, () => {
                        console.log( 'green' )
                    } )
                } )
                .then( () => {
                    return tip( 2000, () => {
                        console.log( 'yellow' )
                    } )
                } )
                .then(() => {
                    step()
                })
        }

        step()

```

参考和一些面试题资料 :

- [PromiseA+规范](https://promisesaplus.com/)
- [你可能不知道的promise](https://zhuanlan.zhihu.com/p/25603062)
- [一道promise面试题](http://www.cnblogs.com/dojo-lzz/p/5495671.html)
- [Promise 必知必会（十道题](https://zhuanlan.zhihu.com/p/30828196)

- [理解 Promise 的工作原理](https://cnodejs.org/topic/569c8226adf526da2aeb23fd)
- [JavaScript Promise迷你书（中文版](http://liubin.org/promises-book/)
- [Node.js最新技术栈之Promise篇](https://cnodejs.org/topic/560dbc826a1ed28204a1e7de)