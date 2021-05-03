#### 循环实现数组 map 方法

第一个参数是一个函数，用来对数组进行处理。第二个参数为一个对象，可以用来更改this，默认是window（可不传）

```javascript
//普通实现
let selfMap=function (fn,context) {
  let arr=Array.prototype.slice.call(this);
  let newArr=Array();
  for(let i=0;i<arr.length;i++){
    if(!arr.hasOwnProperty(i))continue;
    newArr[i]=fn.call(context,arr[i],i,this)
  }
  return newArr;
}
Array.prototype.selfMap=selfMap

//reduce实现
let selfMap2=function (fn,context) {
  let arr=Array.prototype.slice.call(this);
  return arr.reduce((pre,cur,index)=>{
    return [...pre,fn.call(context,cur,index,this)]
  },[])
}



let ssr=[7,4,3,26,2];
obj12={
  name:"aaaaa"
}
let fa=null;
let aa=ssr.selfMap(function (value) {
  fa=this;
  return value+1
},obj12)
console.log(aa);
console.log(fa);
```

#### filter()
filter() 方法创建一个新的数组，新数组中的元素是通过检查指定数组中符合条件的所有元素。
filter()和map()类似，Array的filter()也接收一个函数。和map()不同的是，filter()把传入的函数依次作用于每个元素，然后根据返回值是true还是false决定保留还是丢弃该元素。

```javascript
let selfFilter=function (fn,context) {
  let arr=Array.prototype.slice.call(this);
  let newArr=Array();
  for(let i=0;i<arr.length;i++){
    if(!arr.hasOwnProperty(i))continue;
    fn.call(context,arr[i],i,this)&&newArr.push(arr[i])
  }
  return newArr;
}
Array.prototype.selfFilter=selfFilter;

//reduce实现
let selfFilter2=function (fn,contex) {
  let arr=Array.prototype.slice.call(this);
  return arr.reduce((pre,cur,index)=>{
    return fn.call(context,cur,index,this)?[...pre,cur]:[...pre]
  })
}

let ssr=[7,4,3,26,2];
obj12={
  name:"aaaaa"
}
let aaa=ssr.selfFilter(function (value) {
  if(value%2==0)
  return value
})
console.log(aaa);
```

#### reduce 方法
定义：接收一个函数作为累加器，数组中的每一个值（从左到右）开始遍历，最终计算为一个值。
注意：对空数组是不会执行回调函数的
```javascript
arr.reduce(callback,[initialValue])
callback （执行数组中每个值的函数，包含四个参数）

    1、previousValue （上一次调用回调返回的值，或者是提供的初始值（initialValue））
    2、currentValue （数组中当前被处理的元素）
    3、index （当前元素在数组中的索引）
    4、array （调用 reduce 的数组）


initialValue （作为第一次调用 callback 的第一个参数。）

let selfReduce=function (fn,initialValue) {
  var arr = Array.prototype.slice.call(this);
    var res, startIndex;
    res = initialValue ? initialValue : arr[0];
    startIndex = initialValue ? 0 : 1;
    for(var i = startIndex; i < arr.length; i++) {
      res = fn.call(null, res, arr[i], i, this);
    }
    return res;
}
Array.prototype.selfReduce=selfReduce;
```

#### some
执行 some 方法的数组如果是一个空数组，最终始终会返回 false，而另一个数组的 every 方法中的数组如果是一个空数组，会始终返回 true

```javascript
let selfSome=function (fn,context) {
  let arr=Array.prototype.slice.call(this);
  if(!arr.length)return false;
  for(let i=0;i<arr.length;i++){
    if(!arr.hasOwnProperty(i))continue;
    let res=fn.call(context,arr[i],i,this)
    if(res)return true;
  }
  return false;
}
```

#### flat
flat()用于将嵌套的数组“拉平”，变成一维数组。该方法返回一个新数组，对原数据没有影响。 flat()默认只会“拉平”一层，如果想要“拉平”多层的嵌套数组，可以将flat()方法的参数写成一个整数，表示想要拉平的层数，默认为1。

```javascript
selfFlat=function (depth=1) {
  let arr=Array.prototype.slice.call(this);
  if(depth==0)return arr;
  return arr.reduce((pre,cur)=>{
    if(Array.isArray(cur)){
      return [...pre,...selfFlat.call(cur,depth-1)]
    }else{
      return [...pre,cur]
    }
  },[])
}
```

