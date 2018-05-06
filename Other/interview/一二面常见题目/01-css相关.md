# CSS相关

- 1.布局
- 2.BFC
    + BFC的特点以及应用
    + 如何创建BFC
- 3.层叠上下文
- 4.单行...和多行...

```
           text-overflow:ellipsis;
            white-space: nowrap;
            overflow: hidden;

```

```
        display: -webkit-box;
        -webkit-box-orient: vertical;
        -webkit-line-clamp: 2;
        overflow: hidden;

```

- 5.flex

- 6.css动画

```
      div {
            width: 200px;
            height: 200px;
            background: red;
        }

        @keyframes myfirst {
            from {
                background: red;
            }
            to {
                background: yellow;
            }
        }

        div:hover {
            animation: myfirst 1s;
        }

```