Promise 是异步编程的一种解决方案： 从语法上讲，promise是一个对象，从它可以获取异步操作的消息；从本意上讲，它是承诺，承诺它过一段时间会给你一个结果。 promise有三种状态：pending(等待态)，fulfiled(成功态)，rejected(失败态)；状态一旦改变，就不会再变。创造promise实例后，它会立即执行。

promise用来解决两个问题的：
	* 回调地狱，代码难以维护， 常常第一个的函数的输出是第二个函数的输入这种现象
	* promise可以支持多个并发的请求，获取并发请求中的数据
	
promise可以解决异步的问题，但不能说promise是异步的



promise 有 3 个状态，分别是 pending, fulfilled 和 rejected。

在 pending 状态，promise 可以切换到 fulfilled 或 rejected。

在 fulfilled 状态，不能迁移到其它状态，必须有个不可变的 value。

在 rejected 状态，不能迁移到其它状态，必须有个不可变的 reason。

##### reject的用法 :
把Promise的状态置为rejected，这样我们在then中就能捕捉到，然后执行“失败”情况的回调。看下面的代码。

##### catch的用法：
我们知道Promise对象除了then方法，还有一个catch方法，它是做什么用的呢？
其实它和then的第二个参数一样，用来指定reject的回调。

##### all的用法：
谁跑的慢，以谁为准执行回调。all接收一个数组参数，里面的值最终都算返回Promise对象。Promise的all方法提供了并行执行异步操作的能力，并且在所有异步操作执行完后才执行回调。

##### race的用法：
谁跑的快，以谁为准执行回调

### Promise实现
* 步骤一：实现成功和失败的回调方法
* 步骤二：then方法链式调用
* 步骤三：链式调用成功之后

手写Promise思路：

步骤一：创建一个Promise Class， 实现promise的三种状态以及成功和失败回调的函数。
步骤二：在Promise Class内构造一个then构造函数，实现then方法的链式调用
步骤三：创建一个resolvePromise函数，解析then方法链式调用的结果。 


