# 实现数组的shift、unshift、pop、push

## unshift

> 实现数组原生的unshift方法 : 在数组头部，增加一些元素

- 注意unshift方法的返回值： 返回新数组的长度
- unshift方法，我是需要返回一个新数组，还是对原来的数组做修改
- 实现思路，注意不能用for(var i = 0 ;i<arr.length ; i++),会导致所有元素都被赋值为undefined
- 最后我再看文档时，发现unshift并不是往数组最前面插入一个，其实也可以插入多个
- this、原型链的基本使用

```
        Array.prototype.unshift = function ( ...elements ) {

            for ( let i = this.length + ( elements.length - 1 ); i >= 0; i-- ) {
                this[ i ] = this[ i - ( elements.length ) ]
            }
            console.log( this )
            for ( var i = 0; i < elements.length; i++ ) {
                this[ i ] = elements[ i ]
            }
            return this.length
        }

```

## push

> 思路:
-  push一个的话，就是 this[this.length] = params
-  push n 个的话，就是 this[this.length + i] = params[i]
-  要注意this.length的长度会变化，所以最初用变量保存了一下。

```
        Array.prototype.push = function ( ...elements ) {

            var initLength = this.length   // 保存一下最初的长度
            for ( var i = 0; i < elements.length; i++ ) {
                this[ initArrLength + i ] = elements[ i ]   
            }

            return this.length
        }

```

## shift

> 这里我arr.length--,它删除掉array里最后一个元素，说明数组符合栈的特点后进先出

- 作用: 删除数组头部的第一个元素
- 没有参数
- 返回值: 返回被删除的那个元素 


```
Array.prototype.shift = function () {
        var fitstItem = this[ 0 ]
        for ( var i = 0; i < arr.length; i++ ) {
            arr[ i ] = arr[ i + 1 ]
        }
        arr.length--;    // 这里我通过数组长度减一，自动就减掉数组的最后一项
        return fitstItem
    }

```

## pop

- 作用: 删除数组尾部的第一个元素
- 没有参数
- 返回值: 返回被删除的那个元素 

```
   Array.prototype.pop = function () {
            var lastItem = this[ this.length-1 ]
            arr.length--; // 这里我通过数组长度减一，自动就减掉数组的最后一项
            return lastItem
        }

```