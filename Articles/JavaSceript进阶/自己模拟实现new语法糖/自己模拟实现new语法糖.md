# 自己模拟实现new语法糖


> 为什么要自己实现一个new语法糖呢?

因为之前对于JS里的new语法糖一直是理论理解,但是并亲自尝试实现过。

直到有一天,在头条的面试中 我聊high了,摸着自己的良心说: 我可以实现一个new语法糖!

面试官: 嗯,那你实现一个吧。

我: ...


## 步骤1: 观察new做了哪些事?

先随便用一下new,然后观察、脑补、想象,最后猜一下new可能干了点什么勾当!
```
    function Human(){
        this.type = 'human'
        this.age = 18
    }

    var human1 = new Human()
    console.log(h1)  // {type:'human',age:18}

```

掐指一算,先不管怎么做,直觉上new大概做了这样几件事情

- 创建一个空对象
- 让空对象的__proto__指向构造函数的prototype,并且this指向这个对象
- 偷偷return这个对象

```
function Human(){
            // var obj = {}                         创建了一个空对象
            // obj.__proto__ = Human.prototype      让空对象的__proto__指向构造函数的prototype
            // this = obj                           obj赋值给this
            this.type = 'human'
            this.age = 18

            // return this                          偷偷帮你return这个对象
        }

        var human1 = new Human()
        console.log(h1)  // {type:'human',age:18}

```

## 步骤2: 设计一下接口

面向接口编程嘛,设计一个程序,先思考最后是以什么形式调用呢? 毕竟我们不是JS之父,没法真的去创造语法。

我们打算封装一个函数newFn(),可以用来创建实例对象,然后它是这样用的:


下面是伪代码
```
 function Human(){
        this.type = 'human'
        this.age = 18
    }

 // var human1 = new Human()     不用new了,我们用newFn()来创建实例
 
 function newFn(){
    ... 写一些代码
 }
 
 var human1 = newFn(Human)   

```

## 步骤3: 实现newFn()

那么newFn()的输入和输出是什么呢?

- 参数: Constructor和数量不定的参数。我们肯定得知道是创造哪个构造函数的实例吧,所以至少需要一个Constructor,同时也可以自定义一些参数
- 返回: 返回一个obj,也即是构造函数的实例

```
      function newFn() {
            const obj = {};
            const Constructor = [].shift.call(arguments)            // 通过arguments取出第一个参数 Constructor ,并截取掉arguments第一个参数,方便之后使用   
            obj.__proto__ = Constructor.prototype;                  // 让这个obj继承一下Constructor原型链上的东西
            Constructor.apply( obj, arguments )                     // 这里是obj借用一下Constructor这个方法,从而给obj添加type 和 age

            return obj                                              // 返回对象
        }
        
        
     注: shift() 方法用于把数组的第一个元素从其中删除，并返回第一个元素的值。

```

> Example: 使用newFn去创造一个实例 (可以拿到浏览器测试用)

```

        function newFn() {
            const obj = {};
            const Constructor = [].shift.call(arguments)
            obj.__proto__ = Constructor.prototype;
            Constructor.apply( obj, arguments )
            return obj
        }


        function Human() {
            this.type = 'human'
            this.age = 18
        }

        var human1 = newFn( Human )

        console.log( human1 )

```

可以发现human1这个实例,跟我们用new语法创造出来的是一个效果。

## 小小的补充修改

为了让newFn跟new的行为表现更加相似,需要做一点点修改完善。

我们知道,构造函数一般是不写return的。它偷偷自动给你return一个实例对象

但是如果,我们手动return呢? 2个情况

- return 了一个基本数据类型,不会返回这个简单数据类型, 依旧帮你return一个实例对象
- return 了一个对象,那么就返回你这个对象。不再帮你return实例对象

所以,为了实现同样的效果,newFn修改一行代码

```

    function newFn() {
        const obj = {};
        const Constructor = [].shift.call(arguments)
        obj.__proto__ = Constructor.prototype;
        const result = Constructor.apply( obj, arguments ) // 接受一下构造函数的返回值,是对象就return该对象,否则还是return实例对象

        return typeof result === 'object' ? result : obj
    }


```

> Example: 使用完善后的newFn去创造一个实例 (可以拿到浏览器测试用)

```
        function newFn() {
            const obj = {};
            const Constructor = [].shift.call( arguments )
            obj.__proto__ = Constructor.prototype;
            const result = Constructor.apply( obj, arguments ) // 接受一下构造函数的返回值,是对象就return该对象,否则还是return实例对象

            return typeof result === 'object' ? result : obj
        }


        function Human( name, age ) {
            this.type = 'human'
            this.age = 18
            return {
                name: name,
                age: age
            }
        }

        var human1 = newFn( Human, 'ziwei', '24' )

        console.log( human1 )


```






