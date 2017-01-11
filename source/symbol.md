# 1.Symbol

---

> **ES5的objact property name都是string，這容易造成property name的衝突。**
* 如果有一種機制，promise all property name都是獨一無二的就好了，這樣就從根本上防止property name的衝突。這就是ES6引入Symbol的原因。  

In ES6 javascript variable type 多一種 : undefined、NULL、Number、String、Boolean、Object、<font color='red'>**Symbol**。  

Symbol值通過**Symbol函數**生成，object property name現在可以有兩種類型，一種是原來就有的string，另一種就是新增的Symbol類型。凡是property name屬於Symbol類型，就都是獨一無二的。  

---

``` js

ES5 
var obj = {name : "thomas" , age : 18 , single : false}
/*
    name : "thomas"
    age : 18
    single : false
*/

ES6
var _name = Symbol('name'),
    age = Symbol('age'),
    single = Symbol('single'),
    obj = {}

obj[_name] = 'thomas'
obj[age] = 18
obj[single] = false
/*
    Symbol(name) : "thomas"
    Symbol(age) : 18
    Symbol(single) : false
*/


```

---

``` js
var sym = Symbol();

typeof sym     // "symbol"
```

* 注意，Symbol function前不能使用new prefix，否則會報錯。這是因為生成的Symbol function是一個Primitive type的值，不是object。也就是說，由於Symbol is not object，所以不能添加property。基本上，它是一種類似於string type。  

Symbol function可以接受一個string作為argument，表示對Symbol實例的描述，主要是為了在控制台顯示，或者轉為字符串時，比較容易區分。  

``` js
var s1 = Symbol('foo');
var s2 = Symbol('bar');

s1 // Symbol(foo)
s2 // Symbol(bar)

s1.toString() // "Symbol(foo)"
s2.toString() // "Symbol(bar)"
```

上面code中，s1和s2是兩個Symbol value。如果不加argument，它們在console的output都是Symbol()，不利於區分。有了argument以後，就等於為它們加上了description，output的時候就能夠分清。  

如果Symbol function的argument is a object，就會invoke this object的toString method，將其轉為string，然後才生成一個Symbol值。  

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

另外，Symbol值也可以轉為布爾值，但是不能轉為數值。  

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

Symbol值作為對象屬性名時，不能用點運算符。  

``` js
var mySymbol = Symbol();
var a = {};

a.mySymbol = 'Hello!';
a[mySymbol] // undefined
a['mySymbol'] // "Hello!"
```




