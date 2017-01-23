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
1. 輸出的是“Promise”。
2. then方法指定的callback。
3. 將當前script所有同步任務執行完才會執行。
4. 所以“Resolved”最後輸出。

``` js
var p1 = new Promise(function (resolve, reject) {
  setTimeout(() => reject(new Error('fail')), 3000)
})

var p2 = new Promise(function (resolve, reject) {
  setTimeout(() => resolve(p1), 1000)
})

p2
  .then(result => console.log(result))
  .catch(error => console.log(error))
// Error: fail
```

上面code中，p1是一個Promise，3秒之後狀態改變成rejected。p2的狀態在1秒之後改變，resolve方法返回的是p1。此時，由于p2返回的是另一個Promise，所以後面的then語句都變成針對後者（p1）。又過了2秒，p1變為rejected，導致觸發catch方法指定的callback。  

---

### **Promise.prototype.then()**

Promise instance具有then方法，也就是说，then方法是定義在Promise.prototype上的。它的作用是為Promise instance添加status改變時的callback function。then方法的第一個argument是Resolved狀態的callback，第二個argument是Rejected狀態的callback。  

``` js
    getJSON("/posts.json")
    .then(function(json) {
      return json.post;
    })
    .then(function(post) {
      // ...
    });
```

``` js
    getJSON("/post/1.json")
    .then(function(post) {
      return getJSON(post.commentURL);
    })
    .then(function funcA(comments) {
      console.log("Resolved: ", comments);
    }, function funcB(err){
      console.log("Rejected: ", err);
    });
```

上面code中，第一個then方法指定的callback，return的是另一個Promise object。這時，第二個then方法指定的callback，就會等待這個新的Promise object狀態發生變化。如果變為Resolved，就調用funcA，如果狀態變為Rejected，就調用funcB。  

如果採用箭頭函數，上面的代碼可以寫得更簡潔。  

``` js
   getJSON("/post/1.json")
   .then(post => getJSON(post.commentURL))
   .then(comments => console.log("Resolved: ", comments),
         err => console.log("Rejected: ", err)
);
```

---

### **Promise.all()**

Promise.all方法用於將多個Promise實例，包裝成一個新的Promise實例。  

``` js
 var p = Promise.all([p1, p2, p3]);
```

p的狀態由p1、p2、p3決定，分成兩種情況。

1. 只有p1、p2、p3的狀態都變成fulfilled，p的狀態才會變成fulfilled，此時p1、p2、p3的return value組成一個array，傳遞給p的callback。

2. 只要p1、p2、p3之中有一個被rejected，p的狀態就變成rejected，此時第一個被reject的instance的return，會傳遞給p的callback。

``` js
const databasePromise = connectDatabase();

const booksPromise = databaseProimse
  .then(findAllBooks);

const userPromise = databasePromise
  .then(getCurrentUser);

Promise.all([
  booksPromise,
  userPromise
])
.then(([books, user]) => pickTopRecommentations(books, user));
});
```

---

### **Promise.race()**

Promise.race方法同樣是將多個Promise實例，包裝成一個新的Promise實例。

* Promise.race方法的argument與Promise.all方法一樣，如果不是Promise instance，就會先invoked Promise.resolve()，將argument轉為Promise instance，再進一步處理。

``` js
var p = Promise.race([p1, p2, p3]);
```

上面code中，只要p1、p2、p3之中有一個instance率先改變狀態，p的狀態就跟著改變。那個率先改變的Promise instance的return value，就傳遞給p的callback。  

``` js
var p = Promise.race([
  fetch('/resource-that-may-take-a-while'),
  new Promise(function (resolve, reject) {
    setTimeout(() => reject(new Error('request timeout')), 5000)
  })
])
p.then(response => console.log(response))
p.catch(error => console.log(error))
```

上面代碼中，如果5秒之內fetch方法無法返回結果，變量p的狀態就會變為rejected，從而觸發catch方法指定的回調函數。

---

### **Promise.resolve()**

有時需要將現有object轉為Promise object，Promise.resolve方法就起到這個作用。  

Promise.resolve方法的參數分成四種情況。

1.argument是一個Promise instance

如果argument是Promise instance，那麼Promise.resolve將不做任何修改、原封不動地return這個instance。  

2.argument是一個thenable object

thenable object指的是具有then方法的object，比如下面這個object。  

``` js
let thenable = {
  then: function(resolve, reject) {
    resolve(42);
  }
};
```

Promise.resolve方法會將這個object轉為Promise object，然後就立即執行thenable object的then方法。  

``` js
let thenable = {
  then: function(resolve, reject) {
    resolve(42);
  }
};

let p1 = Promise.resolve(thenable);
p1.then(function(value) {
  console.log(value);  // 42
});
```

3.argument不是具有then方法的object，或根本就不是object

如果argument是一個primitive value，或者是一個不具有then方法的object，則Promise.resolve()return一個新的Promise object，狀態為Resolved。

``` js
var p = Promise.resolve('Hello');

p.then(function (s){
  console.log(s)
});
// Hello
```

上面code生成一個新的Promise object的instance "p"。

4.不帶有任何參數

直接返回一個Resolved狀態的Promise對象。

``` js
var p = Promise.resolve();

p.then(function () {
  // ...
});
```

# **Promise.prototype.catch()**