# let、const和var的区别及性质

标签： es6

---



### let、const和var声明部分：声明次数、声明规则、声明提升、块级作用域、在全局声明



### 定义：

#### 声明次数

let、const只能声明一次，var可以多次声明。

```JavaScript
var a;
var a;
// 正常

let b;
let b;
// Uncaught SyntaxError: Identifier 'b' has already been declared

var c ;
let c ; 
// Uncaught SyntaxError: Identifier 'c' has already been declared

const d = 0;
const d = 0;
// Uncaught SyntaxError: Identifier 'd' has already been declared

const d;
// Uncaught SyntaxError: Missing initializer in const declaration

```


#### 关于const的声明和规则：

1. const声明变量的时候必须赋值，否则会报错（可见上面的例子）。

2. const声明一个基本类型的变量，则不能改变它的值，否则会报错。

3. const声明一个引用类型的变量，则不能改变它的内存地址，否则会报错。

   const声明引用类型，限制的是const声明的变量对于对象类型的指向关系，所以之后都不能改变变量的指向关系；但是并没有限定对象类型本身，所以对象类型里的内容是可以改变的。

```JavaScript
const obj = {a: 1}

obj.b = 2;   //成功添加

obj = {c: 3} // Uncaught TypeError: Assignment to constant variable.
```

解释这个例子：变量obj指向变量，指向关系改变就会报错，对象里的内容改变没有问题。

所以对于引用类型，限制的只是这个指向关系。





### 性质：

#### 声明提升

let 和 const 会被提升,但不会被初始化，
var会被提升,会被初始化。

具体过程：
let/const也存在变量声明提升，只是没有初始化分配内存。
一个变量有三个操作，声明(提到作用域顶部)，初始化(赋默认值)，赋值(继续赋值)。
var 是一开始变量声明提升，然后初始化成undefined，代码执行到那行的时候赋值。
let 是一开始变量声明提升，然后没有初始化分配内存，代码执行到那行初始化，之后对变量继续操作是赋值。因为没有初始化分配内存，你找它就找不到，所以会报错，这是暂时性死区。
const 是只有声明和初始化，没有赋值操作，所以不可变。

一个很好理解的说法：

临时死区：

这是因为 JavaScript 引擎在扫描代码发现变量声明时，要么将它们提升到作用域顶部(遇到 var 声明)，要么将声明放在 TDZ 中(遇到 let 和 const 声明)。访问 TDZ 中的变量会触发运行时错误。只有执行过变量声明语句后，变量才会从 TDZ 中移出，然后方可访问。





##  let、const和var在循环中的区别（块级作用域）

### for循环

- var  是只有一个块级作用域。

```JavaScript
for(var i=0;i<10;i++){
	...
}
console.log(i);  //10
```

在十次循环中var i=0只会执行一次，后面两条语句每次都会执行，块级作用域始终都是这一个。



- let/const  每次循环都会声明一次（对比var声明的for循环只会声明一次），每次都会创建一个块级作用域，还会在每次重新创建的的每个块级作用域中重新赋值。

```JavaScript
for(let i=0;i<10;i++){
	...
}
console.log(i);  //Uncaught ReferenceError: i is not defined
```

let/const  每次循环的内部：

```JavaScript
{
    let i=0;
    ...
}
{
    let i=1;  //每次都会创建一个新的作用域块，且记住上次的循环结果并赋值给下一个循环
    ...
}
{
    let i=2;
    ...
}
...
```

### for in循环

for in 循环很简单，在这里简单提一下：

```
var funcs = [], object = {a: 1, b: 1, c: 1};
for (var key in object) {
    funcs.push(function(){
        console.log(key)
    });
}

funcs[0]()
```

结果是 'c';

那如果把 var 改成 let 或者 const 呢？

使用 let，结果自然会是 'a'，const 呢？ 报错还是 'a'?

结果是正确打印 'a'，这是因为在 for in 循环中，每次迭代不会修改已有的绑定，而是会创建一个新的绑定。

tip：for in循环in之前的变量（本题中是key）表示：
在数组中，表示从首地址arr[0]、arr[1]...递增

在对象中，表示从第1个属性名、第2个属性名...递增

以上是一般规律，但是实际上： for-in 遍历属性的顺序并不确定，即输出的结果顺序与属性在对象中的顺序无关，也与属性的字母顺序无关，与其他任何顺序也无关。





#### 在全局声明



这使我们能放心的使用新语法，不用担心污染全局的window对象。

在全局作用域中用`var`来声明，该变量因为变量提升到全局，变成window`的属性，即使`window`上已经存在原生属性，也会被`var给覆盖掉。

而`let、const`在全局声明时则不会替换掉`window`上的属性。

```JavaScript
var a=0;
console.log(window.a); //0

let b=1;
console.log(window.b); //undefined
```

这个其实也是let/const的特点，ES6规定它们不属于顶层全局变量的属性.

![let、const在全局中的存储位置](https://github.com/bigengbuchuo/doc/blob/master/fe/pic/window.png)

可以看到使用let声明的变量s是在一个叫script作用域下的，

而var声明的变量因为变量提升所以提升到了全局变量window对象中，

这使我们能放心的使用新语法，不用担心污染全局的window对象。

<br>
<br>

总结：在我们开发的时候，可能认为应该默认使用 let 而不是 var ，这种情况下，对于需要写保护的变量要使用 const。然而另一种做法日益普及：默认使用 const，只有当确实需要改变变量的值的时候才使用 let。这是因为大部分的变量的值在初始化后不应再改变，而预料之外的变量之的改变是很多 bug 的源头。



<br>
<br>
<br>

补充本文的一些小知识点：

①数据类型的分类：

|数据类型        | 包括   |
| --------   | :  |
| 基本类型     | undefined、null、number、boolean、string等 |
| 引用类型    |  Object、array、function等    |

1.js的数据类型分为两种：原始类型（即基本数据类型）和对象类型（即引用数据类型）；

2.js常用的基本数据类型包括undefined、null、number、boolean、string；

3.js的引用数据类型也就是对象类型Object，比如：Object、array、function等；



②let、const 对临时死区和块级作用域结合例题的测试（很简单，熟悉的可以跳过）

---

原题：

```JavaScript
var value = "global";

// 例子1
(function() {
    console.log(value);

    let value = 'local';
}());

// 例子2
{
    console.log(value);

    const value = 'local';
};

```

两个例子中，结果并不会打印 "global"，而是报错 `Uncaught ReferenceError: value is not defined`，就是因为 TDZ 的缘故。

原题结束。

---

分析：因为块级作用域的原因，立即执行函数内的value值只取决于函数块内的声明，而let虽然变量提升，但是不初始化，所以会报错。

验证：

1.
```JavaScript
var value = "global";

(function() {
 
   let value = 'local';
  
 console.log(value);

}());

结果：
Local 
undefined  返回值
```
 声明放在前面就不报错，而且打出的值是local，说明确实是块级作用域的作用。
而整个过程并没有返回值，所以是undefined。

2.
```JavaScript
var value = "global";

(function() {
 
   let value = 'local';
  
 console.log(value);

}());

console.log(value);

结果：
local
global
undefined  返回值

```
在全局打印value，仍然可以打印出global，说明全局的作用域与块级作用域确实是分开的。

