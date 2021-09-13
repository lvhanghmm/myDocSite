# JavaScript高级程序设计第四版

## this 对象

10.14.1 this 对象
在闭包中使用 this 会让代码变复杂。如果内部函数没有使用箭头函数定义，则 this 对象会在运
行时绑定到执行函数的上下文。如果在全局函数中调用，则 this 在非严格模式下等于 window，在严
格模式下等于 undefined。如果作为某个对象的方法调用，则 this 等于这个对象。匿名函数在这种情
况下不会绑定到某个对象，这就意味着 this 会指向 window，除非在严格模式下 this 是 undefined。
不过，由于闭包的写法所致，这个事实有时候没有那么容易看出来。来看下面的例子：
window.identity = 'The Window';
let object = {
identity: 'My Object',
getIdentityFunc() {
return function() {
return this.identity;
};
}
};
console.log(object.getIdentityFunc()()); // 'The Window'
这里先创建了一个全局变量 identity，之后又创建一个包含 identity 属性的对象。这个对象还
包含一个 getIdentityFunc()方法，返回一个匿名函数。这个匿名函数返回 this.identity。因为
getIdentityFunc()返回函数，所以 object.getIdentityFunc()()会立即调用这个返回的函数，
从而得到一个字符串。可是，此时返回的字符串是"The Winodw"，即全局变量 identity 的值。为什
10.14 闭包 313
8
1
2
3
4
5
14
6
7
9
10
11
13
12
么匿名函数没有使用其包含作用域（ getIdentityFunc()）的 this 对象呢？
前面介绍过，每个函数在被调用时都会自动创建两个特殊变量： this 和 arguments。内部函数永
远不可能直接访问外部函数的这两个变量。但是，如果把 this 保存到闭包可以访问的另一个变量中，
则是行得通的。比如：
window.identity = 'The Window';
let object = {
identity: 'My Object',
getIdentityFunc() {
let that = this;
return function() {
return that.identity;
};
}
};
console.log(object.getIdentityFunc()()); // 'My Object'
这里加粗的代码展示了与前面那个例子的区别。在定义匿名函数之前，先把外部函数的 this 保存
到变量 that 中。然后在定义闭包时，就可以让它访问 that，因为这是包含函数中名称没有任何冲突的
一个变量。即使在外部函数返回之后， that 仍然指向 object， 所以调用 object.getIdentityFunc()()
就会返回"My Object"。
注意 this 和 arguments 都是不能直接在内部函数中访问的。如果想访问包含作用域中
的 arguments 对象，则同样需要将其引用先保存到闭包能访问的另一个变量中。
在一些特殊情况下， this 值可能并不是我们所期待的值。比如下面这个修改后的例子：
window.identity = 'The Window';
let object = {
identity: 'My Object',
getIdentity () {
return this.identity;
}
};
getIdentity()方法就是返回 this.identity 的值。以下是几种调用 object.getIdentity()
的方式及返回值：
object.getIdentity(); // 'My Object'
(object.getIdentity)(); // 'My Object'
(object.getIdentity = object.getIdentity)(); // 'The Window'
第一行调用 object.getIdentity()是正常调用，会返回"My Object"，因为 this.identity
就是 object.identity。第二行在调用时把 object.getIdentity 放在了括号里。虽然加了括号之
后看起来是对一个函数的引用，但 this 值并没有变。这是因为按照规范， object.getIdentity 和
(object.getIdentity)是相等的。第三行执行了一次赋值，然后再调用赋值后的结果。因为赋值表
达式的值是函数本身， this 值不再与任何对象绑定，所以返回的是"The Window"。
一般情况下，不大可能像第二行和第三行这样调用对象上的方法。但通过这个例子，我们可以知道，
即使语法稍有不同，也可能影响 this 的值。
314 第 10 章 函 数
10.14.2 内存泄漏
由于 IE 在 IE9 之前对 JScript 对象和 COM 对象使用了不同的垃圾回收机制（第 4 章讨论过），所以
闭包在这些旧版本 IE 中可能会导致问题。在这些版本的 IE 中，把 HTML 元素保存在某个闭包的作用域
中，就相当于宣布该元素不能被销毁。来看下面的例子：
function assignHandler() {
let element = document.getElementById('someElement');
element.onclick = () => console.log(element.id);
}
以上代码创建了一个闭包，即 element 元素的事件处理程序（事件处理程序将在第 13 章讨论）。
而这个处理程序又创建了一个循环引用。匿名函数引用着 assignHandler()的活动对象，阻止了对
element 的引用计数归零。只要这个匿名函数存在， element 的引用计数就至少等于 1。也就是说，
内存不会被回收。其实只要这个例子稍加修改，就可以避免这种情况，比如：
function assignHandler() {
let element = document.getElementById('someElement');
let id = element.id;
element.onclick = () => console.log(id);
element = null;
}
在这个修改后的版本中，闭包改为引用一个保存着 element.id 的变量 id，从而消除了循环引用。
不过，光有这一步还不足以解决内存问题。因为闭包还是会引用包含函数的活动对象，而其中包含
element。即使闭包没有直接引用 element，包含函数的活动对象上还是保存着对它的引用。因此，必
须再把 element 设置为 null。这样就解除了对这个 COM 对象的引用，其引用计数也会减少，从而确
保其内存可以在适当的时候被回收。
10.15 立即调用的函数表达式
立即调用的匿名函数又被称作立即调用的函数表达式（ IIFE， Immediately Invoked Function
Expression）。它类似于函数声明，但由于被包含在括号中，所以会被解释为函数表达式。紧跟在第一组
括号后面的第二组括号会立即调用前面的函数表达式。下面是一个简单的例子：
(function() {
// 块级作用域
})();
使用 IIFE 可以模拟块级作用域，即在一个函数表达式内部声明变量，然后立即调用这个函数。这
样位于函数体作用域的变量就像是在块级作用域中一样。 ECMAScript 5 尚未支持块级作用域， 使用 IIFE
模拟块级作用域是相当普遍的。比如下面的例子：
// IIFE
(function () {
for (var i = 0; i < count; i++) {
console.log(i);
}
})();
console.log(i); // 抛出错误
图灵社区会员 aSINKz(1561821892@qq.com) 专享 尊重版权
10.15 立即调用的函数表达式 315
8
1
2
3
4
5
14
6
7
9
10
11
13
12
前面的代码在执行到 IIFE 外部的 console.log()时会出错，因为它访问的变量是在 IIFE 内部定义
的，在外部访问不到。在 ECMAScript 5.1 及以前，为了防止变量定义外泄， IIFE 是个非常有效的方式。
这样也不会导致闭包相关的内存问题，因为不存在对这个匿名函数的引用。为此，只要函数执行完毕，
其作用域链就可以被销毁。
在 ECMAScript 6 以后， IIFE 就没有那么必要了，因为块级作用域中的变量无须 IIFE 就可以实现同
样的隔离。下面展示了两种不同的块级作用域形式：
// 内嵌块级作用域
{
let i;
for (i = 0; i < count; i++) {
console.log(i);
}
}
console.log(i); // 抛出错误
// 循环的块级作用域
for (let i = 0; i < count; i++) {
console.log(i);
}
console.log(i); // 抛出错误
说明 IIFE 用途的一个实际的例子，就是可以用它锁定参数值。比如：
let divs = document.querySelectorAll('div');
// 达不到目的！
for (var i = 0; i < divs.length; ++i) {
divs[i].addEventListener('click', function() {
console.log(i);
});
}
这里使用 var 关键字声明了循环迭代变量 i，但这个变量并不会被限制在 for 循环的块级作用域内。
因此，渲染到页面上之后，点击每个<div>都会弹出元素总数。这是因为在执行单击处理程序时，迭代变
量的值是循环结束时的最终值，即元素的个数。而且，这个变量 i 存在于循环体外部，随时可以访问。
以前，为了实现点击第几个<div>就显示相应的索引值，需要借助 IIFE 来执行一个函数表达式，传
入每次循环的当前索引，从而“锁定”点击时应该显示的索引值：
let divs = document.querySelectorAll('div');
for (var i = 0; i < divs.length; ++i) {
divs[i].addEventListener('click', (function(frozenCounter) {
return function() {
console.log(frozenCounter);
};
})(i));
}
而使用 ECMAScript 块级作用域变量，就不用这么大动干戈了：
let divs = document.querySelectorAll('div');
for (let i = 0; i < divs.length; ++i) {
divs[i].addEventListener('click', function() {
316 第 10 章 函 数
console.log(i);
});
}
这样就可以让每次点击都显示正确的索引了。这里，事件处理程序执行时就会引用 for 循环块级作
用域中的索引值。这是因为在 ECMAScript 6 中，如果对 for 循环使用块级作用域变量关键字，在这里
就是 let，那么循环就会为每个循环创建独立的变量，从而让每个单击处理程序都能引用特定的索引。
但要注意，如果把变量声明拿到 for 循环外部，那就不行了。下面这种写法会碰到跟在循环中使用
var i = 0 同样的问题：
let divs = document.querySelectorAll('div');
// 达不到目的！
let i;
for (i = 0; i < divs.length; ++i) {
divs[i].addEventListener('click', function() {
console.log(i);
});
}  