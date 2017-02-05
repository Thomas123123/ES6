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
let handler = {
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

let person = new Proxy({}, handler);

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

new p() // Uncaught TypeError: 'construct' on proxy: trap returned non-object ('1')
```

#### **<font color = 'red'>deleteproperty()</font>**  

deleteProperty方法用於interception delete操作，如果這個方法拋出錯誤或者返回false，當前屬性就無法被delete命令刪除。

``` js
var handler = {
  deleteProperty (target, key) {
    invariant(key, 'delete');
    return true;
  }
};
function invariant (key, action) {
  if (key[0] === '_') {
    throw new Error(`Invalid attempt to ${action} private "${key}" property`);
  }
}

var target = { _prop: 'foo' };
var proxy = new Proxy(target, handler);
delete proxy._prop
// Error: Invalid attempt to delete private "_prop" property
```

上面代碼中，deleteProperty方法攔截了delete操作符，刪除第一個字符為下劃線的屬性會報錯。

注意，目標對象自身的不可配置（configurable）的屬性，不能被deleteProperty方法刪除，否則報錯。

#### **<font color = 'red'>defineProperty()</font>** 

defineProperty方法攔截了Object.defineProperty操作。  

``` js
var handler = {
  defineProperty (target, key, descriptor) {
    return false;
  }
};
var target = {};
var proxy = new Proxy(target, handler);
proxy.foo = 'bar'
// TypeError: proxy defineProperty handler returned false for property '"foo"'
```

上面代碼中，defineProperty方法返回false，導致添加新屬性會拋出錯誤。

注意，如果目標對像不可擴展（extensible），則defineProperty不能增加目標對像上不存在的屬性，否則會報錯。另外，如果目標對象的某個屬性不可寫（writable）或不可配置（configurable），則defineProperty方法不得改變這兩個設置。

#### **<font color = 'red'>getOwnPropertyDescriptor()</font>** 

getOwnPropertyDescriptor方法攔截Object.getOwnPropertyDescriptor，返回一個屬性描述對像或者undefined。  

``` js
var handler = {
  getOwnPropertyDescriptor (target, key) {
    if (key[0] === '_') {
      return;
    }
    return Object.getOwnPropertyDescriptor(target, key);
  }
};
var target = { _foo: 'bar', baz: 'tar' };
var proxy = new Proxy(target, handler);
Object.getOwnPropertyDescriptor(proxy, 'wat')
// undefined
Object.getOwnPropertyDescriptor(proxy, '_foo')
// undefined
Object.getOwnPropertyDescriptor(proxy, 'baz')
// { value: 'tar', writable: true, enumerable: true, configurable: true }
```

上面代碼中，handler.getOwnPropertyDescriptor方法對於第一個字符為下劃線的屬性名會返回undefined。

#### **<font color = 'red'>getPrototypeOf()</font>** 

getPrototypeOf方法主要用來攔截Object.getPrototypeOf()運算符，以及其他一些操作。

* Object.prototype.__proto__
* Object.prototype.isPrototypeOf()
* Object.getPrototypeOf()
* Reflect.getPrototypeOf()
* instanceof運算符

``` js
var proto = {};
var p = new Proxy({}, {
  getPrototypeOf(target) {
    return proto;
  }
});
Object.getPrototypeOf(p) === proto // true
```

上面代碼中，getPrototypeOf方法攔截Object.getPrototypeOf()，返回proto對象。

注意，getPrototypeOf方法的返回值必須是對像或者null，否則報錯。另外，如果目標對像不可擴展（extensible），getPrototypeOf方法必須返回目標對象的原型對象。

#### **<font color = 'red'>isExtensible()</font>** 

isExtensible方法攔截Object.isExtensible操作。  

``` js
var p = new Proxy({}, {
  isExtensible: function(target) {
    console.log("called");
    return true;
  }
});

Object.isExtensible(p)
// "called"
// true
```

注意，該方法只能返回布爾值，否則返回值會被自動轉為布爾值。

這個方法有一個強限制，它的返回值必須與目標對象的isExtensible屬性保持一致，否則就會拋出錯誤。

#### **<font color = 'red'>ownKeys()</font>** 

ownKeys方法用來攔截以下操作。

* Object.getOwnPropertyNames()
* Object.getOwnPropertySymbols()
* Object.keys()

下面是攔截Object.keys()的例子。

``` js
let target = {
  a: 1,
  b: 2,
  c: 3
};

let handler = {
  ownKeys(target) {
    return ['a'];
  }
};

let proxy = new Proxy(target, handler);

Object.keys(proxy)
// ['a']
```

上面代碼攔截了對於target對象的Object.keys()操作，只返回a、b、c三個屬性之中的a屬性。  

下面的例子是攔截第一個字符為下劃線的屬性名。  

``` js
let target = {
  _bar: 'foo',
  _prop: 'bar',
  prop: 'baz'
};

let handler = {
  ownKeys (target) {
    return Reflect.ownKeys(target).filter(key => key[0] !== '_');
  }
};

let proxy = new Proxy(target, handler);
for (let key of Object.keys(proxy)) {
  console.log(target[key]);
}
// "baz"
```

注意，使用Object.keys方法時，有三類屬性會被ownKeys方法自動過濾，不會返回。

* 目標對像上不存在的屬性
* 屬性名為Symbol 值
* 不可遍歷（enumerable）的屬性

``` js
let target = {
  a: 1,
  b: 2,
  c: 3,
  [Symbol.for('secret')]: '4',
};

Object.defineProperty(target, 'key', {
  enumerable: false,
  configurable: true,
  writable: true,
  value: 'static'
});

let handler = {
  ownKeys(target) {
    return ['a', 'd', Symbol.for('secret'), 'key'];
  }
};

let proxy = new Proxy(target, handler);

Object.keys(proxy)
// ['a']
```

上面代碼中，ownKeys方法之中，顯式返回不存在的屬性（d）、Symbol值（Symbol.for('secret')）、不可遍歷的屬性（key），結果都被自動過濾掉。  

---

ownKeys方法還可以攔截Object.getOwnPropertyNames()。

``` js
var p = new Proxy({}, {
  ownKeys: function(target) {
    return ['a', 'b', 'c'];
  }
});

Object.getOwnPropertyNames(p)
// [ 'a', 'b', 'c' ]
```

ownKeys方法返回的數組成員，只能是字符串或Symbol 值。如果有其他類型的值，或者返回的根本不是數組，就會報錯。  

``` js
var obj = {};

var p = new Proxy(obj, {
  ownKeys: function(target) {
    return [123, true, undefined, null, {}, []];
  }
});

Object.getOwnPropertyNames(p)
// Uncaught TypeError: 123 is not a valid property name
```

如果目標對象自身包含不可配置的屬性，則該屬性必須被ownKeys方法返回，否則報錯。  

``` js
var obj = {};
Object.defineProperty(obj, 'a', {
  configurable: false,
  enumerable: true,
  value: 10 }
);

var p = new Proxy(obj, {
  ownKeys: function(target) {
    return ['b'];
  }
});

Object.getOwnPropertyNames(p)
// Uncaught TypeError: 'ownKeys' on proxy: trap result did not include 'a'
```

另外，如果目標對像是不可擴展的（non-extensition），這時ownKeys方法返回的數組之中，必須包含原對象的所有屬性，且不能包含多餘的屬性，否則報錯。  

``` js
var obj = {
  a: 1
};

Object.preventExtensions(obj);

var p = new Proxy(obj, {
  ownKeys: function(target) {
    return ['a', 'b'];
  }
});

Object.getOwnPropertyNames(p)
// Uncaught TypeError: 'ownKeys' on proxy: trap returned extra keys but proxy target is non-extensible
```

#### **<font color = 'red'>preventExtensions()</font>** 

preventExtensions方法攔截Object.preventExtensions()。該方法必須返回一個布爾值，否則會被自動轉為布爾值。

這個方法有一個限制，只有目標對像不可擴展時（即Object.isExtensible(proxy)為false），proxy.preventExtensions才能返回true，否則會報錯。

``` js
var p = new Proxy({}, {
  preventExtensions: function(target) {
    return true;
  }
});

Object.preventExtensions(p) // 报错
```

上面代碼中，proxy.preventExtensions方法返回true，但這時Object.isExtensible(proxy)會返回true，因此報錯。

為了防止出現這個問題，通常要在proxy.preventExtensions方法裡面，調用一次Object.preventExtensions。  

``` js
var p = new Proxy({}, {
  preventExtensions: function(target) {
    console.log('called');
    Object.preventExtensions(target);
    return true;
  }
});

Object.preventExtensions(p)
// "called"
// true
```

#### **<font color = 'red'>setPrototypeOf()</font>** 

setPrototypeOf方法主要用來攔截Object.setPrototypeOf方法。

下面是一個例子。

``` js
var handler = {
  setPrototypeOf (target, proto) {
    throw new Error('Changing the prototype is forbidden');
  }
};
var proto = {};
var target = function () {};
var proxy = new Proxy(target, handler);
Object.setPrototypeOf(proxy, proto);
// Error: Changing the prototype is forbidden
```

上面代碼中，只要修改target的原型對象，就會報錯。

注意，該方法只能返回布爾值，否則會被自動轉為布爾值。另外，如果目標對像不可擴展（extensible），setPrototypeOf方法不得改變目標對象的原型。

---

## **Proxy.revocable()**

Proxy.revocable方法返回一個可取消的Proxy 實例。

``` js
let target = {};
let handler = {};

let {proxy, revoke} = Proxy.revocable(target, handler);

proxy.foo = 123;
proxy.foo // 123

revoke();
proxy.foo // TypeError: Revoked
```

Proxy.revocable方法返回一個對象，該對象的proxy屬性是Proxy實例，revoke屬性是一個函數，可以取消Proxy實例。上面代碼中，當執行revoke函數之後，再訪問Proxy實例，就會拋出一個錯誤。

Proxy.revocable的一個使用場景是，目標對像不允許直接訪問，必須通過代理訪問，一旦訪問結束，就收回代理權，不允許再次訪問。  

## **this問題**

雖然Proxy可以代理針對目標對象的訪問，但它不是目標對象的透明代理，即不做任何攔截的情況下，也無法保證與目標對象的行為一致。主要原因就是在Proxy代理的情況下，目標對象內部的this關鍵字會指向Proxy代理。  

``` js
const target = {
  m: function () {
    console.log(this === proxy);
  }
};
const handler = {};

const proxy = new Proxy(target, handler);

target.m() // false
proxy.m()  // true
```




























