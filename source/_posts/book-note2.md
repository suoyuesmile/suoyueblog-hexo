---
title: 《JavaScript高级程序设计》读后记<二>：作用域
date: 2015-11-27 21:18:03
categories: JavaScript
tags: JavaScript
---
在第一篇博客里面，提到过变量是松散变量，决定它只是特定时间保持特定的值的符号而已，而变量的值和数据类型可以在脚本的生命周期内改变的。接下来就是理解这种特性的性质了。

### 理解引用类型，作用域
前面我们已经知道了，ES 变量类型包括 5 种基本类型和引用类型。

__基本类型与引用类型的区别？__
- 基本类型值在内存中占据固定大小的空间，保存在栈中，引用类型的值是对象，被保存在堆中
- 这 5 种基本类型可以按值访问的，而引用类型的值是按引用访问
- 基本类型的变量包含的是实际的值，引用对象实际包含的并不是对象本身而是指向对象的指针。

__这里要解释一下什么是按值访问？什么是按引用访问？__
如果学过 C++ ,我们就会知道 C++ 传参有两种方式，一种是传值，一种是传引用。

__这两种方式有什么区别呢？__
区别在于传值是将实参拷贝给形参，而传引用并不是简单的赋值，而是将形参与实参绑定在一起，它们同时指向同一内存空间，修改形参相当于修改实参。__引用相当于变量的别名__，理解这句话。
我们可能大致知道什么是按值访问，什么是按引用访问了

__那么，为什么基本类型是按值访问？而引用类型是按引用访问？__
我们知道引用类型不同于基本类型，它不是单纯的值，而是一种数据结构，包含有属性和方法。
如果按值传递，会拷贝整个对象，这样消耗的空间是否太大了呢，所以`ES`将这些对象的访问都设置成了按引用访问。

__刚才我们提到了，`c++`的值拷贝方式和引用拷贝方式，`JS`是否也是这样的呢？__
的确，`JS`也是这样的，下面简单的验证一下
```js
// 基本类型的拷贝
var a = "hello";
var b = a;
console.log(a+','+b); // hello,hello
```
```js
// 引用类型的拷贝
var obj = new Objcet();
var obj1 = obj2;
obj.name = "hello";
console.log(obj.name+','+obj1.name); // hello,hello
```
因此我们可以看到，对象`obj1`是随`obj`变化的，他们是指向同一个内存块的，而不是简单的拷贝而已

__那么传参是否也是在我们的意料之中呢？__
然而并不是这样的，ES 中所有函数的参数都是按值传递的，基本类型值的传递就像复制一样
而引用类型的值的传递，就行引用类型的变量复制一样，为什么会这样呢？
这里我们重点看一下传递引用参数
```js
function setName(obj) {
    obj.name = "suo";
}
var person = new Object;
setName(person);
console.log(person.name);   // suo
```
由上可知，对象传递后，通过函数修改对象属性影响了全局变量的值
在这里有人可能会觉得，函数传的是引用并非是值吧
其实 ES 中所有的函数都是按值传递，并非引用传递，有人肯定会有疑问为什么`obj`会被改变呢
其实这里说的按值传递有点特殊，这个值恰好是指向内存的指针，与引用传递的不同在于，并没有，形参并没有和实参进行引用绑定
接下来进一步验证一下
```js
function setName(obj) {
    obj.name = "suo";
    obj = new Object;   
    obj.name = "yue";
}
var person = new Object;
setName(person);
console.log(person.name);   // suo
```
上面代码的意义是什么呢？我们在函数内重新定义了这个对象，并给了它一个属性
而执行后，结果`person`并没有和`obj`的属性一样，也就是说`obj`并不是`person`的引用，仅仅是它的地址的值等于`person`的地址而已

我们已经知道了`ES`有引用类型和基本类型，而且`typeof`操作符可以检测基本类型的类型
__那么我们怎样检测引用类型呢？__其实也有个操作符可以利用，不过它只能判断是或者不是，不能直接得出是什么类型
下面我们测试一下几个引用类型检测
```js
var person = new Object;
console.log(person instanceof Object);
```

刚刚看到了，函数通过传值的形式来传参
__现在思考一下，函数是怎么执行的呢？__
原来每个函数都有执行环境，每个执行环境都有与之对应的变量对象，环境中所有的对象和函数都保存在这个对象中。
代码在一个环境中执行时，会创建变量对象的作用域链，作用域链控制执行环境访问变量和函数的顺序和权限。
下面先看一段代码
```js
var color = "red";
function changeColor() {
    if (color === "red")
        color = "yellow";
    else
        color = "green";
}
changeColor();
console.log(color); // yellow
```
可知，函数执行时可以访问到外部的变量，这里的作用域包括两个对象：自己的对象，全局环境的对象
__进一步研究，多层函数套用，执行是怎样的呢？__
```js
var color = "blue";
function changeColor() {
    var anotherColor = "red";
    function swapColors() {
        var tempColor = anotherColor;
        anotherColor = color;
        color = tempColor; // 都能访问
    } // 只能访问color,anothor
    swapColor();
} // 只能访问color
changeColor();
```
可以看出作用域链是由内而外的，也就是说函数是从当前的环境对象向外搜索变量，直至到全局对象。函数可以访问外部的变量，但是无法访问当前环境下函数里的局部变量。
用更确切的话说，__ES 的作用域链是单向的，而这个方向是由里向外的访问原则__。
下面画一个图式来说明这个问题
![img](/images/dm2.png)
现在问题来了，既然作用域是单向访问，有没有办法访问到由外访问到里面的局部变量呢？
答案是有的，下节谈一下闭包，这个特性能很好的解决这个问题。
还有一点要说明的是,__JS 没有块级作用域__，不同域 c 语言，花括号括起来的都有作用域，__JS 只有函数作用域__
下面用代码解释下
```c
#include <stdio.h>
int main(void) {
    {
        int b = 0;
    }
    for (int a=0; a<10; a++) {
        b++; // 出错
    }
    printf("%d,%d", &a, &b);
    return 0;
}
// error: 'b' was not declared in this scope,b++;
// error: 'a' was not declared in this scope
```

```js
{
    var b = 0;
}
for (var a=0; a<10; a++) {
    b++;
}
console.log(a+','+b);
// 10,10
```
根据上面的运行结果，我们很容易看到 C 中花括号的是单独的作用域，其他作用域是访问不到的，`for`循环里的变量，循环结束后，也不会保留
而 JS 里的不一样，他们虽然有花括号，但是这个花括号如同形式一般，没特别的意义，与去掉花括号是同样的意义。 JS 是通过函数作用域，来实现作用域链，控制访问顺序和权限。所有它的最小执行的单元就是函数。