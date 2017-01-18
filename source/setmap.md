# 2.set&map

---
>ES6提供了新的數據結構Set。它類似於數組，但是成員的值都是唯一的，沒有重複的值。

Set本身是一個構造函數，用來生成Set數據結構。  

``` js
var set = new Set()
```

**Set結構不會添加重複的值**
Set function可以接受一個array（或類似array的object）作為argument，用來初始化。

``` js
// 例一
var set = new Set([1, 2, 3, 4, 4]);
[...set]
// [1, 2, 3, 4]

// 例二
var items = new Set([1, 2, 3, 4, 5, 5, 5, 5]);
items.size // 5

// 例三
function divs () {
  return [...document.querySelectorAll('div')];
}

var set = new Set(divs());
set.size 
```

set也可以用來當去除重複數值的方法

``` js
var array = [1,2,3,3,3,3,4,5];

[...new Set(array)] 
//[1, 2, 3, 4, 5]
```

Set加入value的時候，不會發生類型轉換，所以1和"1"是兩個不同的value。Set內部判斷兩個值是否不同，使用的算法叫做“Same-value equality”，它類似於精確相等運算符（===）。  

``` js
var set = new Set();
set.add(1);
set.add(2);
set.add('1');
set.add('2');

set ;
// [1,2,'1','2']
```

兩個object總是不相等的

``` js
let set = new Set();

set.add({});
set.size // 1

set.add({});
set.size // 2
```

### **Set instance method**  

**Set instance method分為兩大類：操作方法（用於操作數據）和遍歷方法（用於遍歷成員）。**

下面先介紹四個操作方法。  

<font color = 'red'>add(value)</font>：添加某個value。
<font color = 'red'>delete(value)</font>：刪除某個value，return a boolean，表示刪除是否成功。
<font color = 'red'>has(value)</font>：return a boolean，表示該值是否為Set的成員。
<font color = 'red'>clear()</font>：清除所有成員。

Array.from方法可以將Set結構轉為數組。

``` js
function dedupe(array) {
  return Array.from(new Set(array));
}

dedupe([1, 1, 2, 3]) // [1, 2, 3]
```

再來介紹四個遍歷方法。  

<font color = 'red'>keys()</font>：返回鍵名的遍歷器
<font color = 'red'>values()</font>：返回鍵值的遍歷器
<font color = 'red'>entries()</font>：返回鍵值對的遍歷器
<font color = 'red'>forEach()</font>：使用回調函數遍歷每個成員


**（1）keys()，values()，entries()**

keys方法、values方法、entries方法返回的都是iterator object。由於Set結構沒有key name，只有key value（或者說key name和key value是同一個值），所以keys方法和values方法的行為完全一致。  

``` js
let set = new Set(['red', 'green', 'blue']);

for (let item of set.keys()) {
  console.log(item);
}
// red
// green
// blue

for (let item of set.values()) {
  console.log(item);
}
// red
// green
// blue

for (let item of set.entries()) {
  console.log(item);
}
// ["red", "red"]
// ["green", "green"]
// ["blue", "blue"]
```

上面code中，entries方法返回的iterator，同時包括key name和key value，所以每次輸出一個array，它的兩個成員完全相等。  

**（2）forEach()**

Set結構的實例的forEach方法，用於對每個成員執行某種操作。

``` js
let set = new Set([1, 2, 3]);
set.forEach((value) => console.log(value * 2) )
// 2
// 4
// 6
```

**（3）遍歷的應用**

擴展運算符（...）內部使用for...of循環，所以也可以用於Set結構。  

``` js
let set = new Set(['red', 'green', 'blue']);
let arr = [...set];
// ['red', 'green', 'blue']
```

數組的map和filter方法也可以用於Set了。  

``` js
let set = new Set([1, 2, 3]);
set = new Set([...set].map(x => x * 2));
// Set：{2, 4, 6}

let set = new Set([1, 2, 3, 4, 5]);
set = new Set([...set].filter(x => (x % 2) == 0));
// Set：{2, 4}
```

因此使用Set可以很容易地實現並集（Union）、交集（Intersect）和差集（Difference）。  

