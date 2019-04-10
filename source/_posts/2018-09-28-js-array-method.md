---
layout: post
title: JavaScript - 常用数组操作方法
date: 2018-09-28
categories: JavaScript
tags: [JavaScript, 前端]
description: JavaScript常用数组操作方法，包含ES6方法
---

# ES6

## Array.of()

用于将一组值，转换为数组。这个方法的主要目的，是弥补数组构造函数 Array() 的不足。因为参数个数的不同，会导致 Array() 的行为有差异。

```javascript
Array(3) //[,,,]
Array.of(3) //[3]
Array(1, 2, 3) //[1,2,3]
Array.of(1, 2, 3) //[1,2,3] 多于2个参数，组成新数组
```

### includes()

判断数组中是否存在该元素，参数：查找的值、起始位置，可以替换 ES5 时代的 indexOf 判断方式。indexOf 判断元素是否为 NaN，会判断错误。

```
[1, 2, 3].includes(2) // true
[1, 2, 3].includes(4) // false
[1, 2, NaN].includes(NaN) // true
[1, 2, 3].includes(3, 3) // false
[1, 2, 3].includes(3, -1) // true
[NaN].includes(NaN) //true
```

### Array.from()

用于将两类对象转为真正的数组：类似数组的对象（array-like object）和可遍历（iterable）的对象（包括 ES6 新增的数据结构 Set 和 Map）。

```javascript
Array.from('foo') //['f','o','o']
```

### find() findIndex()

`find`方法用于找出第一个符合条件的数组成员。它的参数是一个回调函数，所有数组成员依次执行该回调函数，直到找出第一个返回值为`true`的成员，然后返回该成员。如果没有符合条件的成员，则返回 undefined。

```
[1, 4, -5, 10].find(n => n < 0) // 返回第一个满足条件的成员
// -5
```

`findIndex`方法的用法与 find 方法非常类似，返回第一个符合条件的数组成员的位置，如果所有成员都不符合条件，则返回-1

```
[1, 5, 10, 15].findIndex(function(value, index, arr) {
  return value > 9
}) // 2
```

这两个方法都可以接受第二个参数，用来绑定回调函数的 this 对象。

```javascript
function f(v) {
  return v > this.age
}
let person = { name: 'John', age: 20 }
;[10, 12, 26, 15].find(f, person) // 26
```

上面的代码中，find 函数接收了第二个参数 person 对象，回调函数中的 this 对象指向 person 对象。
另外，这两个方法都可以发现 `NaN`，弥补了数组的 `indexOf` 方法的不足。

```javascript
const arr = [NAN]
arr.indexOf(NaN)
// -1
arr.findIndex(y => Object.is(NaN, y))
// 0
```

上面代码中，indexOf 方法无法识别数组的 NaN 成员，但是 findIndex 方法可以借助 Object.is 方法做到。

### fill()

方法使用给定值，填充一个数组。

```
['a', 'b', 'c'].fill(1) // [1, 1, 1]
new Array(3).fill(1) // [1, 1, 1]
//上面代码表明，fill方法用于空数组的初始化非常方便。数组中已有的元素，会被全部抹去。
```

fill 方法还可以接受第二个和第三个参数，用于指定填充的起始位置和结束位置。

```
['a', 'b', 'c'].fill(7, 1, 2) // ['a', 7, 'c']
//上面代码表示，fill方法从 1 号位开始，向原数组填充 7，到 2 号位之前结束。
```

注意，如果填充的类型为对象，那么被赋值的是同一个内存地址的对象，而不是深拷贝对象。

```javascript
let arr = new Array(3).fill({ name: 'Mike' })
arr[0].name = 'Ben'
arr
// [{name: "Ben"}, {name: "Ben"}, {name: "Ben"}]

let arr = new Array(3).fill([])
arr[0].push(5)
arr
// [[5], [5], [5]]
```

### copyWithin()

在当前数组内部，将指定位置的成员复制到其他位置（会覆盖原有成员），然后返回当前数组。会修改当前数组。

```
[1, 2, 3, 4, 5].copyWithin(0, 3) // [4, 5, 3, 4, 5]
[1, 2, 3, 4, 5].copyWithin(0, 3, 4) // [4, 2, 3, 4, 5]
[1, 2, 3, 4, 5].copyWithin(0, -2, -1) //[4, 2, 3, 4, 5]
```

### flat() flatMap()

数组的成员有时还是数组，Array.prototype.flat()用于将嵌套的数组“拉平”，变成一维的数组。该方法返回一个新数组，对原数据没有影响。

```
[1, 2, [3, 4]].flat() // [1, 2, 3, 4]
[1, 2, [3, [4, 5]]].flat(2) // [1, 2, 3, 4, 5]
//flat()默认只会“拉平”一层，如果想要“拉平”多层的嵌套数组，可以将flat()方法的参数写成一个整数，表示想要拉平的层数，默认为1。
[1, [2, [3]]].flat(Infinity) // [1, 2, 3] 不管有多少层嵌套，都要转成一维数组
[1, 2, , 4, 5].flat() // [1, 2, 4, 5]会跳过空位。
```

flatMap()方法对原数组的每个成员执行一个函数（相当于执行 Array.prototype.map()），然后对返回值组成的数组执行 flat()方法。该方法返回一个新数组，不改变原数组。

```
// 相当于 [[2, 4], [3, 6], [4, 8]].flat()
[2, 3, 4].flatMap(x => [x, x * 2])
// [2, 4, 3, 6, 4, 8]

// 相当于 [[[2]], [[4]], [[6]], [[8]]].flat()
  [(1, 2, 3, 4)].flatMap(x => [[x * 2]])
// [[2], [4], [6], [8]]
```

