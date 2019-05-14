### 为什么有迭代器（iterator）？
为了解决循环语句存在的问题。
我们在js中写过这样的代码：
```JavaScript
var colors=["red","green","blue"];
for(var i=0,len=colors.length;i<len;i++){
  console.log(colors[i]);
}
//red green blue
```
上面是一段标准的for循环代码，通过变量i来追踪数组的索引。这个循环现在看起来比较简单，但是如果存在多个循环嵌套，代码的复杂度会大大增加，使用过程中可能会错误地使用其他for循环的跟踪变量，从而导致程序出错。
迭代器的出现旨在消除这种复杂性并减少循环中的错误。

### 什么是迭代器？
迭代器是一种特殊对象，它具有一些专门为迭代过程设计的专有接口，所有的迭代器对象都有一个next()方法，每次调用都返回一个结果对象。结果对象有两个属性：
- value：表示下一个将要返回的值
- done：表示当前数据是不是最后一个。是一个布尔类型的值，默认值为false，当没有更多可返回数据时返回true。
 
迭代器还包含一个内部指针，用来指向当前集合中值的位置，每调用一次next()方法，都返回下一个可用的值。

关于next()方法:第一次调用指针对象的next方法，可以将指针指向数据结构的第一个成员，第二次指向第二个，以此类推。

```JavaScript
function createIterator(items){
  var i=0;
  return{
    next:function(){
      var done=(i>=items.length);
      var value=!done?items[i++]:undefined;
      return{
        done:done,
        value:value
      };     //返回结果对象
    }
  };
}
var iterator=createIterator([1,2,3]);

console.log(iterator.next());  //"{value:1,done:false}"
console.log(iterator.next());  //"{value:2,done:false}"
console.log(iterator.next());  //"{value:3,done:false}"
console.log(iterator.next());  //"{value:undefined,done:true}"

//之后调用都会返回相同的内容
console.log(iterator.next());  //"{value:undefined,done:true}"
```

这个例子解释了上面的几个概念。例子很简单，但是迭代器的写法还稍微有点复杂。es6还引入了一个生成器（Generator）对象，它可以让创建迭代器对象的过程变得更简单。


### 什么是生成器（Generator）？
生成器是一种返回迭代器的函数，通过function关键字后的星号（*）来表示，函数中会用到关键字yield。


先看一个简单的例子：

```JavaScript
function *createIterator(a){
    yield 1;
    yield 2;
    yield 3;
}
let iterator=createIterator();  //iterator接收了生成好的迭代器

console.log(iterator.next().value);  //1
console.log(iterator.next().value);  //2
console.log(iterator.next().value);  //3
```

在这个例子中，createIterator()前的星号表明它是一个生成器。yield是es6里一个新的关键字，可以指定调用迭代器的next()方法的返回值及返回顺序。生成初始迭代器后，连续三次调用next()方法返回3个不同的值，分别是1、2、3，最终返回创建（赋值）好的迭代器。

生成器最特殊的部分在于，每当执行完一条yield语句后，函数就会自动停止。上面的代码中，第一次调用next()方法进入函数执行yield 1，执行完之后会自动停止到那。直到第二次调用next()，才会继续执行yield 2。

使用yield关键字可以返回任何值或表达式，所以可以通过生成器函数批量地给迭代器添加元素。
例如：

```JavaScript
function *createIterator(a){
    for(let i=0;i<a.length;i++){
        yield a[i];
    }
}
let iterator=createIterator([1,2,3]);

console.log(iterator.next());  //"{value:1,done:false}"
console.log(iterator.next());  //"{value:2,done:false}"
console.log(iterator.next());  //"{value:3,done:false}"
console.log(iterator.next());  //"{value:undefined,done:true}"

//之后所有调用都会返回相同内容
console.log(iterator.next());  //"{value:undefined,done:true}"
```

不能用箭头函数来创建生成器。

生成器本身是一个函数，所以也可以作为对象的方法。
```JavaScript
let o={
    createIterator: function *(a){
        for(let i=0;i<a.length;i++){
            yield a[i];
        }
    }
};
let iterator=o.createIterator([1,2,3]);
```


在本节一开始，我们的主要目的是**解决追踪内部循环索引复杂**的问题，所以我们引入了迭代器，（又因为原生迭代器写法复杂所以引入了生成器），其实，要解决一开始的问题，需要两个工具：一个是**迭代器**，另一个是**for-of循环**。

说到循环，肯定是用于可迭代对象的。

### 可迭代对象和for-of循环
可迭代对象和for-of循环通过什么关联在一起呢？<br>
答案是Symbol.iterator属性。

可迭代对象具有Symbol.iterator属性，Symbol.iterator对象可以通过特定函数返回一个作用于附属对象的迭代器。
es6中，所有的集合对象（数组、Set集合、Map集合）和字符串都是可迭代对象，都有默认的迭代器。

说完迭代器来说for-of循环，for-of循环每执行一次都会调用可迭代对象的next()方法，返回的结果对象的value属性存储在‘ for 变量 of ’循环的变量里。循环将持续这一过程直到返回对象的done属性的值为true。

```JavaScript
let a=[1,2,3];
for(let num of a){
    console.log(num);
}
//1 2 3
```
这段for-of循环的代码通过调用a数组的Symbol.iterator方法来获取迭代器。这一过程是在js引擎背后完成的。随后迭代器的next()方法被多次调用，从其返回对象的value属性独取值并存储在变量的num中，依次为1，2，3，当结果对象的done属性值为true时循环推出，所以num不会被赋值为undefined。

for-of循环相比于传统的for循环，其控制条件更简单，不需要追踪复杂的索引，所以更不容易出错。

