# 7.Promise
---
>所謂Promise，簡單說就是一個容器，裡面保存著某個未來才會結束的event（通常是一個異步操作）結果。從語法上說，Promise是一個object，從它可以獲取異步操作的消息。  

Promise object有以下兩個特點。

* 1.Promise的status不受外界影響。Promise object代表一個異步操作，有三種status：Pending（進行中）、Resolved（已完成，又稱Fulfilled）和Rejected（已失敗）。只有異步操作的結果，可以決定當前是哪一種狀態，任何其他操作都無法改變這個狀態。  

* 2.一旦狀態改變，就不會再變。Promise object status改變，只有兩種可能：從Pending變為Resolved和從Pending變為Rejected。只要這兩種情況發生，狀態就凝固了，不會再變了，會一直保持這個結果。就算改變已經發生了，你再對Promise對象添加callback function，也會立即得到這個結果。  

### **基本用法**

ES6規定，Promise object是一個構造函數，用來生成Promise instance。

``` js
var promise = new Promise(function(resolve, reject) {
  // ... some code

  if (/* 异步操作成功 */){
    resolve(value);
  } else {
    reject(error);
  }
});
```

Promise構造函數接受一個function作為argument，該function的兩個argument分別是<font color = 'red'>resolve</font>和<font color = 'red'>reject</font>。它們是function，由JavaScript engine提供，不用自己部署。

* <font color = 'red'>resolve function</font>的作用是，將Promise對象的狀態從“未完成”變為“成功”（即從Pending變為Resolved），在異步操作成功時調用，並將異步操作的結果，作為參數傳遞出去；  

* <font color = 'red'>reject function</font>的作用是，將Promise對象的狀態從“未完成”變為“失敗”（即從Pending變為Rejected），在異步操作失敗時調用，並將異步操作報出的錯誤，作為參數傳遞出去。


Promise instance生成以後，可以用then方法分別指定Resolved和Reject的callback function。  

``` js
promise.then(function(value) {
  // success
}, function(error) {
  // failure
});
```

Promise新建後就會立即執行。

``` js
let promise = new Promise(function(resolve, reject) {
  console.log('Promise');
  resolve();
});

                                                        promise.then(function() {
                                                          console.log('Resolved.');
                                                        });

console.log('Hi!');

// Promise
// Hi!
// Resolved.
```

上面code，Promise新建後立即執行
* 1. 輸出的是“Promise”。
* 2. then方法指定的回調函數。
* 3. 將當前腳本所有同步任務執行完才會執行。
* 4. 所以“Resolved”最後輸出。


