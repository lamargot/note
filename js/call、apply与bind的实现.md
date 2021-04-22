## call
call() 方法调用一个函数, 其具有一个指定的this值和一些分别传入的参数

模拟的步骤可以分为：

	1. 将函数设为对象的属性
	2. 执行该函数
	3. 删除该函数

```javascript
Function.prototype.call=function (context) {
  context=context||window;
  context.fn=this;
  let arr=[];
  for(let i=1,len=arguments.length;i<len;i++){
    arr.push('arguments['+i+']');
  }
  let result=eval('context.fn('+arr+')')
  delete context.fn;
  return result;
}

```

### apply
apply方法调用一个函数, 其具有一个指定的this值和一个数组参数

模拟的步骤可以分为：

	1. 将函数设为对象的属性
	2. 执行该函数
	3. 删除该函数

```javascript
Function.prototype.apply = function (context, arr) {
    var context = Object(context) || window;
    context.fn = this;
    let result;
    if (!arr) {
        result = context.fn();
    }
    else {
        var args = [];
        for (var i = 0, len = arr.length; i < len; i++) {
            args.push('arr[' + i + ']');
        }
        result = eval('context.fn(' + args + ')')
    }
    delete context.fn
    return result;
}
```

### bind
bind() 方法会创建一个新函数。当这个新函数被调用时，bind() 的第一个参数将作为它运行时的 this，之后的一系列参数将会在传递的实参前传入作为它的参数。

 bind 函数的两个特点：
	1. 可以传入参数
	2. 返回一个函数

```javascript
Function.prototype.bind2 = function (context) {
  let self = this;
  // 获取bind2函数从第二个参数到最后一个参数
  var args = Array.prototype.slice.call(arguments, 1);//slice从已有的数组中返回选定的元素，规定从何处开始选取
  var fNOP = function () {};
  var fbound = function () {
    // 这个时候的arguments是bind返回的函数传入的参数
      var bindArgs = Array.prototype.slice.call(arguments);
      return self.apply(this instanceof self ? this : context, args.concat(bindArgs));//concat连接两个数组 
  }
  fNOP.prototype = this.prototype;
  fbound.prototype = new fNOP();
  return fbound;//返回一个函数
}
```

## 总结

	* 三者都是用来改变函数的this指向
	* 三者的第一个参数都是this指向的对象
	* bind返回一个绑定函数可稍后执行，call、apply是立即调用
	* 三者都可以给定参数传递
	* call、bind给定参数需要将参数全部列出，apply给定参数数组

