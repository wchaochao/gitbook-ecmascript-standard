# Array构造器

标签（空格分隔）： ECMAScript规范

---

## 构造函数

### Array([item1 [ , item2 [ , … ] ] ] )

作为函数使用，等同于作为构造器使用

```javascript
/**
 * 调用new Array
 */
return new Array([item1 [ , item2 [ , … ] ] ] )
```

### new Array([item0 [ , item1 [ , … ] ] ] )

0个参数或至少2个参数，作为构造器使用，创建数组对象，指定元素

```javascript
/**
 * 创建数组对象
 * [[Class]]属性为'Array'
 * [[Prototype]]属性为Array.prototype
 * [[Extensible]]属性为true
 * length属性为参数个数
 * 元素为对应的参数
 */
let arr = new Object()
arr.[[Class]] = 'Array'
arr.[[Prototype]] = Array.prototype
arr.[[Extensible]] = true

arr.[[DefineOwnProperty]]('length', {
  [[Value]]: arguments.length,
  [[Writable]]: true,
  [[Enumerable]]: false,
  [[Configurable]]: false
})
for (let i = 0; i < arguments.length; i++) {
  arr.[[DefineOwnProperty]](ToString(i), {
    [[Value]]: arguments[i],
    [[Writable]]: true,
    [[Enumerable]]: true,
    [[Configurable]]: true
  })
}

return arr
```

示例

```javascript
new Array() // []
new Array(1, 2) // [1, 2]
new Array(1, undefined, 2) // [1, undefined, 2]
```

### new Array(len)

1个参数，作为构造器使用，创建数组对象，指定长度

```javascript
/**
 * 创建数组对象
 * [[Class]]属性为'Array'
 * [[Prototype]]属性为Array.prototype
 * [[Extensible]]属性为true
 * len为Number类型
 *    len为UInt32类型，length属性为len
 *    len不为UInt32类型，抛出RangeError
 * len不为Number类型
 *    length属性为1
 *    索引为0的元素值为len
 */
let arr = new Object()
arr.[[Class]] = 'Array'
arr.[[Prototype]] = Array.prototype
arr.[[Extensible]] = true

if (Type(len) === Number) {
  if (ToUint32(len) === len) {
    arr.[[DefineOwnProperty]]('length', {
      [[Value]]: len,
      [[Writable]]: true,
      [[Enumerable]]: false,
      [[Configurable]]: false
    })
  } else {
    throw RangeError
  }
} else {
  arr.[[DefineOwnProperty]]('length', {
    [[Value]]: 1,
    [[Writable]]: true,
    [[Enumerable]]: false,
    [[Configurable]]: false
  })
  arr.[[DefineOwnProperty]]('0', {
    [[Value]]: len,
    [[Writable]]: true,
    [[Enumerable]]: true,
    [[Configurable]]: true
  })
}

return arr
```

示例

```javascript
new Array(8) // [empty × 8]
new Array(Math.pow(2, 32)) // 抛出RangeError
new Array(3.2) // 抛出RangeError
new Array('a') // ['a']
```

### 静态属性

```javascript
Array.[[Class]] = 'Function'
Array.[[Prototype]] = Function.Prototype
Array.[[Extensible]] = true

Array.[[DefineOwnProperty]]('length', {
  [[Value]]: 1,
  [[Writable]]: false,
  [[Enumerable]]: false,
  [[Configurable]]: false
}, false)
Array.[[DefineOwnProperty]]('prototype', {
  [[Value]]: {constructor: Array,...},
  [[Writable]]: false,
  [[Enumerable]]: false,
  [[Configurable]]: false
}, false)
```

### 静态方法

#### Array.isArray(arg)

判断arg是否是数组

```javascript
/**
 * arg不为Object类型，返回false
 * arg为Object类型
 *    [[Class]]属性为'Array'，返回true
 *    [[Class]]属性不为'Array'，返回false
 */
if (Type(arg) !== Object) {
  return false
} else {
  return arg.[[Class]] === 'Array'
}
```

示例

```javascript
Array.isArray(1) // false
Array.isArray({}) // false
Array.isArray([]) // true
```

## 原型对象

Array对象的原型

```javascript
Array.prototype
```

### 原型属性

```javascript
Array.prototype.[[Class]] = 'Array'
Array.prototype.[[Prototype]] = Object.prototype
Array.prototype.[[Extensible]] = true

Array.prototype.constructor = Array
Array.prototype.length = 0
```

### 原型方法

**转换方法**

#### Array.prototype.toString()

获取数组的字符串表示

```javascript
/**
 * this转换为Object类型O
 * O有join方法，调用O.join的[[Call]]方法，thisArg为O, argList为空List
 * O没有join方法，调用Object.prototype.toString的[[Call]]方法，thisArg为O, argList为空List
 */
let O = ToObject(this)
let func = O.[[Get]]('join')
if (!IsCallable(func)) {
  func = Object.prototype.toString
}

return func.[[Call]](O, new List())
```

示例

```javascript
Array.prototype.toString.call(undefined) // 抛出TypeError

let arr = []
arr.toString() // ''

let arr = [1, 2, 3]
arr.toString() // '1,2,3'

let o = {a: 1}
Array.prototype.toString.call(o) // '[object Object]'

let o = {
  0: 0,
  1: 1,
  length: 2
}
Array.prototype.toString.call(o) // '[object Object]'
Array.prototype.toString.call('abc') // '[object String]'
```

#### Array.prototype.toLocaleString()

获取数组的本地字符串表示

```javascript
/**
 * this转换为Object类型O，length属性转换为UInt32类型，sep为listSeparatorString（由实现决定）
 * 遍历[0, length)，用分隔符将各索引值拼接起来
 *     索引值为undefined或null，转换为''
 *     索引值不为undefined或null，转换为Object类型并调用toLocaleString方法
 */
let O = ToObject(this)
let len = ToUint32(O.[[Get]]('length'))
let sep = listSeparatorString

let R = ''
for (let i = 0; i < len; i++) {
  if (i !== 0) {
    R += sep
  }
  let element = O.[[Get]](ToString(i))
  let next = element == null ? '' : ToObject(element).toLocaleString()
  R += next
}
return R
```

