# 前端路由

前端路由原理: 直接找到与地址匹配的一个组件，然后把它渲染出来

改变浏览器地址，但不向服务端发出请求的方式有2个:

- 使用html5的window.history，使用url来模拟一个完成url
- 在地址中加入#



实现的需求和效果有2点:

- 比如有入口1、2、3，点击之后对应跳转到页面1、2、3，而且这个跳转不是访问服务端接口，而是JS控制的
- 当浏览器点击了前进、后退,我们要有一个正确的响应，也就是要有历史记录单。


## history

这个2需求，其实第一个点比较容易做到，第二个History有提供对应api

- history.onpopstate可以响应浏览器的前进、后退，在这个事件里可以做操作
- histroy.pushState可以创建历史记录单


## hash

- location.hash，跳转时我们修改hash
- hashchange，监听hash的变化，然后在事件里做对应的动作


## 代码的实现

> history

- pushState可以实现，点击入口1，url对应也发生变化(url变化应该加载对应组件)。也可以传参数
- popstate是监听url的变化和浏览器的前进后退，可以接受到参数，监听到变化

```
    <a class="api a">a.html</a>
    <a class="api b">b.html</a>

```

```
          // 注册路由
            document.querySelectorAll( '.api' ).forEach( item => {
                item.addEventListener( 'click', e => {
                    e.preventDefault();
                    let link = item.textContent
                    console.log( link )
                    window.history.pushState( {
                        name: 'api'
                    }, link, link )
                }, false )
            } )

            // 监听路由

            window.addEventListener( 'popstate', e => {
                console.log( {
                    location: location.href,
                    state: e.state
                } )
            } )

```

> hash的实现

- 点击不同btn，修改location.hash，传递参数可以location.hash + ?
- onhashchange监听hash的变化，前进后退也可以正常响应，因为前进后退也会触发事件嘛

```
        <a class="hash a">#a.html</a>
        <a class="hash b">#b.html</a>
```

```
            // 注册路由
            document.querySelectorAll( '.hash' ).forEach( item => {
                item.addEventListener( 'click', e => {
                    e.preventDefault();
                    let link = item.textContent
                    console.log( link )
                    location.hash = link
                }, false )
            } )

            // 监听路由

            window.addEventListener( 'hashchange', e => {
                console.log( {
                    location: location.href,
                    hash: location.hash
                } )
            } )

```