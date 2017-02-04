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
 
有時，我們會在object上面設置內部屬性，property name的第一個字符使用底線開頭，表示這些屬性不應該被外部使用。結合get和set方法，就可以做到防止這些內部屬性被外部讀寫。

``` js
var handler = {
  get (target, key) {
    func(key, 'get');
    return target[key];
  },
  set (target, key, value) {
    func(key, 'set');
    target[key] = value;
    return true;
  }
};
function func(key, action) {
  if (key[0] === '_') {
    throw new Error(`Invalid attempt to ${action} private "${key}" property`);
  }
}
var target = {};
var proxy = new Proxy(target, handler);
proxy._prop
// Error: Invalid attempt to get private "_prop" property
proxy._prop = 'c'
// Error: Invalid attempt to set private "_prop" property
```
 
#### **<font color = 'red'>apply()</font>** 

apply方法攔截函數的調用、call和apply操作。

apply方法可以接受三個參數，分別是目標對象、目標對象的上下文對象（this）和目標對象的參數數組。 

``` js
var handler = {
  apply (target, ctx, args) {
    return Reflect.apply(...arguments);
  }
};
```

下面是一個例子。  

``` js
var target = function () { return 'I am the target'; };
var handler = {
  apply: function () {
    return 'I am the proxy';
  }
};

var p = new Proxy(target, handler);

p()
// "I am the proxy"
```

下面是另外一個例子。

``` js
var twice = {
  apply (target, ctx, args) {
    return Reflect.apply(...arguments) * 2;
  }
};
function sum (left, right) {
  return left + right;
};
var proxy = new Proxy(sum, twice);
proxy(1, 2) // 6
proxy.call(null, 5, 6) // 22
proxy.apply(null, [7, 8]) // 30
```

上面code中，每當執行proxy function（直接調用或call和apply調用），就會被apply方法攔截。  

#### **<font color = 'red'>has()</font>**  

has方法用來interception HasProperty操作，即判斷object是否具有某個property時，這個方法會生效。典型的操作就是in運算符。 

下面的例子使用has方法隱藏某些屬性，不被in運算符發現。  

``` js
var handler = {
  has (target, key) {
    if (key[0] === '_') {
      return false;
    }
    return key in target;
  }
};
var target = { _prop: 'foo', prop: 'foo' };
var proxy = new Proxy(target, handler);
'_prop' in proxy // false
```

如果原對像不可配置或者禁止擴展，這時has攔截會報錯。

``` js
var obj = { a: 10 };
Object.preventExtensions(obj);

var p = new Proxy(obj, {
  has: function(target, prop) {
    return false;
  }
});

'a' in p // TypeError is thrown
```

上面代碼中，obj對象禁止擴展，結果使用has攔截就會報錯。也就是說，如果某個屬性不可配置（或者目標對像不可擴展），則has方法就不得“隱藏”（即返回false）目標對象的該屬性。  

* has方法interception的是HasProperty操作，而不是HasOwnProperty操作，即has方法不判斷一個屬性是object自身的property，還是繼承的property。  

另外，雖然for...in循環也用到了in運算符，但是has interception對for...in循環不生效。  

``` js
let stu1 = {name: '張三', score: 59};
let stu2 = {name: '李四', score: 99};

let handler = {
  has(target, prop) {
    if (prop === 'score' && target[prop] < 60) {
      console.log(`${target.name} 不及格`);
      return false;
    }
    return prop in target;
  }
}

let oproxy1 = new Proxy(stu1, handler);
let oproxy2 = new Proxy(stu2, handler);

'score' in oproxy1
// 張三 不及格
// false

'score' in oproxy2
// true

for (let a in oproxy1) {
  console.log(oproxy1[a]);
}
// 張三
// 59

for (let b in oproxy2) {
  console.log(oproxy2[b]);
}
// 李四
// 99
```

上面代碼中，has interception只對in循環生效，對for...in循環不生效，導致不符合要求的屬性沒有被排除在for...in循環之外。

#### **<font color = 'red'>constructor()</font>**  

construct方法用於interception new命令，下面是interception object的寫法。

``` js
var handler = {
  construct (target, args, newTarget) {
    return new target(...args);
  }
};
```

construct方法可以接受兩個參數。

* target: 目標對象
* args：構建函數的參數對象

``` js
var p = new Proxy(function () {}, {
  construct: function(target, args) {
    console.log('called: ' + args.join(', '));
    return { value: args[0] * 10 };
  }
});

(new p(1)).value
// "called: 1"
// 10
```

construct方法返回的必須是一個對象，否則會報錯。

``` js
var p = new Proxy(function() {}, {
  construct: function(target, argumentsList) {
    return 1;
  }
});

new p() // 报错
```