```javascript
unction resolvePromise(promise2,x,resolve,reject){
    //判断x是不是promise
    //规范中规定：我们允许别人乱写，这个代码可以实现我们的promise和别人的promise 进行交互
    if(promise2 === x){//不能自己等待自己完成
        return reject(new TypeError('循环引用'));
    };
    // x是除了null以外的对象或者函数
    if(x !=null && (typeof x === 'object' || typeof x === 'function')){
        let called;//防止成功后调用失败
        try{//防止取then是出现异常  object.defineProperty
            let then = x.then;//取x的then方法 {then:{}}
            if(typeof then === 'function'){//如果then是函数就认为他是promise
                //call第一个参数是this，后面的是成功的回调和失败的回调
                then.call(x,y => {//如果Y是promise就继续递归promise
                    if(called) return;
                    called = true;
                    resolvePromise(promise2,y,resolve,reject)
                },r => { //只要失败了就失败了
                    if(called) return;
                    called = true;
                    reject(r);  
                });
            }else{//then是一个普通对象，就直接成功即可
                resolve(x);
            }
        }catch (e){
            if(called) return;
            called = true;
            reject(e)
        }
    }else{//x = 123 x就是一个普通值 作为下个then成功的参数
        resolve(x)
    }


}


class Promise {
    constructor (executor){
        //默认状态是等待状态
        this.status = 'panding';
        this.value = undefined;
        this.reason = undefined;
        //存放成功的回调
        this.onResolvedCallbacks = [];
        //存放失败的回调
        this.onRejectedCallbacks = [];
        let resolve = (data) => {//this指的是实例
            if(this.status === 'pending'){
                this.value = data;
                this.status = "resolved";
                this.onResolvedCallbacks.forEach(fn => fn());
            }
        }
        let reject = (reason) => {
            if(this.status === 'pending'){
                this.reason = reason;
                this.status = 'rejected';
                this.onRejectedCallbacks.forEach(fn => fn());
            }
        }
        try{//执行时可能会发生异常
            executor(resolve,reject);
        }catch (e){
            reject(e);//promise失败了
        }
       
    }
    then(onFuiFilled,onRejected){
        //防止值得穿透
        onFuiFilled = typeof onFuiFilled === 'function' ? onFuiFilled : y => y;
        onRejected = typeof onRejected === 'function' ? onRejected :err => {throw err;}        
        let promise2;//作为下一次then方法的promise
       if(this.status === 'resolved'){
           promise2 = new Promise((resolve,reject) => {
               setTimeout(() => {
                  try{
                        //成功的逻辑 失败的逻辑
                        let x = onFuiFilled(this.value);
                        //看x是不是promise 如果是promise取他的结果 作为promise2成功的的结果
                        //如果返回一个普通值，作为promise2成功的结果
                        //resolvePromise可以解析x和promise2之间的关系
                        //在resolvePromise中传入四个参数，第一个是返回的promise，第二个是返回的结果，第三个和第四个分别是resolve()和reject()的方法。
                        resolvePromise(promise2,x,resolve,reject)
                  }catch(e){
                        reject(e);
                  }
               },0)
           });
       }
       if(this.status === 'rejected'){
            promise2 = new Promise((resolve,reject) => {
                setTimeout(() => {
                    try{
                        let x = onRejected(this.reason);
                        //在resolvePromise中传入四个参数，第一个是返回的promise，第二个是返回的结果，第三个和第四个分别是resolve()和reject()的方法。
                        resolvePromise(promise2,x,resolve,reject)
                    }catch(e){
                        reject(e);
                    }
                },0)


            });
       }
       //当前既没有完成也没有失败
       if(this.status === 'pending'){
           promise2 = new Promise((resolve,reject) => {
               //把成功的函数一个个存放到成功回调函数数组中
                this.onResolvedCallbacks.push( () =>{
                    setTimeout(() => {
                        try{
                            let x = onFuiFilled(this.value);
                            resolvePromise(promise2,x,resolve,reject);
                        }catch(e){
                            reject(e);
                        }
                    },0)
                });
                //把失败的函数一个个存放到失败回调函数数组中
                this.onRejectedCallbacks.push( ()=>{
                    setTimeout(() => {
                        try{
                            let x = onRejected(this.reason);
                            resolvePromise(promise2,x,resolve,reject)
                        }catch(e){
                            reject(e)
                        }
                    },0)
                })
           })
       }
       return promise2;//调用then后返回一个新的promise
    }
    catch (onRejected) {
        // catch 方法就是then方法没有成功的简写
        return this.then(null, onRejected);
    }
}

Promise.all = function (promises) {
    //promises是一个promise的数组
    return new Promise(function (resolve, reject) {
        let arr = []; //arr是最终返回值的结果
        let i = 0; // 表示成功了多少次
        function processData(index, data) {
            arr[index] = data;
            if (++i === promises.length) {
                resolve(arr);
            }
        }
        for (let i = 0; i < promises.length; i++) {
            promises[i].then(function (data) {
                processData(i, data)
            }, reject)
        }
    })
}

// 只要有一个promise成功了 就算成功。如果第一个失败了就失败了
Promise.race = function (promises) {
    return new Promise((resolve, reject) => {
        for (var i = 0; i < promises.length; i++) {
            promises[i].then(resolve,reject)
        }
    })
}

// 生成一个成功的promise
Promise.resolve = function(value){
    return new Promise((resolve,reject) => resolve(value);
}

// 生成一个失败的promise
Promise.reject = function(reason){
    return new Promise((resolve,reject) => reject(reason));
}

Promise.defer = Promise.deferred = function () {
    let dfd = {};
    dfd.promise = new Promise( (resolve, reject) =>  {
        dfd.resolve = resolve;
        dfd.reject = reject;
    });
    return dfd
}


```
