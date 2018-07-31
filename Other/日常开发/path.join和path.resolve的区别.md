# path.join和path.resolve的区别


一句话解释的就是，join是拼接路径，resolve是解析路径

join直接拼接，比较呆板

区别1： resolve一定会帮你解析出一个绝对路径，从右往左解析，当遇到/就直接返回，不再继续向左解析了。

```
 path.join('/a','/b')   ==>  /a/b

 path.resolve('/a','/b') ==> /b     因为遇到/，判断已经是绝对路径了

```



区别2： resolve过程里,如果没遇到/会自动补全一个绝对路径


```
path.join('a','b1','..','b2')    ==> a/b2


path.resolve('a','b1','..','b2')    ==>   /home/job/node/a/b2


```







