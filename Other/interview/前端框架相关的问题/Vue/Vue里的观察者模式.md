# 观察者模式

EventEmitter的实现

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

回头可以结合Vue或者《javascript设计模式和实践开发》来谈理论