示例

```javascript
Array.prototype.toLocaleString.call(undefined) // 抛出TypeError

let arr = []
arr.toLocaleString() // ''

let arr = [1, undefined, 3]
arr.toLocaleString() // '1,,3'

let o = {a: 1}
Array.prototype.toLocaleString.call(o) // ''

let o = {
  0: 0,
  1: 1,
  length: 2
}
Array.prototype.toLocaleString.call(o) // '0,1'
Array.prototype.toLocaleString.call('abc') // 'a,b,c'
```

#### Array.prototype.join(separator)

以指定分隔符连接数组元素

```javascript
/**
 * this转换为Object类型，length属性转换为UInt32类型
 * separator转换为String类型，默认为','
 * 遍历[0, length)，用分隔符将各索引值拼接起来
 *     索引值为undefined或null，转换为''
 *     索引值不为undefined或null，转换为String类型
 */
let O = ToObject(this)
let len = ToUint32(O.[[Get]]('length'))
let sep = separator == null ? ',' : ToString(separator)

let R = ''
for (let i = 0; i < len; i++) {
  if (i !== 0) {
    R += sep
  }
  let element = O.[[Get]](ToString(i))
  let next = element == null ? '' : ToString(element)
  R += next
}
return R
```

示例

```javascript
Array.prototype.join.length // 1

Array.prototype.join.call(undefined) // 抛出TypeError

let arr = []
arr.join() // ''

let arr = [1, undefined, 3]
arr.join() // '1,,3'

let o = {a: 1}
Array.prototype.join.call(o) // ''

let o = {
  0: 0,
  1: 1,
  length: 2
}
Array.prototype.join.call(o) // '0,1'
Array.prototype.join.call('abc', '-') // 'a-b-c'
```

**栈方法**

#### Array.prototype.push([item1 [ , item2 [ , … ] ] ] )

将元素追加到数组末尾并返回数组长度

```javascript
/**
 * this转换为Object类型，length属性转换为UInt32类型len
 * 遍历传入的参数，追加到len末尾，设置length为新length并返回
 */
let O = ToObject(this)
let len = ToUint32(O.[[Get]]('length'))

for (let i = 0; i < arguments.length; i++) {
  O.[[Put]](ToString(len), arguments[i], true)
  len++
}
O.[[Put]]('length', len, true)
return len
```

示例

```javascript
Array.prototype.push.length // 1

Array.prototype.push.call(undefined) // 抛出TypeError

let arr = []
arr.push(1, 2) // 2, arr为[1, 2]

let arr = [1, 2]
arr.push(3) // 3, arr为[1, 2, 3]

let o = {a: 1}
Array.prototype.push.call(o) // 0, o为{a: 1, length: 0}

let o = {
  0: 0,
  1: 1,
  length: 2
}
Array.prototype.push.call(o, 2) // 3, o为{0: 0, 1: 1, 2: 2, length: 3}
Array.prototype.push.call('abc', 1, 2) // Uncaught TypeError: Cannot assign to read only property 'length' of object '[object String]'
```

#### Array.prototype.pop()

移除最后一个元素并返回该元素

```javascript
/**
 * this转换为Object类型，length属性转换为UInt32类型len
 * len为0，设置length属性为0，返回undefined
 * len不为0，删除最后一个索引，设置length属性为len - 1，返回最后一个索引对应的元素
 */
let O = ToObject(this)
let len = ToUint32(O.[[Get]]('length'))

if (len === 0) {
  O.[[Put]]('length', O, true)
  return undefined
} else {
  let lastIndex = ToString(len - 1)
  let element = O.[[Get]](lastIndex)
  O.[[Delete]](lastIndex, true)
  O.[[Put]]('length', len - 1, true)
  return element
}
```

示例

```javascript
Array.prototype.pop.call(undefined) // 抛出TypeError

let arr = []
arr.pop() // undefined, arr为[]

let arr = [1, 2, 3]
arr.pop() // 3, arr为[1, 2]

let o = {a: 1}
Array.prototype.pop.call(o) // undefined, o为{a: 1, length: 0}

let o = {
  0: 0,
  1: 1,
  length: 2
}
Array.prototype.pop.call(o) // 1, o为{0: 0, length: 1}
Array.prototype.pop.call('abc') // Uncaught TypeError: Cannot delete property '2' of [object String]
```

**队列方法**

#### Array.prototype.unshift([item1 [ , item2 [ , … ] ] ] )

将元素追加到数组开头并返回数组长度

```javascript
/**
 * this转换为Object类型，length属性转换为UInt32类型len
 * 反向遍历[0, len), 所有索引后移argCount位
 * 遍历[0, argCount), 设置索引值为对应参数
 * 设置length属性为len + argCount并返回
 */
let O = ToObject(this)
let len = ToUint32(O.[[Get]]('length'))
let argCount = arguments.length

for (let i = len - 1; i >= 0; i++) {
  let from = ToString(i)
  let to = ToString(i + argCount)
  if (O.[[HasProperty]](from)) {
    let fromValue = O.[[Get]](from)
    O.[[Put]](to, fromValue, true)
  } else {
    O.[[Delete]](to, true)
  }
}

for (let i = 0; i < argCount; i++) {
  O.[[Put]](ToString(i), arguements[i], true)
}

O.[[Put]]('length', len + argCount, true)
return len + argCount
```

示例

