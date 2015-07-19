#Lua

### Lua的全局变量
Lua的全局变量存储在`_G`这个表里。

### 为什么应该多用`local`变量，而不用`global`变量？
`global`变量实际上是放在全局变量`_G`的这个table里。每次访问时，变量名作为Key从全局变量表中调用。而`local`变量则是存储在堆栈之中，比访问全局变量更快。有些时候，不经意间就访问了全局变量，比如
```lua
for k1,v1 in pairs(tbl) do
    for k2,v2 in pairs(v1) do
        ...	
    end
end
```
这段代码，用双重循环操作迭代一个table，这里`pairs`其实就是一个全局变量应用的函数，为了提升效率，我们可以写成这样
```lua
do
    local pairs=pairs
    for k1,v1 in pairs(tbl) do
        for k2,v2 in pairs(v1) do
            ...	
        end
    end
end
```
效率就会稍微提高一些。但是注意，如果只有一重循环的话，这样做就没有什么意义了。因为`for ... in`循环中的`pairs` 这个函数只会被调用一次，而不是每次循环都去调。我们的原则其实是，被多次读取的`global`变量，都应该提取出来放到`local`变量中。

### Lua的函数调用时，`:`和`.`的区别？
首先看一段stackflow上的代码
```
> x = {foo = function(a,b) return a end, bar = function(a,b) return b end, }
> return x.foo(3,4)
3
> return x.bar(3,4)
4
> return x:foo(3,4)
table: 0x10a120
> return x:bar(3,4)
3
```
可以发现，同样的函数，返回的值却有很大的区别。
- 使用`x.foo()`，就和其他语言一样调用函数。
- 使用`x:foo()`，第一个参数会默认将自己`self`作为参数传入函数，所以`x:foo(3,4)`返回的是对`x`这个table。

### Lua的`require`
简单说来，`require`可以调用其他Lua文件中的代码，称为模块，有时候，我们会用到`local foo = require 'foo'`这种形式，这其实也是避免全局调用，提升效率。