如果将for-of循环在执行前会检查目标对象中是否存在默认的函数类型迭代器。所以当for-of循环用于不可迭代对象、null、undefined时程序将会抛出错误。

那么我们来介绍默认的函数类型迭代器。

### 访问默认迭代器
可以通过Symbol.iterator来访问对象默认的迭代器。

```JavaScript
let a=[1,2,3];
let iterator=a[Symbol.iterator]();
console.log(iterator.next());  // {value: 1, done: false}
console.log(iterator.next());  // {value: 2, done: false}
console.log(iterator.next());  // {value: 3, done: false}
console.log(iterator.next());  // {value: undefined, done: true}
```
在这段代码中，通过Symbol.iterator获取了a数组的默认迭代器。在js引擎中执行for-of循环语句时也会有这样的隐式操作。

因为具有Symbol.iterator属性的对象都有默认的函数类型的迭代器，所以可以用它来检测对象是否为可迭代对象。

```JavaScript
function a(object){
    return typeof object[Symbol.iterator] === 'function';
}
console.log(a([1,2,3]));  // true
console.log(a("hello"));  // true
console.log(a(new Map()));  // true
console.log(a(new Set()));  // true
console.log(a(new WeakMap()));  // false
console.log(a(new WeakSet()));  // false
```
这里的a函数可以检查指定对象中是否存在默认的函数类型的迭代器，而for-of循环在执行前也会做相似的检查。

### 创建可迭代对象
默认情况下，开发者定义的对象都是不可迭代对象，但如果给Symbol.iterator属性添加一个生成器，则可以将其变为可迭代对象。
```JavaScript
let a={
    *[Symbol.iterator](){
        yield 1;
        yield 2;
        yield 3;

    }
}
for(let x of a){
    console.log(x);
};
```

现在我们已经了解了默认的数组迭代器，而在es6中还有很多内建迭代器可以简化集合数据的操作。

### 内建迭代器
内建迭代器是es6为各种内建类型提供的迭代器，基本可以满足我们的需求，只有这些内建迭代器无法实现目标时才需要自己创建。
下面介绍三个内建迭代器：
- 集合对象迭代器
- 字符串迭代器
- NodeList迭代器

#### 集合对象迭代器
es6中的三种类型的集合对象分别是：数组、Map集合、Set集合，这三种对象都内建了以下三种迭代器：
- entries() 返回一个迭代器，其值为多个键值对。
- values()  返回一个迭代器，其值为集合的值。
- keys()    返回一个迭代器，其值为集合中所有键值。

##### entries()迭代器
先来看几个例子：
```JavaScript
let a=["red","green","blue"];

let b=new Set([1,2,3]);

let c=new Map();
c.set("title","English");
c.set("edition","2019");

for(let d of a.entries()){
    console.log(d);
}
for(let d of b.entries()){
    console.log(d);
}
for(let d of c.entries()){
    console.log(d);
}

//[0, "red"]  类似于键值对
//[1, "green"]
//[2, "blue"]
//[1, 1]
//[2, 2]
//[3, 3]
//["title", "English"]
//["edition", "2019"]
```
for-of循环每执行一次都会调用可迭代对象的next()方法，每次调用next()方法时，entries()返回一个数组（数组本身就是一个迭代器）。<br>
如果被遍历的对象是<br>
数组：键值是数字类型的索引。<br>
Set集合：第一个元素和第二个元素都是值。<br>
Map集合：键名和值。<br>


##### values()迭代器
```JavaScript
let a=["red","green","blue"];

let b=new Set([1,2,3]);

let c=new Map();
c.set("title","English");
c.set("edition","2019");

for(let d of a.values()){
    console.log(d);
}
for(let d of b.values()){
    console.log(d);
}
for(let d of c.values()){
    console.log(d);
}

// red
// green
// blue
// 1
// 2
// 3
// English
// 2019
```


##### keys()迭代器
```JavaScript
let a=["red","green","blue"];

let b=new Set([1,2,3]);

let c=new Map();
c.set("title","English");
c.set("edition","2019");

for(let d of a.keys()){
    console.log(d);
}
for(let d of b.keys()){
    console.log(d);
}
for(let d of c.keys()){
    console.log(d);
}

// 0
// 1
// 2   返回的是a集合里的键
// 1
// 2
// 3   set集合键与值是相同的
// title
// edition   map集合返回每个独立的键
```

当未指定默认迭代器时，集合在for-of循环中会使用默认迭代器。**数组和Set集合的默认迭代器是values()方法，Map()集合的默认迭代器是entries()方法。而WeakSet集合和WeakMap集合没有内建的迭代器，更无从说起默认迭代器了。**



#### 字符串迭代器
字符串迭代器解决了双字节字符打印问题。
书163-164。


#### NodeList迭代器
es6中，NodeList类型也拥有了默认迭代器，行为与数组的默认迭代器完全一样。


### 高级迭代器功能
##### 给迭代器传递参数
如果给迭代器的next()方法传递参数，则这个参数的值就会替代生成器上一条yield语句的返回值。特殊的是，第一次调用next()方法时无论传入什么参数都会被丢弃。

##### 在迭代器中抛出错误
可以通过try-catch结合throw()，作用：抛出错误但继续执行。

##### 生成器返回语句
在生成器中添加return语句。
- return语句表示所有操作已经完成，属性done被设置为true，value仍是undefined。
- 在return语句中也可以指定一个返回值，该值被赋值给返回对象的value属性。
- 展开运算符和for-of语句会直接忽略通过return语句指定的任何返回值，只要done一变为true就立即停止读取其他的值。


##### 委托生成器
在某些情况下，我们需要将已有的两个迭代器合二为一，这时可以创建一个新的生成器，再给yield语句添加一个星号，就可以将生成数据的过程委托给其他生成器。








