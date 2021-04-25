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