```javascript
Array.prototype.unshift.length // 1

Array.prototype.unshift.call(undefined) // 抛出TypeError

let arr = []
arr.unshift(1, 2) // 2, arr为[1, 2]

let arr = [1, 2]
arr.unshift(3) // 3, arr为[3, 1, 2]

let o = {a: 1}
Array.prototype.unshift.call(o) // 0, o为{a: 1, length: 0}

let o = {
  0: 0,
  1: 1,
  length: 2
}
Array.prototype.unshift.call(o, 2) // 3, o为{0: 2, 1: 0, 2: 1, length: 3}
Array.prototype.unshift.call('abc', 1, 2) // Uncaught TypeError: Cannot assign to read only property '2' of object '[object String]'
```

#### Array.prototype.shift()

移除第一个元素并返回该元素

```javascript
/**
 * this转换为Object类型，length属性转换为UInt32类型len
 * len为0，设置length属性为0，返回undefined
 * len不为0，第一个索引后的所有索引往前进一，删除最后一个索引，设置length属性为len - 1，返回第一个索引对应的元素
 */
let O = ToObject(this)
let len = ToUint32(O.[[Get]]('length'))

if (len === 0) {
  O.[[Put]]('length', O, true)
  return undefined
} else {
  let first = O.[[Get]]('0')
  for (let i = 1; i < len; i++) {
    let from = ToString(i)
    let to = ToString(i - 1)
    if (O.[[HasProperty]](from)) {
      let fromValue = O.[[Get]](from)
      O.[[Put]](to, fromValue, true)
    } else {
      O.[[Delete]](to, true)
    }
    O.[[Delete]](ToString(len - 1), true)
  }
  return first
}
```

示例

```javascript
Array.prototype.shift.call(undefined) // 抛出TypeError

let arr = []
arr.shift() // undefined, arr为[]

let arr = [1, 2, 3]
arr.shift() // 1, arr为[2, 3]

let o = {a: 1}
Array.prototype.shift.call(o) // undefined, o为{a: 1, length: 0}

let o = {
  0: 0,
  1: 1,
  length: 2
}
Array.prototype.shift.call(o) // 1, o为{0: 1, length: 1}
Array.prototype.shift.call('abc') // Uncaught TypeError: Cannot assign to read only property '0' of object '[object String]'
```

**排序方法**

#### Array.prototype.reverse()

数组倒序排列

```javascript
/**
 * this转换为Object类型，length属性转换为UInt32类型len
 * 以floor(len/2)为中点，交互两边对应索引的值
 */
let O = ToObject(this)
let len = ToUint32(O.[[Get]]('length'))

len middle = floor(len/2)
let lower = 0
while (lower !== middle) {
  let upper = len - 1 - lower
  let lowerP = ToString(lower)
  let upperP = ToString(upper)
  let lowerValue = O.[[Get]](lowerP)
  let upperValue = O.[[Get]](upperP)
  let lowerExists = O.[[HasProperty]](lowerP)
  let upperExists = O.[[HasProperty]](upperP)
  
  if (lowerExists && upperExists) {
    O.[[Put]](lowerP, upperValue, true)
    O.[[Put]](upperP, lowerValue, true)
  } else if (lowerExists && !upperExists) {
    O.[[Delete]](lowerP, true)
    O.[[Put]](upperP, lowerValue, true)
  } else if (!lowerExists && upperExists) {
    O.[[Put]](lowerP, upperValue, true)
    O.[[Delete]](upperP, true)
  }
  lower++
}

return O
```

示例

```javascript
Array.prototype.reverse.call(undefined) // 抛出TypeError

let arr = []
arr.reverse() // []

let arr = [1, 2, 3]
arr.reverse() // [3, 2, 1]

let o = {a: 1}
Array.prototype.reverse.call(o) // {a: 1}

let o = {
  0: 0,
  1: 1,
  length: 2
}
Array.prototype.reverse.call(o) // {0: 1, 1: 0, length: 2}
Array.prototype.reverse.call('abc') // Uncaught TypeError: Cannot assign to read only property '0' of object '[object String]'
```

#### Array.prototype.sort(comparefn)

数组按comparefn排序

```javascript
/**
 * this转换为Object类型，length属性转换为UInt32类型len
 * 对任意两个索引j、k进行排序，按排序完的顺序排列
 *     空位 > undefined > 其他值
 *     comparefn为undefined，索引值转换为String类型比较
 *     comparefn不为undefined
 *         不为函数，抛出TypeError
 *         为函数，调用[[Call]]方法，thisArg为undefined，参数为两索引值
 *             comparefn(a, b) < 0, 则a < b
 *             comparefn(a, b) = 0, 则a = b
 *             comparefn(a, b) > 0, 则a = b
 */
let O = ToObject(this)
let len = ToUint32(O.[[Get]]('length'))

let jString = ToString(j)
let kString = ToString(k)
let hasj = O.[[hasProperty]](jString)
let hask = O.[[hasProperty]](kString)
if (!hasj && !hask) {
  return +0
}
if (!hasj) {
  return 1
}
if (!hask) {
  return -1
}

let x = O.[[Get]](jString)
let y = O.[[Get]](kString)
if (x === undefined && y === undefined) {
  return +0
}
if (x === undefined) {
  return 1
}
if (y === undefined) {
  return 1
}
if (comparefn === undefined) {
  let xString = ToString(x)
  let yString = ToString(y)
  if (xString > yString) {
    return 1
  }
  if (xString < yString) {
    return -1
  }
  return +0
} else {
  if (!IsCallable(comparefn)) {
    throw TypeError
  }
  return comparefn.[[Call]](undefined, x, y)
}
```

示例

```javascript
Array.prototype.reverse.sort(undefined) // 抛出TypeError

let arr = []
arr.sort() // []

let arr = [1, , 3, undefined, null]
arr.sort() // [1, 3, null, undefined, empty]

let arr = [1, , 3, undefined, null]
arr.sort(2) // TypeError

let arr = [1, , 3, undefined, null]
arr.sort((a, b) => a - b) // [null, 1, 3, undefined, empty]

let o = {a: 1}
Array.prototype.sort.call(o) // {a: 1}

let o = {
  0: 1,
  2: 3,
  3: undefined,
  4: null,
  length: 5
}
Array.prototype.sort.call(o) // {0: 1, 1: 3, 2: null, 3: undefined, length: 5}
Array.prototype.sort('abc') // Uncaught TypeError: Cannot assign to read only property '0' of object '[object String]'
```

