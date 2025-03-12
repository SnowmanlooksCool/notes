
# 执行栈
执行栈是执行上下文存储和运行的环境 

# 执行上下文（Execution Context） 
是代码执行时的环境，它包含了代码执行所需的所有信息。执行上下文是在代码执行时动态创建的，在编译阶段（或称为解析阶段），JavaScript 引擎会进行一些准备工作，比如变量和函数的声明提升（hoisting），这些操作与执行上下文的创建密切相关。
执行上下文的类型

# JavaScript 中有三种主要的执行上下文：
## 全局执行上下文：
    这是默认的、最外层的执行上下文，不在任何函数内的代码都在全局执行上下文中执行。在浏览器中，全局对象是 window，在 Node.js 中是 global。
## 函数执行上下文：
    每当一个函数被调用时，就会为该函数创建一个新的执行上下文。
## Eval 执行上下文：
    在 eval 函数中执行的代码也会创建自己的执行上下文（不常用，且不推荐使用）。

# 执行上下文的内容
每个执行上下文都包含以下内容：
变量环境（Variable Environment）：存储变量和函数的声明（通过 var 声明的变量和函数声明）。在编译阶段，变量和函数声明会被提升（hoisted），但 var 声明的变量会被初始化为 undefined，而函数声明会被完全提升。
词法环境（Lexical Environment）：存储块级作用域的变量（通过 let 和 const 声明的变量）。词法环境是一个嵌套结构，用于支持作用域链。
this 绑定（This Binding）：确定当前执行上下文中的 this 值。在全局执行上下文中，this 指向全局对象（如 window 或 global）。
在函数执行上下文中，this 的值取决于函数的调用方式。
外部环境引用（Outer Environment Reference）：指向外部的词法环境，用于实现作用域链。通过作用域链，可以访问外部作用域中的变量和函数。

# 执行上下文的生命周期
创建阶段：
创建变量环境、词法环境、this 绑定和外部环境引用。
进行变量和函数的声明提升。
执行阶段：
逐行执行代码，为变量赋值，调用函数等。
示例
javascript
var a = 10;
let b = 20;

function foo() {
    var c = 30;
    console.log(a + b + c);
}

foo();
在全局执行上下文中，a 和 foo 被提升，a 初始化为 undefined，foo 是一个函数。
在 foo 的函数执行上下文中，c 被提升并初始化为 undefined。
执行阶段，a 被赋值为 10，b 被赋值为 20，c 被赋值为 30，然后执行 console.log(a + b + c)。


https://blog.poetries.top/browser-working-principle/guide/part2/lesson07.html#%E5%8F%98%E9%87%8F%E6%8F%90%E5%8D%87-hoisting

https://juejin.cn/post/7255720279934648375#heading-6


作用域链  词法作用域 代码编写时候就定义好的


事件循环机制
微任务是在第一次事件循环中就会执行的每次都会全部执行完毕 但是宏任务会在下一次事件循环中执行（因为js引入本身就是一个宏任务） 并且每次只会执行一个任务 
todo 最后还是要确定setTimeout 打印的值 