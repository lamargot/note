模块化的开发方式可以提高代码复用率，方便进行代码的管理。通常一个文件就是一个模块，有自己的作用域，只向外暴露特定的变量和函数。目前流行的js模块化规范有CommonJS、AMD、CMD以及ES6的模块系统。

### 一、CommonJS
CommonJS是一个看不见摸不着的东西，他只是一个规范！就像校纪校规一样，用来规范JS编程，束缚住前端们。
那么CommonJS规范了些什么呢？要解释这个规范，就要从JS的特性说起了。JS是一种直译式脚本语言，也就是一边编译一边运行，所以没有模块的概念。因此CommonJS是为了完善JS在这方面的缺失而存在的一种规范。 每个文件都是一个模块，有自己的作用域。在一个文件里面定义的变量、函数、类，都是私有的，对其他文件不可见。

#### CommonJS定义了两个主要概念：
	* require函数，用于导入模块
	* module.exports变量，用于导出模块

#### 特点
	1. 所有模块运行在模块自己的作用域中，不影响全局作用域；
	2. 模块可以被多次加载，但只有第一次会运行一次，后面再加载使用缓存。
	3. 读取模块的路径可以不加后缀，自动找到对应后缀模块。


commonJS的关键字是require和exports。然而这两个关键字，浏览器都不支持，所以我认为这是为什么浏览器不支持CommonJS的原因。如果一定要在浏览器上使用CommonJs，那么就需要一些编译库，比如browserify来帮助哦我们将CommonJs编译成浏览器支持的语法，其实就是实现require和exports。 CommonJS因为关键字的局限性，因此大多用于服务器端

那么CommonJS可以用于那些方面呢？虽然CommonJS不能再浏览器中直接使用，但是nodejs是基于CommonJS规范而实现的，亲儿子的感觉。在nodejs中我们就可以直接使用require和exports这两个关键词来实现模块的导入和导出。
Node.js是commonJS规范的主要实践者，它有四个重要的环境变量为模块化的实现提供支持：module、exports、require、global。实际使用时，用module.exports定义当前模块对外输出的接口（不推荐直接用exports），用require加载模块。

##### 例子：
```javascript
// 定义模块math.js
var basicNum = 0;
function add(a, b) {
  return a + b;
}
module.exports = { //在这里写上需要向外暴露的函数、变量
  add: add,
  basicNum: basicNum
}


// 引用自定义的模块时，参数包含路径，可省略.js
var math = require('./math');
math.add(2, 5);


// 引用核心模块时，不需要带路径
var http = require('http');
http.createService(...).listen(3000);

```
commonJS用同步的方式加载模块。在服务端，模块文件都存在本地磁盘，读取非常快，所以这样做不会有问题。但是在浏览器端，限于网络原因，更合理的方案是使用异步加载。

### 二、AMD和require.js
由于不是JavaScript原生支持，使用AMD规范进行页面开发需要用到对应的库函数，也就是大名鼎鼎RequireJS，实际上AMD 是 RequireJS 在推广过程中对模块定义的规范化的产出
AMD规范采用异步方式加载模块，模块的加载不影响它后面语句的运行。所有依赖这个模块的语句，都定义在一个回调函数中，等到加载完成之后，这个回调函数才会运行。这里介绍用require.js实现AMD规范的模块化：用require.config()指定引用路径等，用define()定义模块，用require()加载模块。

#### requireJS主要解决两个问题

1、多个js文件可能有依赖关系，被依赖的文件需要早于依赖它的文件加载到浏览器
2、js加载的时候浏览器会停止页面渲染，加载文件越多，页面失去响应时间越长

#### 语法
requireJS定义了一个函数 define，它是全局变量，用来定义模块
define(id?, dependencies?, factory);
	1. id：可选参数，用来定义模块的标识，如果没有提供该参数，脚本文件名（去掉拓展名）
	2. dependencies：是一个当前模块依赖的模块名称数组
	3. factory：工厂方法，模块初始化要执行的函数或对象。如果为函数，它应该只被执行一次。如果是对象，此对象应该为模块的输出值