**操作方法**

#### Array.prototype.concat([item1 [ , item2 [ , … ] ] ] )

数组拼接

```javascript
/**
 * this转换为Object类型，与参数组成列表
 * 创建新数组，遍历列表
 *     列表值不为数组，设置新数组元素为列表值
 *     列表值为数组，继续遍历数组，设置新数组元素为数组元素
 */
let O = ToObject(this)
let items = new List(O).concat(arguments)

let A = new Array()
let n = 0
for (let i = 0; i < items.length; i++) {
  let E = items[i]
  if (E.[[Class]] === 'Array') {
    for (let j = 0; j < E.length; i++) {
      let P = ToString(j)
      if (E.[[HasProperty]](P)) {
        let subElement = E.[[Get]](P)
        A.[[DefineOwnProperty]](ToString(n), {
          [[Value]]: subElement,
          [[Writable]]: true,
          [[Enumerable]]: true,
          [[Configurable]]: true
        })
      }
      n++
    }
  } else {
    A.[[DefineOwnProperty]](ToString(n), {
      [[Value]]: E,
      [[Writable]]: true,
      [[Enumerable]]: true,
      [[Configurable]]: true
    })
    n++
  }
}
```

示例

```javascript
Array.prototype.concat.length // 1

Array.prototype.concat.call(undefined) // 抛出TypeError

let arr = [1,,2]
arr.concat() // [1, empty, 2]

let arr = [1,,2]
arr.concat(3, undefined, [4,,5]) // [1, empty, 2, 3, undefined, 4, empty, 5]

let o = {a: 1}
Array.prototype.concat.call(o) // [{a: 1}]

let o = {
  0: 0,
  1: 1,
  length: 2
}
Array.prototype.concat.call(o) // [{0: 0, 1: 1, length: 2}]
Array.prototype.concat.call('abc', [1, 2]) // [String{'abc'}, 1, 2]
```

#### Array.prototype.slice(start, end)

数组截取

```javascript
/**
 * this转换为Object类型，length属性转换为UInt32类型len
 * start转换为整数，为负数时加len
 *      小于0时置为0，大于len时置为len，落在[0, len]之间
 * end为undefined时，置为len
 * end不为undefined时，与start同样处理
 * 遍历[start, end)，将索引值添加到新数组中，返回新数组
 */
let O = ToObject(this)
let len = ToUint32(O.[[Get]]('length'))

function format (num) {
  num = ToInterger(num)
  if (num < 0) {
    num = max(num + len, 0)
  } else {
    num = min(num, len)
  }
}

start = format(start)
end = end === undefined ? len : format(end)

let A = new Array()
let n = 0
for (let i = start; i < end; i++) {
  let P = ToString(i)
  if (O.[[HasProperty]](P)) {
    let value = O.[[Get]](P)
    A.[[DefineOwnProperty]](ToString(n), {
      [[Value]]: value,
      [[Writable]]: true,
      [[Enumerable]]: true,
      [[Configurable]]: true
    }, false)
  }
  n++
}
return A
```

示例

```javascript
Array.prototype.slice.length // 2

Array.prototype.slice.call(undefined) // 抛出TypeError

let arr = [1,,2]
arr.slice() // [1, empty, 2]

let arr = [1,,2]
arr.slice(1, -1) // [empty]
arr.slice(2, 1) // []

let o = {a: 1}
Array.prototype.slice.call(o, 1, -1) // []

let o = {
  0: 0,
  1: 1,
  length: 2
}
Array.prototype.slice.call(o, 0, -1) // [0]
Array.prototype.slice.call('abc', 1, -1) // ['b']
```

#### Array.prototype.splice(start, deleteCount [ , item1 [ , item2 [ , … ] ] ] )

数组替换

```javascript
/**
 * this转换为Object类型，length属性转换为UInt32类型len
 * start转换为整数，为负数时加len
 *      小于0时置为0，大于len时置为len，落在[0, len]之间
 * deleteCount转换为整数
 *      小于0时置为0，大于len - start时置为len - start，落在[0, len - start]之间
 * 遍历[start, start + deleteCount)，设置新数组元素为对应的索引值
 * deleteCount > argCount, 遍历[start + deleteCount, len)，将索引往前移动 deleteCount - argCount位
 * deleteCount < argCount, 反向遍历[start + deleteCount, len)，将索引往后移动argCount - deleteCount位
 * 遍历[start, start + itemCount)，将索引值设为对应的参数
 * 设置length属性为len + itemCount - deleteCount
 * 返回新数组
 */
let O = ToObject(this)
let len = ToUint32(O.[[Get]]('length'))

function format (num) {
  num = ToInterger(num)
  if (num < 0) {
    num = max(num + len, 0)
  } else {
    num = min(num, len)
  }
}

start = format(start)
deleteCount = min(max(ToInterger(deleteCount), 0), len - start)
let argCount = arguments.length

let A = new Array()
for (let i = start; i < start + deleteCount; i ++) {
  let from = ToString(i)
  let fromPresent = O.[[HasProperty]](from)
  if (fromPresent) {
    let fromValue = O.[[Get]](from)
    A.[[DefineOwnProperty]](ToString(i - start), {
      [[Value]]: fromValue,
      [[Writable]]: true,
      [[Enumerable]]: true,
      [[Configurable]]: true
    }, false)
  }
}

if (deleteCount > argCount) {
  for (let i = start + deleteCount; i < len; i++) {
    let from = ToString(i)
    let to = ToString(i + argCount - deleteCount)
    let fromPresent = O.[[HasProperty]](from)
    if (fromPresent) {
      let fromValue = O.[[Get]](from)
      O.[[Put]](to, fromValue, true)
    } else {
      O.[[Delete]](to, true)
    }
  }
  for (let i = len - deleteCount + startCount; i < len; i++) {
    O.[[Delete]](to, true)
  }
} else if (deleteCount < argCount) {
  for (let i = len - 1; i >= start + deleteCount; i--) {
    let from = ToString(i)
    let to = ToString(i + argCount - deleteCount)
    let fromPresent = O.[[HasProperty]](from)
    if (fromPresent) {
      let fromValue = O.[[Get]](from)
      O.[[Put]](to, fromValue, true)
    } else {
      O.[[Delete]](to, true)
    }
  }
}

for (let i = start; i < start + argCount; i++) {
  O.[[Put]](ToString(i), arguments[i - start], true)
}

O.[[Put]]('length', len - deleteCount + startCount, true)
return A
```

