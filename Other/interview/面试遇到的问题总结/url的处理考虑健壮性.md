# url的处理要考虑健壮性


问题1:需要考虑对hash的处理，也就是对#的处理

问题2:需要考虑是否有？的处理，没有的话要返回{}

比较好的做法是，通过window.location.href得到url地址

通过new URL(window.location.href).search得到queryString

问题3: 考虑到 name&age=18的情况，我们希望自动处理为{name : null }

问题4: 考虑到 tag=all?name=ziwei, 我们希望自动处理为 {tag : all?name=ziwei}



// 考虑是否有#hash值，有的话需要去掉
// 考虑是否有？，有的话就很好，没有的话直接返回{}

// 通过window.location.href
// new URL().search

```
const parseQueryString = (str) => {
    str = new URL(str).search
    if(str == '') return {}
    var obj = {}
    var arr = str.substr(1).split('&')

    var result = arr.reduce((obj,item) => {
        var [k,...v] = item.split('=')
        obj[k] = v.length === 0 ? null : v.join('=')
        return obj
    },{})

    return result
}

```

补充知识:

reduce可以处理一些累加、累乘

arr.reduce((obj,item) => {
    return obj
},{})

例子用reduce实现累乘
var arr = [1,2,3,4]
reduce((result,item) => {
    return result * item
},1)