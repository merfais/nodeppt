title: promise 介绍
speaker: 毕文清
url: https://merfais.github.io/nodeppt/promise.htmnl
transition: slide
files: /js/demo.js,/css/demo.css

[slide]

# Promise 介绍
## 演讲者：毕文清

[slide]
# Why Promise
----
- 回调函数 劣势 {:&.moveIn}
    * [回调金字塔，层层缩进](Promise_example.md#0) {:&.rollIn} 
    * [大量回调跳转，容易迷失在回调迷宫里](Promise_example.md#1)
    * [高质量的回调函数不容易构造与编写](Promise_example.md#3)
- Promise 优势
    + 无论是异步或是同步，都可以在Promise Chain 中顺序执行 {:&.rollIn}
    + 流程可知，可控

[slide]
# What Promise
> 所谓Promise，简单说就是一个容器，里面保存着某个未来才会结束的事件（通常是一个异步操作）的结果。（这里的翻译源自ECMAScript 2015关于Promise的解释，没有原文翻译MDN的原话，如果您有疑问，可以参看英文的说明文档：[MDN Pormise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise) ）

[slide]

## 状态
+ "unresolved" - Pending: 初始状态, 既不是 fulfilled 也不是 rejected. {:&.rollIn}
+ "has-resolution" - Fulfilled: 成功的操作.此时会调用 onFulfilled
+ "has-rejection" - Rejected: 失败的操作.此时会调用 onRejected
+ 说明
    > pending状态的promise对象既可转换为带着一个成功值的fulfilled状态，也可变为带着一个失败信息的 rejected 状态。
    > 当状态发生转换时，promise.then绑定的方法就会被调用。(当绑定方法时，如果 promise对象已经处于 fulfilled 或 rejected 状态，那么相应的方法将会被立刻调用)
+ 特别注意
    > promise对象的状态，从Pending转换为Fulfilled或Rejected之后，这个promise对象的状态就不会再发生任何变化。也就是说，在.then 后执行的函数可以肯定地说只会被调用一次。另外，Fulfilled和Rejected这两个中的任一状态都可以表示为Settled(不变的)。

[slide]
## 构造函数
```javascript
var promise = new Promise(function(resolve, reject) {
    // 异步处理
    // 处理结束后、调用resolve 或 reject
    if (success) {
        resolve();
    } 
    if(error) {
        reject();
    }
});
```
这样创建的`promise`的就是pending状态的，只有当内部处理完成,直到`resolve`或`reject`被调用，其会一直处在pending状态。当状态一旦发生改变则立即调用`then`方法进行后续处理（下面会讲到`catch`可以理解为`then`的一个变种）

[slide]
## 实例方法
```javascript
promise.then(onFulfilled, onRejected)
```
+ resolve(成功)时,`onFulfilled`会被调用 {:&.rollIn}
+ reject(失败)时, `onRejected`会被调用
+ 说明
    * onFulfilled、onRejected 两个都为可选参数。 {:&.moveIn}
    * 但采用`promise.then(undefined, onRejected)` 不如 `promise.catch(onRejected)`
+ [案例](Promise_example.md#4)

[slide]
## 实例方法
```javascript
promise.catch(onRejected|onError)
```
+ reject(失败)时, `onRejected`会被调用 {:&.rollIn}
+ throw Error 时， `onError`会被调用
+ 说明
    * 无论是`reject`还是`error`发生都会进入catch {:&.moveIn}
    * 当使用promise.then(undefined, onRejected).catch(onRejected)时，判定错误来源是件很头疼的事（墙裂不推荐）

[slide]
## 静态方法
+ Promise.resolve {:&.rollIn}
+ Promise.reject
+ Promise.all
+ Promise.race
+ 说明
    + Promise.resolve可简单理解为 new Promise() 的快捷方式，在promise初始化或者编写测试代码的时候都非常方便。 {:&.moveIn}
    + 同理 Promise.reject

[slide]
## 举个栗子
----
```javascript
Promise.resolve(42).then(function(value){
    console.log(value);
});
```
相当于
```javascript
let testP = new Promise(function(resolve){
    resolve(42);
});
testP.then(function(value){
    console.log(value);
});
```

[slide]
## 举个栗子
----
```javascript
new Promise(function(resolve,reject){
    reject(new Error("出错了"));
});
```
相当于
```javascript
Promise.reject(new Error("BOOM!")).catch(function(error){
    console.error(error);
});
```

[slide]
# How Promise
## 我们只谈 <span class="red" style="font-size: 2em">坑</span>

[slide]
## 用代码说话
----
1. [永远不要忘了return，除非你是猴子派来的](Promise_example.md#5) {:&.bounceIn}
2. [每次`.then`都会返回全新的promise，就是这么任性](Promise_example.md#7)
3. [Promise Chain 中的异步操作，一定只能使用 `new Promise` 来 resolve](Promise_example.md#9)
4. [new Promise内一定要`try{}catch(e){reject(e)}`,除非你想崩溃](Promise_example.md#10)
5. [即使已经`resolve` or `reject`，但后面的代码依旧我行我素](Promise_example.md#14)
6. [即使`onReject`了，Promise Chain也不会停下前进的脚步](Promise_example.md#15)
7. [Pending 到 Settled 只有一次机会，没有后悔药](Promise_example.md#16)
8. [`.catch` 要比 `.then(null,onReject)` 好用一万倍](Promise_example.md#17)

[slide]
# [对Promise的hack](Promise_example.md#18)
+ 增加`Promise.stop(data)` 终止 Promise Chain {:&.rollIn}
+ 重写了then的替代方法 `.next(data)` 跳过所有的Chain
+ 增加 `.whenStop(data)` 捕获 Promise.stop 的时刻

[slide]
<div style="text-align: left;">
<p style='margin: 0px auto; width: 450px'>
如果时间允许，我们分析一下，<br>
从 <code>ElasticSearch</code> 拉去数据 <br>
到 <code>G2</code>渲染页面的promise过程<br>
</p></div>

[slide]
# > 感谢各位宝贵的时间 <

[slide]
# 项目问题
+ 项目架构和目录 {:&.rollIn}
+ request api | 各模块 API 什么时候到位
+ 面包屑组件（vuex的理解）
+ 导航组件（对router-view的理解）
+ 时间组件（是否扩展）
+ 表单组件（validator的理解，valid流程）
    * input （password） {:&.moveIn}
    * radio
    * checkBox
    * select