示例

```javascript
Array.prototype.splice.length // 2

Array.prototype.splice.call(undefined) // 抛出TypeError

let arr = [1,,2]
arr.splice() // [], arr为[1, empty, 2]

let arr = [1,,2]
arr.splice(0, 0, 1) // [], arr为[1, 1, empty, 2]
arr.splice(-2, 2, 2) // [empty, 2], arr为[1, 2]
arr.splice(4, 5, 6, 7)  // [], arr为[1, empty, 2, 6, 7]

let o = {a: 1}
Array.prototype.splice.call(o, 1, -1) // [], o为{a: 1, length: 0}

let o = {
  0: 0,
  1: 1,
  length: 2
}
Array.prototype.splice.call(o, 0, 1, 2) // [0], o为{0: 2, 1: 1, length: 2}
Array.prototype.splice.call('abc', 1, 1) // Uncaught TypeError: Cannot assign to read only property '1' of object '[object String]'
```

**位置方法**

#### Array.prototype.indexOf(searchElement [ , fromIndex ] )

按索引升序从指定索引开始查找元素

```javascript
/**
 * this转换为Object类型，length属性转换为UInt32类型len
 * fromIndex未提供，置为0
 * fromIndex已提供
 *      转换为整数，为负数时加len，小于0时置为0
 * 遍历[fromIndex, len)
 *     若有索引值与searchElement严格相等，则返回对应的索引
 *     若所有索引值都不与searchElement严格相等，则返回-1
 */
let O = ToObject(this)
let len = ToUint32(O.[[Get]]('length'))

function format (num) {
  num = ToInterger(num)
  if (num < 0) {
    num = max(num + len, 0)
  }
}

fromIndex = arguments.length < 2 ? 0 : format(fromIndex)

for (let i = fromIndex; i < len; i++) {
  let from = ToString(i)
  if (O.[[HasProperty]](from)) {
    let fromValue = O.[[Get]](from)
    if (fromValue === searchElement) {
      return i
    }
  }
}

return -1
```

示例

```javascript
Array.prototype.indexOf.length // 1

Array.prototype.indexOf.call(undefined) // 抛出TypeError

let arr = [1,,2,undefined,1,NaN]
arr.indexOf() // 3
arr.indexOf(1) // 0
arr.indexOf(1, 2) // 4
arr.indexOf('1') // -1
arr.indexOf(NaN) // -1

let o = {a: 1}
Array.prototype.indexOf.call(o, 1) // -1

let o = {
  0: 0,
  1: 1,
  length: 2
}
Array.prototype.indexOf.call(o, 0) // 0
Array.prototype.indexOf.call('abcba', 'b', 2) // 3
```

#### Array.prototype.lastIndexOf(searchElement [ , fromIndex ] )

按索引降序从指定索引开始查找元素

```javascript
/**
 * this转换为Object类型，length属性转换为UInt32类型len
 * fromIndex未提供，置为len - 1
 * fromIndex已提供
 *      转换为整数，为负数时加len，大于len - 1时置为len - 1
 * 反向遍历[0, fromIndex]
 *     若有索引值与searchElement严格相等，则返回对应的索引
 *     若所有索引值都不与searchElement严格相等，则返回-1
 */
let O = ToObject(this)
let len = ToUint32(O.[[Get]]('length'))

function format (num) {
  num = ToInterger(num)
  if (num < 0) {
    num += len
  } else {
    num = min(num, len - 1)
  }
}

fromIndex = arguments.length < 2 ? len - 1 : format(fromIndex)

for (let i = fromIndex; i >= 0; i--) {
  let from = ToString(i)
  if (O.[[HasProperty]](from)) {
    let fromValue = O.[[Get]](from)
    if (fromValue === searchElement) {
      return i
    }
  }
}

return -1
```

示例

```javascript
Array.prototype.lastIndexOf.length // 1

Array.prototype.lastIndexOf.call(undefined) // 抛出TypeError

let arr = [1,,2]
arr.lastIndexOf() // -1

let arr = [1,,2,undefined,1,NaN]
arr.lastIndexOf() // 3
arr.lastIndexOf(1) // 4
arr.lastIndexOf(1, 2) // 0
arr.lastIndexOf('1') // undefined
arr.lastIndexOf(NaN) // undefined

let o = {a: 1}
Array.prototype.lastIndexOf.call(o, 1) // -1

let o = {
  0: 0,
  1: 1,
  length: 2
}
Array.prototype.lastIndexOf.call(o, 0) // 0
Array.prototype.lastIndexOf.call('abcba', 'b', 2) // 1
```

**遍历方法**

#### Array.prototype.forEach(callbackfn [ , thisArg ] )

数组遍历

```javascript
/**
 * this转换为Object类型O，length属性转换为UInt32类型len
 * callbackfn不为函数抛出TypeError, thisArg未提供为undefined
 * 遍历[0, len)，对非空位的索引值调用callbackfn，this为thisArg，参数为List{索引值，索引，O}
 * 返回undefined
 */
let O = ToObject(this)
let len = ToUint32(O.[[Get]]('length'))

if (!IsCallable(callbackfn)) {
  throw TypeError
}
let T = arguments.length < 2 ? undefined : thisArg

for (let i = 0; i < len; i++) {
  let Pk = ToString(i)
  if (O.[[HasProperty]](Pk)) {
    let kValue = O.[[Get]](Pk)
    callbackfn.[[Call]](T, kValue, i, O)
  }
}

return undefined
```

