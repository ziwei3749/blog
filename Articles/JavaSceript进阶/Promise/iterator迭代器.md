# iterator迭代器对象


## iterator迭代器对象

> 为什么要有iterator

因为JavaScript表示"集合"的数据结构越来越多。包括对象、数组、Map、Set

遍历一个"集合"的话,应该用for in / for of / forEach / for 到底用什么来遍历呢?

比如for in 是无法遍历 Set对象的
```
 var set = new Set( [ 1, 2 ] )

        for ( var k in set ) {
            console.log( k )
        }

```

为了有一个统一的接口,所以ES6标准里出现了iterator接口

只要是符合iterator接口的"集合",就可以使用for...of来遍历。

目前符合iterator接口的"集合"有:

- 数组
- 伪数组的对象
- Map
- Set
- String


> 认识iterator

- ES6的迭代器并不是一种新语法、或者新的内置对象,而是一种约定。符合这个约定的对象就称之为迭代器对象


> iterator的实现原理

- 创建一个指针对象,或者说迭代器对象。
- 第一次调用指针对象的next(),就会指向数据结构的第一个成员
- 第二次调用指针对象的next(),就会指向数据结构的第二个成员
- 不断的调用next,直到数据结构的最后一个位置

每一次调用next(),都会返回一个当前成员的信息对象。
 
形如: 
```
{value : 100 , done: false}
```

value表示当前成员的值,done表示迭代是否已经完成?

> 如何部署iterator接口

iterator的接口部署位置,是在对象的Symbol.Iterator属性上。调用这个方法,就可以得到迭代器对象

```
var arr = [1,2,3]

var iterator = arr[Symbol.iterator]()

console.log( iterator.next() )

```

- 数组和伪数组、Set和Map,天生就具有iterator接口

## for...of和for...in的区别

- for...in遍历的是对象里每一项的属性,for...of遍历的是对象里每一项的值
- for...of只能遍历有iterator接口的对象,没法遍历普通对象


## 实现一个遍历器生成函数

```
var it = makeIterator(['a', 'b']);

it.next() // { value: "a", done: false }
it.next() // { value: "b", done: false }
it.next() // { value: undefined, done: true }

function makeIterator(array) {
  var nextIndex = 0;
  return {
    next: function() {
      return nextIndex < array.length ?
        {value: array[nextIndex++], done: false} :
        {value: undefined, done: true};
    }
  };
}

```


