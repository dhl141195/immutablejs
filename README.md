# Table of Contents

- [Pros and cons of ImmutableJS](#pros-and-cons-of-immutablejs)
  - [Pros](#pros)
    - [Gets rid of unnecessary data copying](#gets-rid-of-unnecessary-data-copying)
    - [Easy change detection](#easy-change-detection)
    - [Mutate tracking](#mutate-tracking)
    - [Undo/redo](#undoredo)
  - [Cons](#cons)
    - [Syntax for getting and setting is kind of ugly](#syntax-for-getting-and-setting-is-kind-of-ugly)
    - [Cumbersome to debug code](#cumbersome-to-debug-code)
    - [Less performant than mutable approach with small datasets](#less-performant-than-mutable-approach-with-small-datasets)
- [How ImmutableJS work](#how-immutablejs-work)
  - [Steps to edit a node](#steps-to-edit-a-node)
- [How to use ImmutableJS](#how-to-use-immutablejs)
  - [Type Definition](#type-definition)
  - [List](#list)
    - [Construction](#construction)
  - [Map](#map)
    - [Construction](#construction)
  - [Set](#set)
    - [Construction](#construction)
  - [Stack](#stack)
    - [Construction](#construction)
  - [Record](#record)
    - [Construction](#construction)
    - [Extending Record](#extending-record)
  - [Seq](#seq)
    - [Seq is lazy](#seq-is-lazy)
    - [Construction](#construction)
  - [Hightlight methods](#hightlight-methods)
    - [All methods example](#all-methods-example)
    - [update()](#update)
    - [setIn()](#setin)
    - [withMutations()](#withmutations)
    - [flatMap()](#flatmap)
    - [zip()](#zip)
    - [zipAll()](#zipall)
    - [zipWith()](#zipwith)
    - [interpose()](#interpose)
    - [interleave()](#interleave)

# Pros and cons of ImmutableJS

## Pros

### Gets rid of unnecessary data copying

```setState()``` của React nhận vào 1 object. Object này sẽ được merge với state hiện tại. Nhưng chỉ merge được ở level đầu tiên của object. Nếu muốn thay đổi từ level thứ 2 trở đi thì ta phải copy các phần data cần giữ lại. Thông thường sẽ dùng ```spread```, ```Object.asign()```. Code sẽ rối nếu state lồng nhiều lớp. Hoặc cũng có thể dùng deep copy của 1 số lib như lodash, nhưng như vậy sẽ giảm hiệu năng do phải copy toàn bộ object

Redux yêu cầu k được mutate state để đảm bảo performance (detect state change thông qua reference) cũng như phục vụ tính năng time-travel debugging. Vì vậy nếu state của Redux mà lồng nhiều lớp, cũng phải copy data y hệt như ```setState()``` của React

> Sử dụng ImmutableJS cho phép deep copy 1 cách ngắn gọn và hiệu quả. Chỉ những phần cần thay đổi mới được copy, còn lại sẽ được tái sử dụng

### Easy change detection

Dễ dàng check được khi nào data thay đổi. Bởi vì immutable data thay đổi thì reference sẽ thay đổi. Chỉ cần check reference thay vì deep equal sâu bên trong object.

> Giúp cải thiện hiệu năng. Có thể dùng kết hợp với **shouldComponentUpdate** hoặc **React.PureComponent** để tăng hiệu năng cho React app

### Mutate tracking

Khi cấu trúc data phức tạp, mutate data có thể sinh ra nhiều bug khó track. ImmutableJS đảm bảo rằng data luôn luôn immutable. Mặc dù Redux yêu cầu data phải immutable nhưng chỉ là convention. Data vẫn có thể bị lấy ra từ store, truyền đi khắp nơi qua reference và bị mutate ở đó.

```js
// in mapStateToProps
return persion = getPeron(state) // {name: 'DHL', age: 22'}

// in component A
const { person } = this.props;
person.name = 'Supa DHL';

// in component B
const { person } = this.props;
this.props.actions.updatePerson({
  ...person
  age: 23
});

// Bug: expect name === 'DHL' but it has been mutated in component A
```

### Undo/redo

Dễ dàng implement undo/redo bằng cách cache reference

## Cons

### Syntax for getting and setting is kind of ugly

Get/set dùng string. Dễ lỗi typo

### Cumbersome to debug code

Ngoài data, Immutable structure còn wrap thêm nhiều thứ. Vì vậy khi debug, data thường khó đọc, phải dùng ```toJS()``` trên console

### Less performant than mutable approach with small datasets

Trong trường hợp cấu trúc data nhỏ, flat, không cần copy nhiều, sử dụng ImmutableJS sẽ không mang lại nhiều lợi ích, đôi khi làm giảm hiệu năng. Nên thường chỉ áp dụng cho những cấu trúc data lớn, lồng nhiều lớp.

# How ImmutableJS work

ImmutableJS sử dụng model **Structure Sharing** hay còn gọi là **Directed Acyclic Graph (DAG)** để tối ưu việc copy data. Chỉ có những phần data cần mutate mới được copy, còn lại sẽ được tái sử dụng

DAG:

<img src="image/DAG.png" width="300" height="300" />

## Steps to edit a node

Chọn 1 node để edit

<img src="image/step1.png" width="300" height="300" />

Copy node đó

<img src="image/step2.png" width="300" height="300" />

Node cha phải trỏ tới node con mới

<img src="image/step3.png" width="300" height="300" />

Nhưng vì k mutate nên copy node cha

<img src="image/step4.png" width="300" height="300" />

Tiếp tục copy cho đến khi tới root

<img src="image/step5.png" width="300" height="300" />

Kết quả

<img src="image/step6.png" width="300" height="300" />

> Như ta có thể thấy, chỉ những phần data cần mutate mới được copy, còn lại sẽ được tái sử dụng
>
> ImmutableJS implement object và array của javascript thành DAG model bằng cách sử dụng ```hash maps tries``` và ```vector tries```
>
> [More about this](https://www.youtube.com/watch?v=I7IdS-PbEgI)

# How to use ImmutableJS

## Type Definition

List<T>: immutable list các phần tử với kiểu dữ liệu T bất kỳ

Map<K, V>: immutable map với mỗi phần tử là key-value pair có kiểu dữ liệu bất kì

Iterable: ES6 Iterable, có property Symbol.Iterator và dùng được for...of. Mọi Immutable Collection đều là Iterable.

Iterable<T>: List<T>, native Array...

Iterable<K, V>: Map<K, V>, OrderedMap<K, V>...

## List

> ```class List<T> extends Collection.Indexed<T>```
>
> Immutable List tương tự Array. Nó có hầu hết các method của Array. Chỉ khác là các method bình thường mutate Array như push, splice... sẽ trả về 1 Immutable List mới

### Construction

```js
List(): List<any>
List<T>(): List<T>
List<T>(collection: Iterable<T>): List<T>
```

Thông thường List được khởi tạo bằng **Array** hoặc bằng 1 **Immutable Collection** khác (vì mọi Immutable collection đều là Iterable)

Example:

```js
const { List, Set } = require('immutable')

const emptyList = List()
// List []

const plainArray = [ 1, 2, 3, 4 ]
const listFromPlainArray = List(plainArray)
// List [ 1, 2, 3, 4 ]

const plainSet = Set([ 1, 2, 3, 4 ])
const listFromPlainSet = List(plainSet)
// List [ 1, 2, 3, 4 ]

const arrayIterator = plainArray[Symbol.iterator]()
const listFromCollectionArray = List(arrayIterator)
// List [ 1, 2, 3, 4 ]
```

## Map

> ```class Map<K, V> extends Collection.Keyed<K, V>```

### Construction

```js
Map<K, V>(collection: Iterable<[K, V]>): Map<K, V>
// Map(OrderedMap({a:1, b:2}))

Map<T>(collection: Iterable<Iterable<T>>): Map<T, T>
// Map([['key1', 'value1'], ['key2', 'value2']])

Map<V>(obj: {[key: string]: V}): Map<string, V>
// Map({ a: 1, b: 2 });

Map<K, V>(): Map<K, V>
Map(): Map<any, any>
```

## Set

> ```class Set<T> extends Collection.Set<T>```
>
> Value trong set luôn uniquie. Value được so sánh bằng ```Immutable.is()```.
> Nhưng đối với plain javascript object thì so sánh ```===```, nên 2 object luôn khác nhau

### Construction

```js
Set(): Set<any>
Set<T>(): Set<T>
Set<T>(collection: Iterable<T>): Set<T>
// Set([1, 1, 2, 3])
// Set(Map({a: 1, b: 2}))
```

## Stack

> ```class Stack<T> extends Collection.Indexed<T>```

```unshift()```, ```shift()```, ```push()```, ```pop()```, ```peek()```: O(1)

Note:

- ```push()```, ```pop()``` của Stack là push/pop vào đầu Stack, không như array và List.
- Không nên dùng các function đảo ngược thứ tự như reverse, reduceRight, lastIndexOf... với Stack.

### Construction

```js
Stack(): Stack<any>
Stack<T>(): Stack<T>
Stack<T>(collection: Iterable<T>): Stack<T>
// Stack([1, 1, 2, 3])
// Stack(Map({a: 1, b: 2}))
```

## Record

> ```class Record<TProps>```
>
> Record giống object nhưng xác định trước các key cho phép và có default value

Note:

- ```remove(key)``` chỉ đưa key đó về default value
- Nếu cần serialize Record thành raw data và convert ngược raw data thành Record thường xuyên thì nên dùng object thay vì dùng Record

### Construction

```js
const { Record } = require('immutable')

// Record() trả về Record Factory
const ABRecord = Record({ a: 1, b: 2 })

// Tạo ínstance từ Record Factory
const myRecord = new ABRecord({ b: 3 })
```

### Extending Record

Extend Record để thêm custom method cho Record

```js
class ABRecord extends Record({ a: 1, b: 2 }) {
  getAB() {
    return this.a + this.b;
  }
}

var myRecord = new ABRecord({b: 3})
myRecord.getAB() // 4
```

## Seq

> ```class Seq<K, V> extends Collection<K, V>```
>
> Seq dùng để **construct a lazy operation**, cho phép nối các method của các collection mà không cần tạo collection trung gian

```js
const l = List([1, 2, 3]);

// Tạo 1 collection trung gian khi filter, tạo 1 collection trung gian khi map
const l2 = l.filter(v => v % 2 === 1).map(v => v * v);
```

### Seq is lazy

Seq chỉ bắt đầu thực thi khi kết quả trả về của nó được dùng. Ví dụ sau Seq không thực thi gì cả vì kết quả trả về không được dùng tới

```js
const { Seq } = require('immutable')
const oddSquares = Seq([ 1, 2, 3, 4, 5, 6, 7, 8 ])
  .filter(x => x % 2 !== 0)
  .map(x => x * x)

// oddSquares is not used
```

### Construction

```js
Seq<S>(seq: S): S // return same kind Seq
Seq<K, V>(collection: Collection.Keyed<K, V>): Seq.Keyed<K, V> // e.g: Map...
Seq<T>(collection: Collection.Indexed<T>): Seq.Indexed<T> // e.g: List...
Seq<T>(collection: Collection.Set<T>): Seq.Set<T> // // e.g: Set...
Seq<T>(collection: Iterable<T>): Seq.Indexed<T> // e.g: List, Array...
Seq<V>(obj: {[key: string]: V}): Seq.Keyed<string, V> // e.g: object
Seq(): Seq<any, any>
```

Seq chỉ thực hiện khối lượng công việc nhỏ nhất để trả về kết quả được yêu cầu. Trong ví dụ này, không cần tới mảng trung gian nào, function truyền vào ```filter``` được gọi 3 lần. function truyền vào ```map``` chỉ gọi 1 lần

```js
oddSquares.get(1); // oddSquares từ ví dụ trên
```

## Hightlight methods

### All methods example

<https://codefiddle.wordpress.com/2016/10/21/immutablejs-examples-for-humans/>

### update()

```update()``` có 3 overload

> ```update(index: number, updater: (value: T) => T): this```:

Nhận vào index và function dùng để update value tại index đó

```js
const list = List([ 'a', 'b', 'c' ])
const result = list.update(2, val => val.toUpperCase())
// List [ "a", "b", "C" ]
```

> ```update(index: number, notSetValue: T, updater: (value: T) => T): this```:

Nhận vào index, notSetValue, và function dùng để update value tại index đó.

Nếu index đó chưa được set value (index >= size) thì ```notSetValue``` sẽ được truyền vào function updater và giá trị trả về sẽ được set cho index (giống ```set()``` với index >= size)

```js
const list = List(['a', 'b', 'c'])
const result = list.update(4, 'd', val => val.toUpperCase())
// List [ "a", "b", "c", undefined, "D" ]
```

> ```update<R>(updater: (value: this) => R): R```

Chỉ nhận vào updater function. List gọi function ```update()``` sẽ được truyền vào cho updater function

```js
List([ 1, 2, 3 ]).update(collection => {
  return collection.reduce((sum, x) => sum + x, 0)
})
// 6
```

### setIn()

> ```setIn(keyPath: Iterable<any>, value: any): this```

Nếu key trong keyPath không tồn tại và key đó k phải key cuối cùng thì 1 Immutable Map sẽ được tạo tại vị trí đó. Còn nếu key đó là key cuối cùng thì chỉ set value bình thường

NOTE: Từ version 4.0.0, có thể set property trực tiếp cho plain object. Còn các phiên bản trước, các node trước node đích bắt buộc phải là Immutble Collection

```js
// Node trước node đích là plain object -> chỉ dùng được từ v4.0.0 trở đi
const list = List([ 0, 1, 2, { plain: 'object' }])
list.setIn([3, 'plain'], 'value');
// Error: invalid keyPath (v3.8)

// Node trước node đích là Immutable Map
onst list = List([ 0, 1, 2, Map({ plain: 'object' })])
list.setIn([3, 'plain'], 'value');
// Correct
```

### withMutations()

> ```withMutations(mutator: (mutable: this) => any): this```

```withMutations()``` dùng khi ta muốn thực hiện liên tiếp nhiều update trên 1 immutable Collection

Ví du:

```js
// This line will create 3 clones of List
List([1, 2, 3]).setSize(1).push(4).shift(5)

// Instead of that, we can use withMutations()
List([1, 2, 3]).withMutations(list => {
  list.setSize(1);
  list.push(4);
  list.shift(5);
  return list;
})
// Only create 1 clone in the end
```

### flatMap()

Tương tự ```map()``` nhưng callback function trả về Array hoặc Collection. Các array này sẽ được **flat** và merge thành kết quả cuối cùng

```js
var list = List([1, 2, 3]);
list.flatMap(v => [v,v * 2]);
// List [ 1, 2, 2, 4, 3, 6 ]
```

### zip()

Nhận vào Collection, 2 Collection hoặc mảng Collection (3 overload)

```zip()``` ghép phần tử cùng index của các Collection thành 1 mảng và trả về List gồm tất cả các mảng này

Note: size của List trả về sẽ bằng size nhỏ nhất trong số các Collection tham số và Collection gọi hàm

```js
var list = List([1, 2]);
list.zip(['one','two', 'three'], ['I', 'II', 'III', 'IV']);
// List[ [1, one, I], [2 ,two, II ] ] -> Collection min size = 2 -> size List = 2
```

### zipAll()

Giống ```zip()``` nhưng thay vì chọn ra size nhỏ nhất thì chọn ra size lớn nhất

```js
var list = List([1, 2]);
list.zip(['one','two', 'three'], ['I', 'II', 'III', 'IV']);
// List[ [1, one, I], [2 ,two ,II ], [undefined, three, I], [undefined ,undefined, IV] ]
// -> Collection max size = 4 -> size List = 4
```

### zipWith()

Thay vì dùng **default zipper** như ```zip()``` và ```zipAll()``` thì ```zipWith()``` nhận vào 1 **zipper function** để định nghĩa quá trình ghép các phần tử của các Collection

```js
const a = List([ 1, 2, 3, 10]);
const b = List([ 4, 5, 6 ]);
const c = a.zipWith((a, b) => a + b, b);
// List [ 5, 7, 9 ]
```

Note: **zipper function** sẽ được gọi và truyền vào các phần tử cùng index của tất cả các Collection tham gia zip cho đến khi 1 Collection hết phần tử

### interpose()

Nhận vào **seperator** và **xen giữa** seperator vào giữa mỗi phần tử

```js
var list = List([ 1,2,3, ]);
list.interpose(0);
// [1, 0, 2, 0, 3, 0, 4]
```

### interleave()

Nhận vào **Collection** và **xen kẽ** các phần tử của 2 Collection

```js
var list = List([ 1,2,3,4 ]);
list.interleave(List(['A','B','C','D', 'E', 'F']))
// [1, "A", 2, "B", 3, "C", 4, "D"]
```

Note: interleave sẽ dừng khi 1 trong 2 Collection hết data
