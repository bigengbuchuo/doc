在es5的五种原始类型基础上，es6引入了一种新的原始类型：Symbol。

起初，引入Symbol是为了表示对象中的非字符串属性名。在es6中，新标准将对象中的Symbol属性与其他属性分别分类。

### 创建Symbol
```JavaScript
let a = Symbol();

typeof a;  // "symbol"
```
由于Symbol是一个原始值，因此调用new Symbol()会导致程序抛出错误。

Symbol函数接收一个可选参数，来描述即将创建的Symbol，这段描述不可用于属性访问，而是在阅读代和在控制台显示或是转换为字符串的时候，比较易于区分。
```JavaScript
let s1 = Symbol('foo');
let s2 = Symbol('bar');

s1 // Symbol(foo)
s2 // Symbol(bar)

s1.toString() // "Symbol(foo)"
s2.toString() // "Symbol(bar)"
```
上面代码中，s1和s2是两个 Symbol 值。如果不加参数，它们在控制台的输出都是Symbol()，不利于区分。有了参数以后，就等于为它们加上了描述，输出的时候就能够分清。

如果 Symbol 的参数是一个对象，就会调用该对象的toString方法，将其转为字符串，然后才生成一个 Symbol 值。
```JavaScript
const obj = {
  toString() {
    return 'abc';
  }
};
const sym = Symbol(obj);
sym // Symbol(abc)
```
Symbol的描述被存储在内部的[[Description]]属性中，只有当调用Symbol的toString()方法时才可以读取这个属性。在调用console.log()时隐式的调用了toString()方法，所以才会被打印在日志中，但不能在代码里直接访问[[Description]]。

需要注意的是，每一个Symbol都是独一无二的，因此Symbol()函数的参数也只是对当前Symbol值的描述。

```JavaScript
// 没有参数的情况
let s1 = Symbol();
let s2 = Symbol();

s1 === s2 // false

// 有参数的情况
let s1 = Symbol('foo');
let s2 = Symbol('foo');

s1 === s2 // false
```

### Symbol的使用方法
- Symbol 值不能与其他类型的值进行运算，会报错。

```JavaScript
let sym = Symbol('My symbol');

"your symbol is " + sym
// TypeError: can't convert symbol to string
```
- 但是，Symbol 值可以显式转为字符串。

```JavaScript
let sym = Symbol('My symbol');

String(sym) // 'Symbol(My symbol)'
sym.toString() // 'Symbol(My symbol)'
```

- 另外，Symbol 值也可以转为布尔值，但是不能转为数值。

```JavaScript
let sym = Symbol();
Boolean(sym) // true
!sym  // false

if (sym) {
  // ...
}

Number(sym) // TypeError
sym + 2 // TypeError
```

### 作为属性名的Symbol
##### 怎样用Symbol()函数生成的变量作为属性名。
三种写法：

```JavaScript
let mySymbol = Symbol();

// 第一种写法
let a = {};
a[mySymbol] = 'Hello!';

// 第二种写法
let a = {
  [mySymbol]: 'Hello!'
};

// 第三种写法
let a = {};
Object.defineProperty(a, mySymbol, { value: 'Hello!' });

console.log(a[mySymbol] ); //"Hello!"
```

##### 需要注意的是，Symbol 值作为对象属性名时，不能用点运算符。

```JavaScript
const mySymbol = Symbol();
const a = {};

a.mySymbol = 'Hello!';
a[mySymbol] // undefined
a['mySymbol'] // "Hello!"
```
上面代码中，因为点运算符后面总是字符串，所以不会读取mySymbol作为标识名所指代的那个值，导致a的属性名实际上是一个字符串，而不是一个 Symbol 值。

##### 同理，在对象的内部，使用 Symbol 值定义属性时，Symbol 值必须放在方括号之中。

```JavaScript
let s = Symbol();

let obj = {
  [s]: function (arg) { ... }
};

obj[s](123);
```

 Symbol值还可以使用在switch函数当中，因为常量使用 Symbol值最大的好处，就是其他任何值都不可能有相同的值了，所以可以保证switch语句会按设计的方式工作。

```JavaScript
const a    = Symbol();
const b  = Symbol();

function getComplement(color) {
  switch (color) {
    case a:
      return b;
    case b:
      return a;
    default:
      throw new Error('Undefined color');
    }
}
```
还有一点需要注意，Symbol 值作为属性名时，该属性还是公开属性，不是私有属性。

##### Symbol 作为属性名的遍历
Symbol 作为属性名，该属性不会出现在for...in、for...of循环中（出现了也输出不了），也不会被Object.keys()、Object.getOwnPropertyNames()、JSON.stringify()返回。但是，它也不是私有属性，有一个Object.getOwnPropertySymbols方法，可以获取指定对象的所有 Symbol 属性名。

```JavaScript
let i = Symbol();
let obj={}
for (let i in obj) {
  console.log(i); // 无输出
}
```

因为无法方便地获取Symbol属性，为了在不同代码片段有效地共享这些Symbol，需要建立一个体系。

### Symbol共享体系
有时为了在不同代码片段有效地共享同一个Symbol。例如，在你的应用中有两种不同地对象类型，但是你希望它们使用同一个Symbol属性来表示一个独特的标识符。es6提供了一个可以随时访问的全局注册表。

与共享有关的俩个特性：

##### Symbol.for()方法
Symbol.for()创建一个可共享地Symbol，接收一个参数，同样用以描述。

```JavaScript
let uid =Symbol.for("uid");
let object={};

object[uid]="12345";

console.log(object[uid]);  //12345
console.log(uid);  //"Symbol(uid)"
```

Symbol.for()方法首先在全局Symbol注册表中搜索键为"uid"的Symbol是否存在，如果存在，直接返回已有的Symbol，否则创建一个新的Symbol，并使用这个键在Symbol全局注册表中注册，然后返回新创建的Symbol。

##### Symbol.keyFor()方法

```JavaScript
let uid =Symbol.for("uid");
console.log(Symbol.keyFor(uid));  //uid

let uid =Symbol.for("uid");
console.log(Symbol.keyFor(uid1));  //undefined
```

Symbol.keyFor()方法首先在全局Symbol注册表中搜索键为"uid"的Symbol是否存在，如果存在，直接返回已有的Symbol，否则，返回undefined。

这里再提一下Symbol()方法。
Symbol()方法不属于共享体系，不会在全局注册表登记，而是每次创建都创建一个新的Symbol。

其实可以这样说，Symbol全局注册表是一个类似全局作用域的共享环境。
