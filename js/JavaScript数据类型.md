### 一、JavaScript数据类型

ECMAScript标准规定了8种数据类型，其把这8种数据类型又分为两种：原始类型和对象类型。
原始类型
	* Null：只包含一个值：null
	* Undefined：只包含一个值：undefined
	* Boolean：包含两个值：true和false
	* Number：整数或浮点数，还有一些特殊值（-Infinity、+Infinity、NaN）
	* String：一串表示文本值的字符序列
	* Symbol：一种实例是唯一且不可改变的数据类型
	* bigInt（es10新出的， 比Number数据类型支持的范围更大的整数值。使用BigInt，整数溢出将不再是问题）

对象类型
	* Object：自己分一类丝毫不过分，除了常用的Object，Array、Function等都属于特殊的对象


### 二、为什么区分原始类型和对象类型

#### 2.1 不可变性

上面所提到的原始类型，在ECMAScript标准中，它们被定义为primitive values，即原始值，代表值本身是不可被改变的。
以字符串为例，我们在调用操作字符串的方法时，没有任何方法是可以直接改变字符串的：

```javascript
var str = 'ConardLi';
str.slice(1);
str.substr(1);
str.trim(1);
str.toLowerCase(1);
str[0] = 1;
console.log(str);  // ConardLi
```
在上面的代码中我们对str调用了几个方法，无一例外，这些方法都在原字符串的基础上产生了一个新字符串，而非直接去改变str，这就印证了字符串的不可变性。

那么，当我们继续调用下面的代码：

```javascript
str += '6'
console.log(str);  // ConardLi6
```

你会发现，str的值被改变了,是不可变性不起作用吗？其实不是，我们从内存上来理解：

在JavaScript中，每一个变量在内存中都需要一个空间来存储。

内存空间又被分为两种，栈内存与堆内存。

栈内存：

	* 存储的值大小固定
	* 空间较小
	* 可以直接操作其保存的变量，运行效率高
	* 由系统自动分配存储空间

JavaScript中的原始类型的值被直接存储在栈中，在变量定义时，栈就为其分配好了内存空间。
由于栈中的内存空间的大小是固定的，那么注定了存储在栈中的变量就是不可变的。
在上面的代码中，我们执行了str += '6'的操作，实际上是在栈中又开辟了一块内存空间用于存储'ConardLi6'，然后将变量str指向这块空间，所以这并不违背不可变性的特点。

#### 2.2 引用类型
堆内存：
	* 存储的值大小不定，可动态调整
	* 空间较大，运行效率低
	* 无法直接操作其内部存储，使用引用地址读取
	* 通过代码进行分配空间


相对于上面具有不可变性的原始类型，我习惯把对象称为引用类型，引用类型的值实际存储在堆内存中，它在栈中只存储了一个固定长度的地址，这个地址指向堆内存中的值。
引用类型不具有不可变性，我们可以轻易的改变它们

### 原始类型和对象类型的区别

####复制
当我们把一个变量的值复制到另一个变量上时，原始类型和引用类型的表现是不一样的，先来看看原始类型：

```javascript
var name = 'ConardLi';
var name2 = name;
name2 = 'code秘密花园';
console.log(name); // ConardLi;
```

内存中有一个变量name，值为ConardLi。我们从变量name复制出一个变量name2，此时在内存中创建了一个块新的空间用于存储ConardLi，虽然两者值是相同的，但是两者指向的内存空间完全不同，这两个变量参与任何操作都互不影响

复制一个引用类型：

```javascript
var obj = {name:'ConardLi'};
var obj2 = obj;
obj2.name = 'code秘密花园';
console.log(obj.name); // code秘密花园
```

当我们复制引用类型的变量时，实际上复制的是栈中存储的地址，所以复制出来的obj2实际上和obj指向的堆中同一个对象。因此，我们改变其中任何一个变量的值，另一个变量都会受到影响，这就是为什么会有深拷贝和浅拷贝的原因。

#### 比较

当我们在对两个变量进行比较时，不同类型的变量的表现是不同的：
对于原始类型，比较时会直接比较它们的值，如果值相等，即返回true。
对于引用类型，比较时会比较它们的引用地址，虽然两个变量在堆中存储的对象具有的属性值都是相等的，但是它们被存储在了不同的存储空间，因此比较值为false。