示例

```javascript
Array.prototype.forEach.length // 1

Array.prototype.forEach.call(undefined) // 抛出TypeError

let arr = []
arr.forEach(1) // 抛出TypeError

let arr = []
arr.forEach(x => console.log(x))

let arr = [1,,2]
arr.forEach(x => console.log(x))
// 1
// 2

let arr = [1,,2]
arr.forEach(function (ele, index, arr) {
  console.log(ele + this.x)
}, {x: 1})
// 2
// 3

let o = {a: 1}
Array.prototype.forEach.call(o, x => console.log(x))

let o = {
  0: 0,
  2: 2,
  length: 3
}
Array.prototype.forEach.call(o, x => console.log(x))
// 0
// 2

Array.prototype.forEach.call('abc', x => console.log(x))
// 'a'
// 'b'
// 'c'
```

#### Array.prototype.map(callbackfn [ , thisArg ] )

数组映射

```javascript
/**
 * this转换为Object类型O，length属性转换为UInt32类型len
 * callbackfn不为函数抛出TypeError, thisArg未提供为undefined
 * 遍历[0, len)，对非空位的索引值调用callbackfn，this为thisArg，参数为List{索引值，索引，O}
 *      callbackfn返回值映射到新数组中
 * 返回新数组
 */
let O = ToObject(this)
let len = ToUint32(O.[[Get]]('length'))

if (!IsCallable(callbackfn)) {
  throw TypeError
}
let T = arguments.length < 2 ? undefined : thisArg

let A = new Array()
for (let i = 0; i < len; i++) {
  let Pk = ToString(i)
  if (O.[[HasProperty]](Pk)) {
    let kValue = O.[[Get]](Pk)
    let mappedValue = callbackfn.[[Call]](T, kValue, i, O)
    A.[[DefineOwnProperty]](Pk, {
      [[Value]]: mappedValue,
      [[Writable]]: true,
      [[Enumberable]]: true,
      [[Configurable]]: true
    })
  }
}

return A
```

示例

```javascript
Array.prototype.map.length // 1

Array.prototype.map.call(undefined) // 抛出TypeError

let arr = []
arr.map(1) // 抛出TypeError

let arr = []
arr.map(x => x + 1) // []

let arr = [1,,2]
arr.map(x => x + 1) // [2, empty, 3]

let arr = [1,,2]
arr.map(function (ele, index, arr) {
  return ele + this.x
}, {x: 1}) // [2, empty, 3]

let o = {a: 1}
Array.prototype.map.call(o, x => x + 1) // []

let o = {
  0: 0,
  2: 2,
  length: 3
}
Array.prototype.map.call(o, x => x + 1) // [1, empty, 3]
Array.prototype.map.call('abc', x => x + 1) // ['a1', 'b1', 'c1']
```

#### Array.prototype.filter(callbackfn [ , thisArg ] )

数组过滤

```javascript
/**
 * this转换为Object类型O，length属性转换为UInt32类型len
 * callbackfn不为函数抛出TypeError, thisArg未提供为undefined
 * 遍历[0, len)，对非空位的索引值调用callbackfn，this为thisArg，参数为List{索引值，索引，O}
 *     若callbackfn返回值转换为true，则将索引值添加到新数组中
 * 返回新数组
 */
let O = ToObject(this)
let len = ToUint32(O.[[Get]]('length'))

if (!IsCallable(callbackfn)) {
  throw TypeError
}
let T = arguments.length < 2 ? undefined : thisArg

let A = new Array()
let to = 0
for (let i = 0; i < len; i++) {
  let Pk = ToString(i)
  if (O.[[HasProperty]](Pk)) {
    let kValue = O.[[Get]](Pk)
    let selected = callbackfn.[[Call]](T, kValue, i, O)
    if (ToBoolean(selected)) {
      A.[[DefineOwnProperty]](ToString(to), {
        [[Value]]: kValue,
        [[Writable]]: true,
        [[Enumerable]]: true,
        [[Configurable]]: true
      })
    }
  }
}

return A
```

示例

```javascript
Array.prototype.filter.length // 1

Array.prototype.filter.call(undefined) // 抛出TypeError

let arr = []
arr.filter(1) // 抛出TypeError

let arr = []
arr.filter(x => x > 0) // []

let arr = [1,,2]
arr.filter(x => x > 0) // [1, 2]

let arr = [1,,2]
arr.filter(function (ele, index, arr) {
  return ele > this.x
}, {x: 1}) // [1, 2]

let o = {a: 1}
Array.prototype.filter.call(o, x => x > 0) // []

let o = {
  0: 0,
  2: 2,
  length: 3
}
Array.prototype.filter.call(o, x => x > 0) // [2]
Array.prototype.filter.call('abc', x => x < 'd') // ['a', 'b', 'c']
```

#### Array.prototype.every(callbackfn [ , thisArg ] )

判断数组是否所有元素符合条件

```javascript
/**
 * this转换为Object类型O，length属性转换为UInt32类型len
 * callbackfn不为函数抛出TypeError, thisArg未提供为undefined
 * 遍历[0, len)，对非空位的索引值调用callbackfn，this为thisArg，参数为List{索引值，索引，O}
 *     若有callbackfn返回值转换为false，则返回false
 *     若所有callbackfn的返回值都转换为true，则返回true
 */
let O = ToObject(this)
let len = ToUint32(O.[[Get]]('length'))

if (!IsCallable(callbackfn)) {
  throw TypeError
}
let T = arguments.length < 2 ? undefined : thisArg

for (let i = 0; i < len; i++) {
  let Pk = ToString(i)
  if (O.[[HasProperty]](Pk)) {
    let kValue = O.[[Get]](Pk)
    let testResult = callbackfn.[[Call]](T, kValue, i, O)
    if (!ToBoolean(testResult)) {
      return false
    }
  }
}

return true
```

