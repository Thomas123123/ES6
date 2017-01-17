# 1.Symbol

---

> **ES5的objact property name都是string，這容易造成property name的衝突。**
* 如果有一種機制，promise all property name都是獨一無二的就好了，這樣就從根本上防止property name的衝突。這就是ES6引入Symbol的原因。  

In ES6 javascript variable type 多一種 : undefined、NULL、Number、String、Boolean、Object、<font color='red'>**Symbol**。  

Symbol value通過**Symbol function**生成，object property name現在可以有兩種類型，一種是原來就有的string，另一種就是新增的Symbol類型。凡是property name屬於Symbol類型，就都是獨一無二的。  

``` js
var sym = Symbol();

typeof sym     // "symbol"
```

---

``` js

ES5 
var obj = {
      name : "thomas", 
      age : 18 , 
      single : false 
    }
/*
    name : "thomas"
    age : 18
    single : false
*/

ES6
var _name = Symbol('name'),
    age = Symbol('age'),
    single = Symbol('single')

var obj = {
      [_name] : 'thomas' ,
      [age] : 18 ,
      [single] : false
    }
/*
    Symbol(name) : "thomas"
    Symbol(age) : 18
    Symbol(single) : false
*/


```

---

* 注意，Symbol function前不能使用new prefix，否則會報錯。這是因為生成的Symbol function是一個Primitive type的值，不是object。也就是說，由於Symbol is not object，所以不能添加property。基本上，它是一種類似於string type。  

Symbol function可以接受一個string作為argument，表示對Symbol實例的描述，主要是為了在console顯示，或者轉為string時，比較容易區分。  

``` js
var s1 = Symbol('foo');
var s2 = Symbol('bar');

s1 // Symbol(foo)
s2 // Symbol(bar)

s1.toString() // "Symbol(foo)"
s2.toString() // "Symbol(bar)"
```

上面code中，s1和s2是兩個Symbol value。如果不加argument，它們在console的output都是Symbol()，不利於區分。有了argument以後，就等於為它們加上了description，output的時候就能夠分清。  

如果Symbol function argument is a object，就會call this object的toString method，將其轉為string，然後才生成一個Symbol value。  

``` js
const obj = {
  toString() {
    return 'abc';
  }
};

const sym = Symbol(obj);
sym // Symbol(abc)
```

Symbol function的argument只是表示對當前Symbol value的description，因此相同argument的Symbol function的return value是不相等的。  

``` js
// 沒有argument
var s1 = Symbol();
var s2 = Symbol();

s1 === s2 // false

// 有argument
var s1 = Symbol('foo');
var s2 = Symbol('foo');

s1 === s2 // false
```

### Symbol value不能與其他類型的值進行運算，會報錯。  

``` js
var sym = Symbol('My symbol');

"your symbol is " + sym
// TypeError: can't convert symbol to string
```

但是，Symbol value可以顯示轉為string。  

``` js
var sym = Symbol('My symbol');

String(sym) // 'Symbol(My symbol)'
sym.toString() // 'Symbol(My symbol)'
```

另外，Symbol值也可以轉為boolean，但是不能轉為number。  

``` js
var sym = Symbol();
Boolean(sym) // true
!sym         // false

Number(sym)  // Uncaught TypeError: Cannot convert a Symbol value to a number
sym + 2      // Uncaught TypeError: Cannot convert a Symbol value to a number
```

### **作為 property name Symbol**  

由於每一個Symbol value都是不相等的，這意味著Symbol value可以作為Identifiers，用於object property name，就能保證不會出現同名的屬性。這對於一個objecr由多個module構成的情況非常有用，能防止某一個chain被不小心改寫或覆蓋。  

``` js
var mySymbol = Symbol();

// 第一种写法
var a = {};
a[mySymbol] = 'Hello!';

// 第二种写法
var a = {
  [mySymbol]: 'Hello!'
};

// 第三种写法
var a = {};
Object.defineProperty(a, mySymbol, { value: 'Hello!' });

// 以上写法都得到同样结果
a[mySymbol] // "Hello!"
```

Symbol value作為object property name時，不能用點運算符。  

``` js
var mySymbol = Symbol();
var a = {};

a.mySymbol = 'Hello!';
a[mySymbol] // undefined
a['mySymbol'] // "Hello!"
```

### **Instance：remove magic string**

Magic string the mean，在code之中多次出現、與code形成strong coupling某一個具體的string或者number。風格良好的code，應該盡量remove magic string。

``` js
function getData(maxwin) {
  switch (maxwin) {
    case 'ks': // magic string
      return '高雄'
      break;
    /* ... more code ... */
  }
}

getData('ks'); // magic string
```

常用的remove magic string的方法，就是把它寫成一個variable。  

``` js
var companyAd = {
  ks : 'ks'
};

function getArea(maxwin) {
  switch (maxwin) {
    case companyAd.ks:
      return '高雄'
      break;
  }
}

getArea(companyAd.ks);
```

可以發現companyAd.ks等於哪個value並不重要，只要確保不會跟其他companyAd property value衝突即可。因此，這裡就很適合改用Symbol value

``` js
  const companyAd = {
    ks : Symbol()
  };
```

### **property name iteration**

Symbol作為property name，該屬性不會出現在<font color = 'red'>for...in</font>、<font color = 'red'>for...of</font>循環中，也不會被<font color = 'red'>Object.keys()</font>、<font color = 'red'>Object.getOwnPropertyNames()</font>、<font color = 'red'>JSON.stringify()</font> return。但是，有一個<font color = 'red'>Object.getOwnPropertySymbols</font>方法，可以獲取指定object的所有Symbol屬性名。  

<font color = 'red'>Object.getOwnPropertySymbols</font>方法return an array，成員是當前object的所有用symbol value作property name的都會被取出。  

``` js
    var obj = {};
    var a = Symbol('a');
    var b = Symbol('b');

    obj[a] = 'Hello';
    obj[b] = 'World';

    var objectSymbols = Object.getOwnPropertySymbols(obj);

    objectSymbols
    // [Symbol(a), Symbol(b)]
```

<font color = 'red'>Reflect.ownKeys</font>方法可以返回所有類型的property name，包括常規name和Symbol value。  

``` js
    var a = Symbol('a');
    var obj = {
          [a]: 1,
          enum: 2,
          nonEnum: 3
        };

    Reflect.ownKeys(obj)   // ["enum", "nonEnum", Symbol(a)]
```

