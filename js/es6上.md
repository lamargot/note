### let/const

JS中作用域有：全局作用域、函数作用域。没有块作用域的概念。ECMAScript 6(简称ES6)中新增了块级作用域。
块作用域由 { } 包括，if语句和for语句里面的{ }也属于块作用域。

let,const用于声明变量，用来替代老语法的var关键字，与var不同的是，let/const会创建一个块级作用域， ES6规定它们不属于顶层全局变量的属性。

##### var关键字的缺陷：

1.变量提升（ 变量可以在声明之前使用，值为undefined）
2.污染全局变量{ }等( 只是可以跨块级作用域，通过var定义的变量不能跨函数作用域访问到)

```javascript
{
        var a = 1;
        console.log(a); // 1
    }
    console.log(a); // 1
    // 可见，通过var定义的变量可以跨块作用域访问到。


    (function A() {
        var b = 2;
        console.log(b); // 2
    })();
    // console.log(b); // 报错，
    // 可见，通过var定义的变量不能跨函数作用域访问到
```
![image](https://user-images.githubusercontent.com/55495739/116400475-a8f4b880-a85c-11eb-9e02-d260a9fce5a7.png)

暂时性死区
使用let/const声明的变量，从一开始就形成了封闭作用域，在声明变量之前是无法使用这个变量的，这个特点也是为了弥补var的缺陷（var声明的变量有变量提升）

##### const

使用const关键字声明一个常量，常量的意思是不会改变的变量，const和let的一些区别是

	1. const声明变量的时候必须赋值，否则会报错，同样使用const声明的变量被修改了也会报错
	2. const声明变量不能改变，如果声明的是一个引用类型，则不能改变它的内存地址


### 箭头函数

ES6 允许使用箭头（=>）定义函数

箭头函数对于使用function关键字创建的函数有以下区别

	1. 箭头函数没有arguments（建议使用更好的语法，剩余运算符替代）
	2. 箭头函数没有prototype属性，不能用作构造函数（不能用new关键字调用）
	3. 箭头函数没有自己this，它的this是词法的，引用的是上下文的this，即在你写这行代码的时候就箭头函数的this就已经和外层执行上下文的this绑定了(这里个人认为并不代表完全是静态的,因为外层的上下文仍是动态的可以使用call,apply,bind修改,这里只是说明了箭头函数的this始终等于它上层上下文中的this)  这个特性很有利于封装回调函数


箭头函数的this指向即使使用call,apply,bind也无法改变

##### 不能使用箭头函数的场景

1.定义对象方法
　　 JS 中对象方法的定义方式是在对象上定义一个指向函数的属性，当方法被调用的时候，方法内的 this 要求指向方法所属的对象。
```javascript
   let obj = {
    sum: () => {
        console.log(this === window); // true
        return this.array.reduce((result, item) => result + item);
    }
};
obj.sum();//报错
```

2. 需要动态this的时候

```javascript

var button = document.getElementById('press');
button.addEventListener('click', () => {
  this.classList.toggle('on');});
```
上面代码运行时，点击按钮会报错，因为button的监听函数是一个箭头函数，导致里面的this就是全局对象。如果改成普通函数，this就会动态指向被点击的按钮对象。