示例

```javascript
Array.prototype.every.length // 1

Array.prototype.every.call(undefined) // 抛出TypeError

let arr = []
arr.every(1) // 抛出TypeError

let arr = []
arr.every(x => x > 0) // true

let arr = [1,,2]
arr.every(x => x > 0) // true

let arr = [1,,2]
arr.every(function (ele, index, arr) {
  return ele > this.x
}, {x: 1}) // false

let o = {a: 1}
Array.prototype.every.call(o, x => x > 0) // true

let o = {
  0: 0,
  2: 2,
  length: 3
}
Array.prototype.every.call(o, x => x > 0) // false
Array.prototype.every.call('abc', x => x < 'd') // true
```

#### Array.prototype.some(callbackfn [ , thisArg ] )

判断数组是否有元素符合条件

```javascript
/**
 * this转换为Object类型O，length属性转换为UInt32类型len
 * callbackfn不为函数抛出TypeError, thisArg未提供为undefined
 * 遍历[0, len)，对非空位的索引值调用callbackfn，this为thisArg，参数为List{索引值，索引，O}
 *     若有callbackfn返回值转换为true，则返回true
 *     若所有callbackfn的返回值都转换为false，则返回false
 */
let O = ToObject(this)
let len = ToUint32(O.[[Get]]('length'))

if (!IsCallable(callbackfn)) {
  throw TypeError
}
let T = arguments.length < 2 ? undefined : thisArg

for (let i = 0; i < len; i++) {
  let Pk = ToString(i)
  if (O.[[HasProperty]](Pk)) {
    let kValue = O.[[Get]](Pk)
    let testResult = callbackfn.[[Call]](T, kValue, i, O)
    if (ToBoolean(testResult)) {
      return true
    }
  }
}

return false
```

示例

```javascript
Array.prototype.some.length // 1

Array.prototype.some.call(undefined) // 抛出TypeError

let arr = []
arr.some(1) // 抛出TypeError

let arr = []
arr.some(x => x > 0) // false

let arr = [1,,2]
arr.some(x => x > 0) // true

let arr = [1,,2]
arr.some(function (ele, index, arr) {
  return ele > this.x
}, {x: 1}) // true

let o = {a: 1}
Array.prototype.some.call(o, x => x > 0) // false

let o = {
  0: 0,
  2: 2,
  length: 3
}
Array.prototype.some.call(o, x => x > 0) // true
Array.prototype.some.call('abc', x => x < 'd') // true
```

**累计方法**

#### Array.prototype.reduce(callbackfn [ , initialValue ] )

按索引升序对数组进行累计

```javascript
/**
 * this转换为Object类型O，length属性转换为UInt32类型len
 * callbackfn不为函数抛出TypeError
 * initialValue提供时，accumulator为initialValue，start为0
 * initialValue未提供时，accumulator为第一个非空位索引值，start为索引 + 1
 *      无非空位索引值时，抛出TypeError
 * 遍历[start, len)，对非空位的索引值调用callbackfn，this为undefined，参数为List{accumulator, 索引值，索引，O}，返回值赋给accumulator
 * 返回accumulator
 */
let O = ToObject(this)
let len = ToUint32(O.[[Get]]('length'))

if (!IsCallable(callbackfn)) {
  throw TypeError
}

let accumulator = initialValue
let start = 0
if (arguments.length < 2) {
  let kPresent = false
  while (start < len) {
    let from = ToString(start)
    if (O.[[HasProperty]](from)) {
      kPresent = true
      accumulator = O.[[Get]](from)
      start++
      break
    }
  }
  if (!kPresent) {
    throw TypeError
  }
}

for (let i = start; i < len; i++) {
  let Pk = ToString(i)
  if (O.[[HasProperty]](Pk)) {
    let kValue = O.[[Get]](Pk)
    accumulator = callbackfn.[[Call]](undefined, accumulator, kValue, i, O)
  }
}

return accumulator
```

示例

```javascript
Array.prototype.reduce.length // 1

Array.prototype.reduce.call(undefined) // 抛出TypeError

let arr = []
arr.reduce(1) // 抛出TypeError

let arr = []
arr.reduce((pre, next) => pre + next) // 抛出TypeError

let arr = [,,,]
arr.reduce((pre, next) => pre + next) // 抛出TypeError

let arr = [,1,,2]
arr.reduce((pre, next) => pre + next) // 3

let arr = [,1,,2]
arr.reduce((pre, next) => pre + next, 3) // 6

let o = {a: 1}
Array.prototype.reduce.call(o, (pre, next) => pre + next) // 抛出TypeError

let o = {
  0: 0,
  2: 2,
  length: 3
}
Array.prototype.reduce.call(o, (pre, next) => pre + next) // 2
Array.prototype.reduce.call('abc', (pre, next) => pre + next) // 'abc'
```

#### Array.prototype.reduceRight(callbackfn [ , initialValue ] )

按索引降序对数组进行累计

```javascript
/**
 * this转换为Object类型O，length属性转换为UInt32类型len
 * callbackfn不为函数抛出TypeError
 * initialValue提供时，accumulator为initialValue，start为len - 1
 * initialValue未提供时，accumulator为倒数第一个非空位索引值，start为索引 - 1
 *      无非空位索引值时，抛出TypeError
 * 反向遍历[0, start]，对非空位的索引值调用callbackfn，this为undefined，参数为List{accumulator, 索引值，索引，O}，返回值赋给accumulator
 * 返回accumulator
 */
let O = ToObject(this)
let len = ToUint32(O.[[Get]]('length'))

if (!IsCallable(callbackfn)) {
  throw TypeError
}

let accumulator = initialValue
let start = len - 1
if (arguments.length < 2) {
  let kPresent = false
  while (start >= 0) {
    let from = ToString(start)
    if (O.[[HasProperty]](from)) {
      kPresent = true
      accumulator = O.[[Get]](from)
      start--
      break
    }
  }
  if (!kPresent) {
    throw TypeError
  }
}

for (let i = start; i > 0; i--) {
  let Pk = ToString(i)
  if (O.[[HasProperty]](Pk)) {
    let kValue = O.[[Get]](Pk)
    accumulator = callbackfn.[[Call]](undefined, accumulator, kValue, i, O)
  }
}

return accumulator
```