在页面上使用require函数加载模块

```javascript
require([dependencies], function(){});
```

##### require()函数接受两个参数
	1. 第一个参数是一个数组，表示所依赖的模块
	2. 第二个参数是一个回调函数，当前面指定的模块都加载成功后，它将被调用。加载的模块会以参数形式传入该函数，从而在回调函数内部就可以使用这些模块


首先我们需要引入require.js文件和一个入口文件main.js。main.js中配置require.config()并规定项目中用到的基础模块。

```javascript
/** 网页中引入require.js及main.js **/
<script src="js/require.js" data-main="js/main"></script>


/** main.js 入口文件/主模块 **/
// 首先用config()指定各模块路径和引用名
require.config({
  baseUrl: "js/lib",
  paths: {
    "jquery": "jquery.min",  //实际路径为js/lib/jquery.min.js
    "underscore": "underscore.min",
  }
});
// 执行基本操作
require(["jquery","underscore"],function($,_){
  // some code here
});
```

引用模块的时候，我们将模块名放在[]中作为reqiure()的第一参数；如果我们定义的模块本身也依赖其他模块,那就需要将它们放在[]中作为define()的第一参数。
```javascript
// 定义math.js模块
define(function () {
    var basicNum = 0;
    var add = function (x, y) {
        return x + y;
    };
    return {
        add: add,
        basicNum :basicNum
    };
});
// 定义一个依赖underscore.js的模块
define(['underscore'],function(_){
  var classify = function(list){
    _.countBy(list,function(num){
      return num > 30 ? 'old' : 'young';
    })
  };
  return {
    classify :classify
  };
})


// 引用模块，将模块放在[]内
require(['jquery', 'math'],function($, math){
  var sum = math.add(10,20);
  $("#sum").html(sum);
});
```

### 三、CMD和sea.js
CMD规范是国内发展出来的，是另一种js模块化方案，它与AMD很类似，就像AMD有个requireJS，CMD有个浏览器的实现SeaJS。
SeaJS要解决的问题和requireJS一样，只不过在模块定义方式和模块加载（可以说运行、解析）时机上有所不同。

#### 不同点在于：
AMD 推崇依赖前置、提前执行，CMD推崇依赖就近、延迟执行。
1、AMD推崇依赖前置，在定义模块的时候就要声明其依赖的模块
2、CMD推崇就近依赖，只有在用到某个模块的时候再去require

这种区别各有优劣，只是语法上的差距，而且requireJS和SeaJS都支持对方的写法
AMD和CMD最大的区别是对依赖模块的执行时机处理不同，注意不是加载的时机或者方式不同

#### 语法
Sea.js 推崇一个模块一个文件，遵循统一的写法
因为CMD推崇
	1. 一个文件一个模块，所以经常就用文件名作为模块id
	2. CMD推崇依赖就近，所以一般不在define的参数中写依赖，在factory中写


factory是一个函数，有三个参数，function(require, exports, module)
	1. require 是一个方法，接受 模块标识 作为唯一参数，用来获取其他模块提供的接口：require(id)
	2. exports 是一个对象，用来向外提供模块接口
	3. module 是一个对象，上面存储了与当前模块相关联的一些属性和方法


require.js在申明依赖的模块时会在第一时间加载并执行模块内的代码：

