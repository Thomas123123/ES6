# 3.Proxy

---

Proxy 可以理解成，在目標對象之前架設一層“interception”，外界對該object的訪問，都必須先通過這層interception，因此提供了一種機制，可以對外界的訪問進行過濾和改寫。Proxy 這個詞的原意是代理，用在這裡表示由它來“代理”某些操作，可以譯為“代理器”。

ES6 原生提供 Proxy 構造函数，用來生成 Proxy  instance。

``` js
var proxy = new Proxy(target, handler);
```

new Proxy()表示生成一個Proxy instance，<font color = 'red'>target</font>參數表示所要攔截的目標object，handler參數也是一個object，用來定制interception行為。

``` js
var proxy = new Proxy({}, {
  get: function(target, property) {
    return 35;
  }
});

proxy.time // 35
proxy.name // 35
proxy.title // 35
```

* 注意，要使得Proxy起作用，必須針對Proxy instance（上例是proxy對象）進行操作，而不是針對目標object（上例是空object）進行操作。

如果handler沒有設置任何interception，那就等同於直接通向原object。

``` js
var target = {};
var handler = {};
var proxy = new Proxy(target, handler);
proxy.a = 'b';
target.a // "b"
```

Proxy instance也可以作為其他object的prototype object。

``` js
var proxy = new Proxy({}, {
  get: function(target, property) {
    return 35;
  }
});

let obj = Object.create(proxy);
obj.time // 35
```

上面code中，proxy object是obj object的prototype，obj object本身並沒有time屬性，所以根據prototype chain，會在proxy object上讀取該property，導致被interception。  

同一個interceptor function，可以設置多個interception操作。

``` js
var handler = {
  get: function(target, name) {
    if (name === 'prototype') {
      return Object.prototype;
    }
    return 'Hello, ' + name;
  },

  apply: function(target, thisBinding, args) {
    return args[0];
  },

  construct: function(target, args) {
    return {value: args[1]};
  }
};

var fproxy = new Proxy(function(x, y) {
  return x + y;
}, handler);

fproxy(1, 2) // 1
new fproxy(1,2) // {value: 2}
fproxy.prototype === Object.prototype // true
fproxy.foo // "Hello, foo"
```

下面是 Proxy 支持的interception方法。

* 對於可以設置、但没有设置interception的操作，則直接落在目標object上，按照原先的方式產生结果。

**（1）get(target, propKey, receiver)**

**（2）set(target, propKey, value, receiver)**

**（3）has(target, propKey)**

**（4）deleteProperty(target, propKey)**

**（5）ownKeys(target)**

**（6）getOwnPropertyDescriptor(target, propKey)**

**（7）defineProperty(target, propKey, propDesc)**

**（8）preventExtensions(target)**

**（9）getPrototypeOf(target)**

**（10）isExtensible(target)**

**（11）setPrototypeOf(target, proto)**

**（12）apply(target, object, args)**

**（13）construct(target, args)**

---

## **Proxy instance的方法**

#### **<font color = 'red'>get()</font>**

用於攔截某個屬性的讀取操作。

``` js
var person = {
  name: "thomas"
};

var proxy = new Proxy(person, {
  get: function(target, property) {
    if (property in target) {
      return target[property];
    } else {
      throw new ReferenceError("Property \"" + property + "\" does not exist.");
    }
  }
});

proxy.name // "thomas"
proxy.age // 拋出一個錯誤
```

下面的例子使用get攔截，實現數組讀取負數的索引。

``` js
function createArray(...elements) {
  let handler = {
    get(target, propKey, receiver) {
      let index = Number(propKey);
      if (index < 0) {
        propKey = String(target.length + index);
      }
      return Reflect.get(target, propKey, receiver);
    }
  };

  let target = [];
  target.push(...elements);
  return new Proxy(target, handler);
}

let arr = createArray('a', 'b', 'c');
arr[-1] // c
```

利用Proxy，可以將讀取屬性的操作（get），轉變為執行某個函數，從而實現屬性的鍊式操作。

``` js
var pipe = (function () {
  return function (value) {
    var funcStack = [];
    var oproxy = new Proxy({} , {
      get : function (pipeObject, fnName) {
        if (fnName === 'get') {
          return funcStack.reduce(function (val, fn) {
            return fn(val);
          },value);
        }
        funcStack.push(window[fnName]);
        return oproxy;
      }
    });

    return oproxy;
  }
}());

var double = n => n * 2;
var pow    = n => n * n;
var reverseInt = n => n.toString().split("").reverse().join("") | 0;

pipe(3).double.pow.reverseInt.get; // 63
```

上面code設置 Proxy 以後，達到了將function name鏈式使用的效果。

#### **<font color = 'red'>set()</font>**  

set方法用來interception某個property的賦值操作。

``` js
let validator = {
  set: function(obj, prop, value) {
    if (prop === 'age') {
      if (!Number.isInteger(value)) {
        throw new TypeError('The age is not an integer');
      }
      if (value > 200) {
        throw new RangeError('The age seems invalid');
      }
    }

    // 對於age以外的property，直接保存
    obj[prop] = value;
  }
};

let person = new Proxy({}, validator);

person.age = 100;

person.age // 100
person.age = 'young' // 'The age is not an integer'
person.age = 300 // The age seems invalid
```
 












