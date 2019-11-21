# JavaScript 变量

## var

ES5 使用 `var` 关键字来定义变量

```js
var foo = 1234;
```

未初始化的变量，默认值为 `undefined`

```js
var foo;  // undefined
```

变量是松散型的，只充当占位符的作用，与初始化的类型无关

```js
var foo = "hello";
foo = [1, 2, 3];
```

变量先使用后定义不会报错，会发生变量提升的情况

```js
console.log(foo); // 输出undefined
var foo = 2;
```

## let

ES6 新增 `let` 关键字，所声明的变量只在代码块中生效

```js
{
  let a = 10;
  var b = 1;
}

a // ReferenceError: a is not defined.
b // 1
```

未初始化的变量，默认值为 `undefined`

```js
let foo;  // undefined
```

`let` 的典型应用场景 - 循环计数器，避免了循环参数的污染全局

```js
for (let i = 0; i < 10; i++) {
	// 注意，循环内部又是一个单独的子作用域
	let i = 'hello'; 
}
```

`let` 不存在变量提升

```js
console.log(bar); // 报错ReferenceError
let bar = 2;
```

`let` 不允许在相同作用域内，重复声明变量

```js
function func() {
  let a = 10;
  let a = 1;
  var a = 2;
}
```

## const

`const` 声明一个只读常量，一旦声明，无法修改值

```js
const PI = 3.1415;
```

`const` 与 `let` 命令一样，只在块级作用域内生效

```js
if (true) {
  const MAX = 5;
}

MAX // Uncaught ReferenceError: MAX is not defined
```

`const` 的本质是使变量存储的地址不可变，而非变量的值不可变，这点在引用类型上就可以看出来

```js
const a = [];
a.push('Hello'); // 可执行
a.length = 0;    // 可执行
a = ['Dave'];    // 报错
```

## 顶层对象

ES5 中，顶层对象与全局变量一样

```js
a = 2;  // 全局变量
window.a // 顶层对象
```

ES6 规定，`let` 和 `const` 命令声明的全局变阿玲不属于顶层对象的属性

```js
var a = 1;
window.a // 1

let b = 1;
window.b // undefined
```