``` js
let a = new Set([1, 2, 3]);
let b = new Set([4, 3, 2]);

// union
let union = new Set([...a, ...b]);
// Set {1, 2, 3, 4}

// intersect
let intersect = new Set([...a].filter(x => b.has(x)));
// set {2, 3}

// difference
let difference = new Set([...a].filter(x => !b.has(x)));
// Set {1}
```

---

# **Map**

JavaScript的Object，本質上是鍵值對的集合（Hash結構），但是傳統上只能用string當作key name。這給它的使用帶來了很大的限制。  

``` js
var data = {};
var element = document.getElementById('myDiv');

data[element] = 'metadata';
data['[object HTMLDivElement]'] // "metadata"
```

上面code是將一個DOM node作為object data的key name，但是由於object只接受string作為key name，所以element被自動轉為string[object HTMLDivElement]  

ES6提供了Map數據結構。它類似於object。
Object結構提供了“string—value”的mapping，Map結構提供了“value—value”的mapping，是一種更完善的Hash結構instance。  

``` js
var m = new Map();
var o = {p: 'Hello World'};

m.set(o, 'content')
m.get(o) // "content"

m.has(o) // true
m.delete(o) // true
m.has(o) // false
```

上面code使用set方法，將object o當作m的一個key name，然後又使用get方法讀取這個key name，接著使用delete方法刪除了這個key name。

* 若argument為array,則可以利用foreach iterator將array遍歷進去map裡

``` js
var items = [
  ['name', 'thomas'],
  ['title', 'javascript']
];
var map = new Map();
items.forEach(([key, value]) => map.set(key, value));
```

如果對同一個鍵多次賦值，後面的值將覆蓋前面的值。  

``` js
let map = new Map();

map
.set(1, 'aaa')
.set(1, 'bbb');

map.get(1) // "bbb"
```

如果讀取一個未知的鍵，則返回undefined。  

``` js
new Map().get('aaa')
// undefined
```

如果Map的key name是一個primitive value（number、string、boolean），則只要兩個值嚴格相等(===)，Map將其視為一個key name，包括0和-0。  

* **雖然NaN不嚴格相等於自身，但Map將其視為同一個key name。**

``` js
let map = new Map();

map.set(NaN, 123);
map.get(NaN) // 123

map.set(-0, 123);
map.get(+0) // 123
```

---

### **instance property and method**

method

``` js
var map = new Map();

map.set(key name, key value)  // 設定name跟value
map.get(key name)    // 取得key value
map.has(key name)    // 判斷key name是否存在
map.delete(key name) // 刪除指定key name
map.clear() // 刪除全部key name
```

遍歷方法跟set一樣

<font color = 'red'>keys()</font>：return key name的遍歷器
<font color = 'red'>values()</font>：return key value的遍歷器
<font color = 'red'>entries()</font>：return [key name,key value]的遍歷器
<font color = 'red'>forEach()</font>：遍歷Map的所有成員

``` js
let map = new Map([
  ['F', 'no'],
  ['T',  'yes'],
]);

for (let key of map.keys()) {
  console.log(key);
}
// "F"
// "T"

for (let value of map.values()) {
  console.log(value);
}
// "no"
// "yes"

for (let item of map.entries()) {
  console.log(item[0], item[1]);
}
// "F" "no"
// "T" "yes"
```

Map結構轉為數組結構，比較快速的方法是結合使用擴展運算符（...）。

``` js
let map = new Map([
  [1, 'one'],
  [2, 'two'],
  [3, 'three'],
]);

[...map.keys()]
// [1, 2, 3]

[...map.values()]
// ['one', 'two', 'three']

[...map.entries()]
// [[1,'one'], [2, 'two'], [3, 'three']]

[...map]
// [[1,'one'], [2, 'two'], [3, 'three']]
```

結合array的map方法、filter方法，可以實現Map的遍歷和過濾（Map本身沒有map和filter方法）。


``` js
let map0 = new Map()
  .set(1, 'a')
  .set(2, 'b')
  .set(3, 'c');

let map1 = new Map(
  [...map0].filter(([k, v]) => k < 3)
);
// Map {1 => 'a', 2 => 'b'}

let map2 = new Map(
  [...map0].map(([k, v]) => [k * 2, '_' + v])
    );
// Map {2 => '_a', 4 => '_b', 6 => '_c'}
```