示例

```javascript
Array.prototype.reduceRight.length // 1

Array.prototype.reduceRight.call(undefined) // 抛出TypeError

let arr = []
arr.reduceRight(1) // 抛出TypeError

let arr = []
arr.reduceRight((pre, next) => pre + next) // 抛出TypeError

let arr = [,,,]
arr.reduceRight((pre, next) => pre + next) // 抛出TypeError

let arr = [,1,,2]
arr.reduceRight((pre, next) => pre + next) // 3

let arr = [,1,,2]
arr.reduceRight((pre, next) => pre + next, 3) // 6

let o = {a: 1}
Array.prototype.reduceRight.call(o, (pre, next) => pre + next) // 抛出TypeError

let o = {
  0: 0,
  2: 2,
  length: 3
}
Array.prototype.reduceRight.call(o, (pre, next) => pre + next) // 2
Array.prototype.reduceRight.call('abc', (pre, next) => pre + next) // 'cba'
```

## 实例对象

[[Class]]为Array的对象

```javascript
// 通过数组直接量创建
[1]

// 通过构造器Array创建
new Array('a', 'b')
```

### 实例属性

```javascript
arr.[[Class]] = 'Array'
arr.[[Prototype]] = Array.prototype
arr.[[Extensible]] = true

arr.[[DefineOwnProperty]]('length', {
  [[Value]]: length,
  [[Writable]]: true,
  [[Enumerable]]: false,
  [[Configurable]]: false
})
```

### 实例方法

#### A.[[DefineOwnProperty]] (P, Desc, Throw )

```javascript
/**
 * P为length属性，设置length值
 *    newLen不为UInt32类型，抛出RangeError
 *    newLen >= oldLen, 设置length属性为newLen
 *    newLen < oldLen, 设置length属性为新值并删除所有索引大于newLen - 1的属性
 *        删除失败时，设置length属性为失败时的索引 + 1
 * P为索引属性，设置元素值
 *    P <= len - 1，设置P对应的元素
 *    P > len - 1，设置P对象的元素值，并设置length为索引 + 1
 * P为普通属性，设置属性值
 */
Reject:
  if (Throw) {
    throw TypeError
  } else {
    return false
  }
  
A.defaultDefineOwnProperty = A.[[DefineOwnProperty]]
A.[[DefineOwnProperty]] = function (P, Desc, Throw) {
  let oldLenDesc = A.[[GetOwnProperty]]('length')
  let oldLen = oldLenDesc.[[Value]]
  
  if (P === 'length') {
    if (IsAbsent(Desc.[[Value]])) {
      return A.defaultDefineOwnProperty(P, Desc, Throw)
    } else {
      if (oldLenDesc.[[Writable]] === false) {
        Reject
      }
      let value = ToNumber(Desc.[[Value]])
      let newLen = ToUint32(value)
      if (value !== newLen) {
        throw RangeError
      }
      
      let newLenDesc = copy(Desc)
      newLenDesc.[[Value]] = newLen
      
      if (newLen >= oldLen) {
        return A.defaultDefineOwnProperty(P, newLenDesc, Throw)
      } else {
        let newWritable = newLenDesc.[[Writalbe]] === false ? false : true
        newLenDesc.[[Writable]] = true
        
        let succeeded = A.defaultDefineOwnProperty(P, newLenDesc, Throw)
        if (succeeded === false) {
          return false
        } else {
          while (newLen < oldLen) {
            oldLen--
            let deletedSucceeded = A.[[Delete]](ToString(oldLen), false)
            if (deletedSucceeded === false) {
              newLenDesc.[[Value]] === oldLen + 1
              if (newWritable === false) {
                newLenDesc.[[Writable]] === false
              }
              A.defaultDefineOwnProperty(P, newLenDesc, false)
              Reject
            }
          }
          
          if (newWritable === false) {
            return A.defaultDefineOwnProperty(P, {[[Writable]]: false}, Throw)
          }
        }
      }
    }
  }
  
  if (IsArrayIndex(P)) {
    let index = ToUint32(P)
    if (index >= oldLen && oleLen.[[Writable]] === false) {
      Reject
    }
    
    let succeeded = A.defaultDefineOwnProperty(P, desc, Throw)
    if (succeeded === false) {
      Reject
    }
    
    if (index >= oldLen) {
      oldLenDesc.[[Value]] = index + 1
      return A.defaultDefineOwnProperty('length', oldLenDesc, false)
    }
  }
  
  return A.defaultDefineOwnProperty(P, desc, Throw)
}
```

示例

```javascript
// 非法length
let arr = [1, 2, 3]

arr.length = -1 // 抛出RangeError
arr.length = Math.pow(2, 32) // 抛出RangeError
arr.length = 2.3 // 抛出RangeError
arr.length = 'abc' // 抛出RangeError

// 增长length
let arr = [1, 2, 3]
arr.length = 10 // arr为[1, 2, 3, empty × 7]

// 减少length
let arr = [1, 2, 3]
arr.length = 2 // arr为[1, 2]

// 修改元素
let arr = [1, 2, 3]
arr[1] = 0 // arr为[1, 0, 3]

// 添加元素
let arr = [1, 2, 3]
arr[9] = 0 // arr为[1, 2, 3, empty × 6, 0]

// 设置属性
arr[-1] = 'a'
```