## ES5

#### concat()

concat()方法用于`连接`两个或多个数组。方法不会改变现在的数组，仅返回被连接生成的新数组。

```javascript
var arr1 = [1, 2]
var arr2 = [3, 4]
var arr3 = arr1.concat(arr2)
console.log(arr1) //[1,2]
console.log(arr3) //[1,2,3,4]
```

### join()

join()方法用于把数组中的所有元素放入一个字符串。元素是通过指定的分隔符进行`分割`的。默认使用“,”分割。不改变原来数组。

```javascript
var arr = [1, 2]
console.log(arr.join()) //"1,2"
console.log(arr) //[1,2];
```

### push()

push()方法用于向数组`末尾添加`一个或多个元素。返回值为新数组的`length`。会改变原数组。

```javascript
var arr = [1, 2]
var newLength = arr.push(3, 4)
console.log(newLength) //4
console.log(arr) //[1,2,3,4];
```

### pop()

pop()方法用于`删除数组最后一个元素`。返回被删除的元素。会改变原数组

```javascript
var arr = [1, 2, 3, 4]
console.log(arr.pop()) //4
console.log(arr) //[1,2,3];
```

### shift()

shift()方法用于`删除数组第一个元素`。返回被删除的元素。会改变原数组

```javascript
var arr = [1, 2, 3, 4]
console.log(arr.shift()) //1
console.log(arr) //[2,3,4];
```

### unshift()

unshift()方法用于向`数组开头添加`一个或多个元素。返回新数组的 length。会改变原数组

```javascript
var arr = [3, 4]
console.log(arr.unshift(1, 2)) //4
console.log(arr) //[1,2,3,4];
```

### slice()

slice()方法可从已有的数组中返回`选定`的元素。返回一个新的数组，包含从 start 到 end （不包括该元素）的 arrayObject 中的元素。不会改变原数组

```javascript
var arr = [1, 2, 3, 4]
console.log(arr.slice(1, 3)) //[2,3]
console.log(arr) //[1,2,3,4];
```

### splice()

splice()方法可用于`删除`元素，并且可选择添加新元素来`替换`被删除的元素，第一个参数表示删除操作的开始下标。第二个参数表示删除的个数，如果没有第三个参数，splice 用于删除元素。如果有第三，第四个参数，表示采用第三第四。。。参数来替换被删除第元素。方法返回被删除元素的数组，会改变原数组。

```javascript
var arr = [1, 2, 3, 4]
console.log(arr.splice(1, 1)) //[2]
console.log(arr) //[1,3,4];
arr.splice(1, 0, 8, 9) //[]
console.log(arr) //[1,8,9,3,4]
```

### sort()

sort()方法按照 Unicode code 位置`排序`，默认升序.会改变原数组。

```javascript
var arr = [1, 10, 21, 2]
console.log(arr.sort()) //[1,10,2,21]
```

### reverse()

reverse()方法用于`颠倒元素顺序`。返回颠倒后的数组。会改变原数组。

```javascript
var arr = [1, 2, 3]
console.log(arr.reverse()) //[3,2,1]
console.log(arr) //[3,2,1]
```

### indexOf() lastIndexOf()

都接受两个参数，参数一为`查找`的值，参数二为查找的起始位置。查找结果不存在为-1，存在则返回下标。
indexOf()从前向后。
lastIndexOf()从后向前。

```javascript
var arr = [1, 2, 3, 2]
console.log(arr.indexOf(2)) //1
console.log(arr.indexOf(9)) //-1
console.log(arr.lastIndexOf(2)) //3
console.log(arr.lastIndexOf(2, 2)) //1
```

### every()

every()方法对数组对每一项元素都运行给定的函数，如果每一项都返回 true、则返回 true

```javascript
var ages = [32, 33, 16, 40]
function checkAdult(age) {
  return age >= 18
}
console.log(ages.every(checkAdult)) //false
```

### some()

some()方法对数组对每一项元素都运行给定的函数，如果任意一项返回 true、则返回 true

```javascript
var ages = [32, 33, 16, 40]
function checkAdult(age) {
  return age >= 18
}
console.log(ages.some(checkAdult)) //true
```

### filter()

filter()方法对数组对每一项元素都运行给定的函数，返回结果为 true 的项组成的数组。

```javascript
var ages = [32, 33, 16, 40]
function checkAdult(age) {
  return age >= 18
}
console.log(ages.filter(checkAdult)) //[32, 33, 40]
```

配和 includes、some 和 every 来使用

```js
const ages = [32, 33, 16, 40]
const value = [32, 33]
ages.filter(item => value.every(it => it !== item))
ages.filter(item => value.includes(item))
ages.filter(item => !value.some(it => it === item))
```

### map()

map()方法对数组对每一项元素都运行给定的函数，返回每次函数调用的结果组成一个新函数,不改变原数组

```javascript
var ages = [1, 2, 3, 4]
var mapFun = ages.map(function(x) {
  return x * 2
})
console.log(mapFun) //[2, 4, 6，8]
console.log(ages) //[1, 2, 3, 4];
```

### forEach() 数组遍历

```javascript
var ages = [1, 2, 3, 4]
var ages2 = []
var mapFun = ages.forEach(item => ages2.push(item))
console.log(ages2) //[1, 2, 3, 4];
console.log(ages) //[1, 2, 3, 4];
```

最后修改日期 2018 年 12 月 21 日