```javascript
/** AMD写法 **/
define(["a", "b", "c", "d", "e", "f"], function(a, b, c, d, e, f) {
     // 等于在最前面声明并初始化了要用到的所有模块
    a.doSomething();
    if (false) {
        // 即便没用到某个模块 b，但 b 还是提前执行了
        b.doSomething()
    }
});


/** CMD写法 **/
define(function(require, exports, module) {
    var a = require('./a'); //在需要时申明
    a.doSomething();
    if (false) {
        var b = require('./b');
        b.doSomething();
    }
});


/** sea.js **/
// 定义模块 math.js
define(function(require, exports, module) {
    var $ = require('jquery.js');
    var add = function(a,b){
        return a+b;
    }
    exports.add = add;
});
// 加载模块
seajs.use(['math.js'], function(math){
    var sum = math.add(1+2);
});

```
很多人说requireJS是异步加载模块，SeaJS是同步加载模块，这么理解实际上是不准确的，其实加载模块都是异步的，只不过AMD依赖前置，js可以方便知道依赖模块是谁，立即加载，而CMD就近依赖，需要使用把模块变为字符串解析一遍才知道依赖了那些模块，这也是很多人诟病CMD的一点，牺牲性能来带来开发的便利性，实际上解析模块用的时间短到可以忽略
同样都是异步加载模块，AMD在加载模块完成后就会执行该模块，所有模块都加载执行完后会进入require的回调函数，执行主逻辑，这样的效果就是依赖模块的执行顺序和书写顺序不一定一致，看网络速度，哪个先下载下来，哪个先执行，但是主逻辑一定在所有依赖加载完成后才执行
CMD加载完某个依赖模块后并不执行，只是下载而已，在所有依赖模块加载完成后进入主逻辑，遇到require语句的时候才执行对应的模块，这样模块的执行顺序和书写顺序是完全一致的
这也是很多人说AMD用户体验好，因为没有延迟，依赖模块提前执行了，CMD性能好，因为只有用户需要的时候才执行的原因

### 四、ES6 Module
ES6 在语言标准的层面上，实现了模块功能，而且实现得相当简单，旨在成为浏览器和服务器通用的模块解决方案。其模块功能主要由两个命令构成：export和import。export命令用于规定模块的对外接口，import命令用于输入其他模块提供的功能。

```javascript
/** 定义模块 math.js **/
var basicNum = 0;
var add = function (a, b) {
    return a + b;
};
export { basicNum, add };


/** 引用模块 **/
import { basicNum, add } from './math';
function test(ele) {
    ele.textContent = add(99 + basicNum);
}
```
如上例所示，使用import命令的时候，用户需要知道所要加载的变量名或函数名。其实ES6还提供了export default命令，为模块指定默认输出，对应的import语句不需要使用大括号。这也更趋近于ADM的引用写法。

```javascript
/** export default **/
//定义输出
export default { basicNum, add };
//引入
import math from './math';
function test(ele) {
    ele.textContent = math.add(99 + math.basicNum);
}
```
ES6的模块不是对象，import命令会被 JavaScript 引擎静态分析，在编译时就引入模块代码，而不是在代码运行时加载，所以无法实现条件加载。也正因为这个，使得静态分析成为可能。

### 五、ES6 模块与 CommonJS 模块的差异

1. CommonJS 模块输出的是一个值的(浅)拷贝，ES6 模块输出的是值的引用。
	* CommonJS 模块输出的是值的拷贝，也就是说，一旦输出一个值，模块内部的变化就影响不到这个值。
	* ES6 模块的运行机制与 CommonJS 不一样。JS 引擎对脚本静态分析的时候，遇到模块加载命令import，就会生成一个只读引用。等到脚本真正执行时，再根据这个只读引用，到被加载的那个模块里面去取值。换句话说，ES6 的import有点像 Unix 系统的“符号连接”，原始值变了，import加载的值也会跟着变。因此，ES6 模块是动态引用，并且不会缓存值，模块里面的变量绑定其所在的模块。

2. CommonJS 模块是运行时加载，ES6 模块是编译时输出接口。
	* 运行时加载: CommonJS 模块就是对象；即在输入时是先加载整个模块，生成一个对象，然后再从这个对象上面读取方法，这种加载称为“运行时加载”。
	* 编译时加载: ES6 模块不是对象，而是通过 export 命令显式指定输出的代码，import时采用静态命令的形式。即在import时可以指定加载某个输出值，而不是加载整个模块，这种加载称为“编译时加载”。


CommonJS 加载的是一个对象（即module.exports属性），该对象只有在脚本运行完才会生成。而 ES6 模块不是对象，它的对外接口只是一种静态定义，在代码静态解析阶段就会生成。
