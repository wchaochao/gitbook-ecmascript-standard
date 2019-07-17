# String构造器

标签（空格分隔）： ECMAScript规范

---

## 构造函数

### String([ value ])

作为函数使用，转换为String类型

```javascript
/**
 * value未提供，返回''
 * value已提供，转换为String类型
 */
if (isAbsent(value)) {
  return ''
} else {
  return ToString(value)
}
```

示例

```javascript
String() // ''
String(undefined) // 'undefined'
String(null) // 'null'
String(true) // 'true'
String(false) // 'false'
String(NaN) // 'NaN'
String(+0) // '0'
String(-0) // '0'
String(Infinity) // 'Infinity'
String(-Infinity) // '-Infinity'
String(12.345) // '12.345'
String(0.0000001) // '1e-7'
String({}) // '[object, Object]'
```

### new String([ value ])

作为构造器使用，创建String对象

```javascript
/**
 * 创建String对象
 * [[Class]]属性为'String'
 * [[Prototype]]属性为String.prototype
 * [[Extensible]]属性为true
 * [[PrimitiveValue]]属性为String([value])
 */
let str = new Object()
str.[[Class]] = 'String'
str.[[Prototype]] = String.prototype
str.[[Extensible]] = true
str.[[PrimitiveValue]] = isAbsent(value) ? '' : ToString(value)
```

示例

```javascript
new String() // String{''}
new String(undefined) // String{'undefined'}
new String(null) // String{'null'}
new String(true) // String{'true'}
new String(false) // String{'false'}
new String(NaN) // String{'NaN'}
new String(+0) // String{'0'}
new String(-0) // String{'0'}
new String(Infinity) // String{'Infinity'}
new String(-Infinity) // String{'-Infinity'}
new String(12.345) // String{'12.345'}
new String(0.0000001) // String{'1e-7'}
new String({}) // String{'[object, Object]'}
```

### 静态属性

```javascript
String.[[Class]] = 'Function'
String.[[Prototype]] = Function.Prototype
String.[[Extensible]] = true

String.[[DefineOwnProperty]]('length', {
  [[Value]]: 1,
  [[Writable]]: false,
  [[Enumerable]]: false,
  [[Configurable]]: false
}, false)
String.[[DefineOwnProperty]]('prototype', {
  [[Value]]: {constructor: String,...},
  [[Writable]]: false,
  [[Enumerable]]: false,
  [[Configurable]]: false
}, false)
```

### 静态方法

#### String.fromCharCode([char0 [ , char1 [ , … ] ] ] )

获取UInt16类型数字对应的字符串

```javascript
/**
 * 无参数，返回''
 * 有参数，全部转换为UInt16类型，再组成字符串
 */
if (arguments.length === 0) {
  return ''
} else {
  let codes = Array.prototype.map(arguments, arg => ToUint16(arg))
  return parseAsString(codes)
}
```

示例

```javascript
String.fromCharCode() // ''
String.fromCharCode(undefined) // ' '
String.fromCharCode(97, 98, 99) // 'abc'
String.fromCharCode(0x20BB7) === String.fromCharCode(0x0BB7) // true
String.fromCharCode('0xD834', '0xDF06') // 多字节字符
```

## 原型对象

String对象的原型

```javascript
String.prototype
```

### 原型属性

```javascript
String.prototype.[[Class]] = 'String'
String.prototype.[[Prototype]] = Object.prototype
String.prototype.[[Extensible]] = true
String.prototype.[[Primitive]] = ''

String.prototype.constructor = String
String.prototype.length = 0
```

### 原型方法

**转换方法**

#### String.prototype.toString( )

获取String对象的字符串表示

```javascript
/**
 * this不为String类型或String对象，抛出TypeError
 * this为String类型或String对象，返回[[PrimitiveValue]]值
 */
if (Object.prototype.toString.call(this) !== '[object String]') {
  throw TypeError
}
return this.[[PrimitiveValue]]
```

示例

```javascript
String.prototype.toString.call(true) // 抛出TypeError
'a'.toString() // 'a'
```

#### String.prototype.valueOf( )

获取String对象的值表示

```javascript
/**
 * this不为String类型或String对象，抛出TypeError
 * this为String类型或String对象，返回[[PrimitiveValue]]值
 */
if (Object.prototype.toString.call(this) !== '[object String]') {
  throw TypeError
}
return this.[[PrimitiveValue]]
```

示例

```javascript
String.prototype.valueOf.call(true) // 抛出TypeError
'a'.valueOf() // 'a'
```

**大小写方法**

#### String.prototype.toLowerCase( )

转换为小写

```javascript
/**
 * 检测this能否转换为对象
 * this转换为String类型S
 * 将S内的字符转换为Unicode对应的小写字符
 */
CheckObjectCoercible(this)
let S = ToString(this)

return toUnicodeLowerCase(S)
```

示例

```javascript
String.prototype.toLowerCase.call(undefined) // TypeError

'ABC'.toLowerCase() // 'abc'

String.prototype.toLowerCase.call(true) // 'true'
```

#### String.prototype.toLocaleLowerCase( )

转换为本地小写

```javascript
/**
 * 检测this能否转换为对象
 * this转换为String类型S
 * 将S内的字符转换为宿主环境对应的小写字符
 */
CheckObjectCoercible(this)
let S = ToString(this)

return toHostLowerCase(S)
```

示例

```javascript
String.prototype.toLocaleLowerCase.call(undefined) // TypeError

'ABC'.toLocaleLowerCase() // 'abc'

String.prototype.toLocaleLowerCase.call(true) // 'true'
```

#### String.prototype.toUpperCase( )

转换为大写

```javascript
/**
 * 检测this能否转换为对象
 * this转换为String类型S
 * 将S内的字符转换为Unicode对应的大写字符
 */
CheckObjectCoercible(this)
let S = ToString(this)

return toUnicodeUpperCase(S)
```

示例

```javascript
String.prototype.toUpperCase.call(undefined) // TypeError

'abc'.toUpperCase() // 'ABC'

String.prototype.toUpperCase.call(true) // 'TRUE'
```

#### String.prototype.toLocaleUpperCase( )

转换为本地大写

```javascript
/**
 * 检测this能否转换为对象
 * this转换为String类型S
 * 将S内的字符转换为宿主环境对应的大写字符
 */
CheckObjectCoercible(this)
let S = ToString(this)

return toHostUpperCase(S)
```

示例

```javascript
String.prototype.toLocaleUpperCase.call(undefined) // TypeError

'abc'.toLocaleUpperCase() // 'ABC'

String.prototype.toLocaleUpperCase.call(true) // 'TRUE'
```

**字符方法**

#### String.prototype.charAt(pos)

获取pos位置的16位二进制对应的字符

```javascript
/**
 * 检测this能否转换为对象
 * this转换为String类型S，获取length属性len
 * pos转换为整数position
 *     position在[0, len)内，返回position位置的16位二进制对应的字符
 *     position不在[0, len)内，返回''
 */
CheckObjectCoercible(this)
let S = ToString(this)
let len = S.length

let position = ToInteger(pos)
if (position < 0 || position >= len) {
  return ''
} else {
  return getUnitChar(S[position])
}
```

示例

```javascript
String.prototype.charAt.call(undefined) // TypeError

'abc'.charAt(-1) // ''
'abc'.charAt(3) // ''
'abc'.charAt(1.2) // 'b'
'\uD834\uDF06'.charAt(0) // '\uD834' 多字节字符

String.prototype.charAt.call(true) // 't'
```

#### String.prototype.charCodeAt(pos)

获取pos位置的16位二进制对应的数字（UInt16类型）

```javascript
/**
 * 检测this能否转换为对象
 * this转换为String类型S，获取length属性len
 * pos转换为整数position
 *     position在[0, len)内，返回position位置的16位二进制对应的数字
 *     position不在[0, len)内，返回NaN
 */
CheckObjectCoercible(this)
let S = ToString(this)
let len = S.length

let position = ToInteger(pos)
if (position < 0 || position >= len) {
  return NaN
} else {
  return getUnitNumber(S[position])
}
```

示例

```javascript
String.prototype.charCodeAt.call(undefined) // TypeError

'abc'.charCodeAt(-1) // NaN
'abc'.charCodeAt(3) // NaN
'abc'.charCodeAt(1.2) // 98
'\uD834\uDF06'.charCodeAt(0) // 0xD834 多字节字符

String.prototype.charCodeAt.call(true) // 116
```

**操作方法**

#### String.prototype.concat([string1 [ , string2 [ , … ] ] ] )

字符串拼接

```javascript
/**
 * 检测this能否转换为对象
 * this转换为String类型S
 * 遍历参数，转换为String类型，与S拼接在一起
 */
CheckObjectCoercible(this)
let S = ToString(this)

let R = S
for (let i = 0; i < arguments.length; i++) {
  R += ToString(arguments[i])
}
return R
```

示例

```javascript
String.prototype.concat.length // 1

String.prototype.concat.call(undefined) // 抛出TypeError

'a'.concat() // 'a'
'a'.concat(undefined, true, 1, {}) // 'aundefinedtrue1[object Object]'
'a' + true // 'atrue'

String.prototype.concat.call(true, 2) // 'true2'
```

#### String.prototype.slice(start, end)

字符串截取，指定开始点和结束点

```javascript
/**
 * 检测this能否转换为对象
 * this转换为String类型S，获取length属性len
 * start转换为整数，为负数时加len
 *      小于0时置为0，大于len时置为len，落在[0, len]之间
 * end为undefined时，置为len
 * end不为undefined时，与start同样处理
 * 截取S中[start, end)的字符串并返回
 */
CheckObjectCoercible(this)
let S = ToString(this)
let len = S.length

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

return slice(S, start, end)
```

示例

```javascript
String.prototype.slice.length // 2

String.prototype.slice.call(undefined) // 抛出TypeError

'abcba'.slice() // 'abcba'
'abcba'.slice(1, -1) // 'bcb'
'abcba'.slice(-2, 3) // 'b'
'abcba'.slice(2.2, 1.1) // ''

String.prototype.slice.call(true, 2) // 'ue'
```

#### String.prototype.substring(start, end)

字符串截取，指定端点

```javascript
/**
 * 检测this能否转换为对象
 * this转换为String类型S，获取length属性len
 * start转换为整数，为负数时加len
 *      小于0时置为0，大于len时置为len，落在[0, len]之间
 * end为undefined时，置为len
 * end不为undefined时，与start同样处理
 * from为start, end中的小值，to为start, end中的大值
 * 截取S中[from, to)的字符串并返回
 */
CheckObjectCoercible(this)
let S = ToString(this)
let len = S.length

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

let from = min(start, end)
let to = max(start, end)

return slice(S, from, to)
```

示例

```javascript
String.prototype.substring.length // 2

String.prototype.substring.call(undefined) // 抛出TypeError

'abcba'.substring() // 'abcba'
'abcba'.substring(1, -1) // 'bcb'
'abcba'.substring(-2, 3) // 'b'
'abcba'.substring(2.2, 1.1) // 'b'

String.prototype.slice.call(true, 2) // 'ue'
```

#### String.prototype.localeCompare(that)

字符串本地比较

```javascript
/**
 * 检测this能否转换为对象
 * this转换为String类型S
 * that转换为String类型
 * 按本地比较方法比较S、that
 *     S < that, 返回-1
 *     S = that, 返回0
 *     S > that, 返回1
 */
CheckObjectCoercible(this)
let S = ToString(this)
that = ToString(that)

return localeCompare(S, that)
```

示例

```javascript
String.prototype.localeCompare.call(undefined) // 抛出TypeError

'b'.localeCompare('a') // 1
'b'.localeCompare('b') // 0
'b'.localeCompare('c') // -1
'b'.localeCompare('B') // -1 本地比较大写字母在小写字母后面

String.prototype.localeCompare.call(true, 2) // 1
```

#### String.prototype.trim( )

字符串去除前置、后置空白

```javascript
/**
 * 检测this能否转换为对象
 * this转换为String类型S
 * 去除S的前置、后置空白
 */
CheckObjectCoercible(this)
let S = ToString(this)

return trim(S)
```

示例

```javascript
String.prototype.trim.call(undefined) // 抛出TypeError

' \nb\t '.trim() // 'b'

String.prototype.trim.call(true, 2) // 'true'
```

**位置方法**

#### String.prototype.indexOf(searchString, position)

按升序从指定位置开始查找子字符串

```javascript
/**
 * 检测this能否转换为对象
 * this转换为String类型S，获取length属性len
 * searchString转换为String类型
 * position为undefined，置为0
 * position不为undefined，转换为整数
 *      小于0时置为0，大于len时置为len，落在[0, len]之间
 * 遍历[position, len]，查找子串searchString
 *     能找到子串，返回对应的位置
 *     不能找到子串，返回-1
 */
CheckObjectCoercible(this)
let S = ToString(this)
let len = S.length

function format (num) {
  num = ToInterger(num)
  return min(max(num, 0), len)
}

searchString = ToString(searchString)
position = position === undefined ? 0 : format(position)

let searchLen = searchString.length
for (let i = position; i <= len; i++) {
  let str = getSubString(S, i, searchLen)
  if (str === searchString) {
    return i
  }
}

return -1
```

示例

```javascript
String.prototype.indexOf.length // 1

String.prototype.indexOf.call(undefined) // 抛出TypeError

'abcba'.indexOf() // -1
'abcba'.indexOf('') // 0
'abcba'.indexOf('', 5) // 5
'abcba'.indexOf('a') // 0
'abcba'.indexOf('a', 2) // 4
'abcba'.indexOf('a', 5) // -1

String.prototype.indexOf.call(true, 't') // 0
```

#### String.prototype.lastIndexOf(searchString, position)

按降序从指定位置开始查找子字符串

```javascript
/**
 * 检测this能否转换为对象
 * this转换为String类型S，获取length属性len
 * searchString转换为String类型
 * position为undefined，置为len
 * position不为undefined，转换为整数
 *      小于0时置为0，大于len时置为len，落在[0, len]之间
 * 反向遍历[0, position]，查找子串searchString
 *     能找到子串，返回对应的位置
 *     不能找到子串，返回-1
 */
CheckObjectCoercible(this)
let S = ToString(this)
let len = S.length

function format (num) {
  num = ToInterger(num)
  return min(max(num, 0), len)
}

searchString = ToString(searchString)
position = position === undefined ? len : format(position)

let searchLen = searchString.length
for (let i = position; i >= 0; i--) {
  let str = getSubString(S, i, searchLen)
  if (str === searchString) {
    return start
  }
}

return -1
```

示例

```javascript
String.prototype.lastIndexOf.length // 1

String.prototype.lastIndexOf.call(undefined) // 抛出TypeError

'abcba'.lastIndexOf() // -1
'abcba'.lastIndexOf('') // 5
'abcba'.lastIndexOf('', 0) // 0
'abcba'.lastIndexOf('a') // 4
'abcba'.lastIndexOf('a', 2) // 0
'abcba'.lastIndexOf('a', -1) // 0

String.prototype.lastIndexOf.call(true, 't') // 0
```

**模式匹配方法**

#### String.prototype.match(regexp)

字符串匹配

```javascript
/**
 * 检测this能否转换为对象
 * this转换为String类型S
 * regexp不为正则时转换为正则
 * 非全局匹配，调用RegExp.prototype.exec方法，this为regexp, 参数为List{S}
 *     无匹配值，返回null
 *     有匹配值，返回数组A, A = [匹配值, ...捕获组], A.input = S, A.index = 匹配开始的位置
 * 全局匹配，遍历调用RegExp.prototype.exec方法，this为regexp, 参数为List{S}
 *     无匹配值，返回null
 *     有匹配值，返回数组A, A = [...匹配值]
 */
CheckObjectCoercible(this)
let S = ToString(this)

let rx = Object.prototype.toString.call(regexp) === '[object RegExp]' ? regexp : new RegExp(regexp)
let global = rx.[[Get]]('global')
let exec = RegExp.prototype.exec

if (global === false) {
  return exec.[[Call]](rx, S)
} else {
  rx.[[Put]]('lastIndex', 0)
  
  let A = new Array()
  let n = 0
  let previousLastIndex = 0
  while (true) {
    let result = exec.[[Call]](rx, S)
    if (result === null) {
      break
    } else {
      let lastIndex = rx.[[Get]]('lastIndex')
      if (lastIndex === previousLastIndex) { // 处理匹配/(?:)/g的情况
        lastIndex++
        rx.[[Put]]('lastIndex', lastIndex)
      }
      previousLastIndex = lastIndex
      
      let matchStr = result.[[Get]]('0')
      A.[[DefineOwnProperty]](ToString(n), {
        [[Value]]: matchStr,
        [[Writable]]: true,
        [[Enumerable]]: true,
        [[Configurable]]: true
      }, false)
      n++
    }
  }
  
  return n === 0 ? null : A
}
```

示例

```javascript
String.prototype.match.call(undefined) // 抛出TypeError

''.match() // ['', index: 0, input: '']
'abcba'.match() // ['', index: 0, input: 'abcba']
'abcba'.match('a') // ['a', index: 0, input: 'abcba']
'abcba'.match(/(a)/) // ['a', 'a', index: 0, input: 'abcba']
'abcba'.match(/(a)/g) // ['a', 'a']

String.prototype.match.call(true, 't') // ['t', index: 0, input: 'abcba']
```

#### String.prototype.search(regexp)

字符串搜索

```javascript
/**
 * 检测this能否转换为对象
 * this转换为String类型S
 * regexp不为正则时转换为正则
 * 在S中查找匹配regexp的子串
 *     能找到子串，返回对应的位置
 *     不能找到子串，返回-1
 */
CheckObjectCoercible(this)
let S = ToString(this)

let rx = Object.prototype.toString.call(regexp) === '[object RegExp]' ? regexp : new RegExp(regexp)
let index = indexOf(S, rx)
return index
```

示例

```javascript
String.prototype.search.call(undefined) // 抛出TypeError

'abcba'.search() // 0
'abcba'.search('') // 0
'abcba'.search('/(?:)/') // -1
'abcba'.search(/(?:)/) // 0
'abcba'.search(/ba*/) // 1

String.prototype.search.call(true, 't') // 0
```

#### String.prototype.replace(searchValue, replaceValue)

字符串替换

```javascript
/**
 * 检测this能否转换为对象
 * this转换为String类型S
 * searchValue不为正则时转换为字符串
 * replaceValue不为函数时转换为字符串
 * 对S进行searchValue匹配
 *     非全局匹配，只匹配一次
 *     全局匹配，连续匹配
 * 对匹配进行替换
 *     replaceValue为String类型，将replaceValue中的$转义后再替换
 *     replaceValue为函数，执行replaceValue, 参数为List{匹配值, ...捕获组, 匹配开始位置, S}，返回值转换为String类型再替换
 * 返回替换后的字符串
 */
CheckObjectCoercible(this)
let S = ToString(this)

let global = false
if (Object.prototype.toString.call(searchValue) !== '[object RegExp]') {
  searchValue = ToString(searchValue)
} else {
  global = searchValue.[[Get]]('global')
}

if (typeof replaceValue !== 'function') {
  replaceValue = ToString(replaceValue)
}

let R = S
while (true) {
  let result = match(searchValue, S) // [匹配值, ...捕获组, index, input]
  if (result === null) {
    return R
  } else {
    let replaceStr
    if (typeof replaceValue === 'string') {
      replaceStr = parseEscape(replaceValue, result)
    } else {
      replaceStr = ToString(replaceValue(...result, result.index, result.input))
    }
    replace(R, result, replaceStr)
    if (global === false) {
      break
    }
  }
}
```

$转义

| Characters | Replacement text |
| --- | --- |
| $$ | $ |
| $& | 匹配值 |
| $` | 匹配值左边的字符串 |
| $' | 匹配值右边的字符串 |
| $n | 第n个捕获组, n为1-9 |
| $nn | 第nn个捕获组, nn为01-99 |

示例

```javascript
String.prototype.replace.call(undefined) // 抛出TypeError

'abcba'.replace() // 'abcba'
'abcba'.replace('', 'a') // 'aabcba'
'abcba'.replace('ab', '$&$$') // 'ab$cba'
'abcba'.replace('ab', (match) => match + '$') // 'ab$cba'
'abcba'.replace(/(?:)/, 'a') // 'aabcba'
'abcbab'.replace(/a(b)/, '$&$1') // 'abbcbab'
'abcbab'.replace(/a(b)/, (match, p1) => match + p1) // 'abbcbab'
'abcba'.replace(/(?:)/g, 'a') // 'aaabacabaaa'
'abcbab'.replace(/a(b)/g, '$&$1') // 'abbcbabb'
'abcbab'.replace(/a(b)/g, (match, p1) => match + p1) // 'abbcbabb'

String.prototype.replace.call(true, 't') // 'undefinedrue'
```

#### String.prototype.split(separator, limit)

字符串分隔

```javascript
/**
 * 检测this能否转换为对象
 * this转换为String类型S，获取length属性len
 * limit处理
 *    limit为undefined，置为2^32 - 1
 *    limit不为undefined，转换为UInt32类型
 *    limit为0，返回空数组
 * separator处理
 *    separtor为undefined，返回[S]
 *    separtor不为正则，转换为字符串
 * 分隔处理
 *    S为空字符串
 *       separator匹配S，返回空数组
 *       separator不匹配S，返回[S]
 *    S不为空字符串
 *       separtor对S进行连续匹配，用匹配值分隔字符串成数组
 *           匹配为空且未分隔出字符串时直接进行下一次匹配
 *           有捕获组时，捕获组也会被添加到数组中
 *           到达limit限制时不再添加
 */
CheckObjectCoercible(this)
let S = ToString(this)
let len = S.length

let A = new Array()
let n = 0

limit = limit === undefined ? Math.pow(2, 32) - 1 : ToUint32(limit)
if (limit === 0) {
  return A
}

let R = seprator
if (R === undefined) {
  A.[[DefineOwnProperty]]('0', {
    [[Value]]: S,
    [[Writable]]: true,
    [[Enumberable]]: true,
    [[Configurable]]: true
  })
  return A
} else if (Object.prototype.toString.call(R) !== '[object RegExp]') {
  R = ToString(R)
}

if (len === 0) {
  let result = match(R, S, 0)
  if (result === null) {
    A.[[DefineOwnProperty]]('0', {
      [[Value]]: S,
      [[Writable]]: true,
      [[Enumberable]]: true,
      [[Configurable]]: true
    })
    return A
  } else {
    return A
  }
}

function addElement (T) {
  A.[[DefineOwnProperty]](ToString(n), {
    [[Value]]: T,
    [[Writable]]: true,
    [[Enumberable]]: true,
    [[Configurable]]: true
  })
  n++
}

let p = 0
let q = p
while (q !== s) {
  let result = match(R, S, q)
  if (result === null) {
    q++
  } else {
    if (result.index === q) {
      if (q > p) {
        let T = slice(S, p, q)
        addElement(T)
        if (n === limit) {
          return A
        }
      }
      q++
      continue
    }
    let T = slice(S, p, q)
    addElement(T)
    if (n === limit) {
      return A
    }
    for (let i = 1; i < result.length; i++) {
      addElement(result[i])
      if (n === limit) {
        return A
      }
    }
    q = p = result.index
  }
}

let T = slice(S, p, q)
A.[[DefineOwnProperty]](ToString(n), {
  [[Value]]: T,
  [[Writable]]: true,
  [[Enumberable]]: true,
  [[Configurable]]: true
})
return A
```

示例

```javascript
String.prototype.split.length // 2

String.prototype.split.call(undefined) // 抛出TypeError

'abcba'.split('a', 0) // []
'abcba'.split() // ['abcba']
''.split('') // []
''.split(/a*/) // []
''.split('a') // ['']
'abcba'.split('') // ['a', 'b', 'c', 'b', 'a']
'abcba'.split(/a*?/) // ['a', 'b', 'c', 'b', 'a']
'abcba'.split(/d*/) // ['a', 'b', 'c', 'b', 'a']
'abcba'.split(/a*/) // ['', 'b', 'c', 'b', '']
'abcba'.split(/a+/) // ['', 'bcb', '']
'abcba'.split(/(a)/) // ['', 'a', 'bcb', 'a', '']
'abcba'.split(/(a)/, 3) // ['', 'a', 'bcb']

String.prototype.split.call(true, 2) // ['true']
```

## 实例对象

[[Class]]为String的对象

```javascript
// 通过构造器String创建
new String(1)

// 通过构造器Object创建
new Object('a')
```

### 实例属性

```javascript
S.[[Class]] = 'String'
S.[[Prototype]] = String.prototype
S.[[Extensible]] = true
S.[[PrimitiveValue]] = str

S.[[DefineOwnProperty]]('length', {
  [[Value]]: str.length,
  [[Writable]]: false,
  [[Enumerable]]: false,
  [[Configurable]]: false
})
```

### 实例方法

#### S.[[GetOwnProperty]] (P)

```javascript
/**
 * P为普通属性，返回属性的属性描述符
 * P为索引属性，返回数据属性描述符
 *    值为索引对应的字符
 *    不可写、不可配置
 *    可枚举
 * P为其他，返回undefined
 */
S.defaultGetOwnProperty = S.[[GetOwnProperty]]
S.[[GetOwnProperty]] = function (P) {
  let desc = S.defaultGetOwnProperty(P)
  if (desc !== undefined) {
    return desc
  }
  
  let index = ToInteger(P)
  if (ToString(abs(index)) !== P) {
    return undefined
  }
  
  let str = S.[[PrimitiveValue]]
  let len = str.length
  if (index >= len) {
    return undefined
  }
  let resultStr = getChar(S, index)
  return Property Descriptor {
    [[Value]]: resultStr,
    [[Writable]]: false,
    [[Enumerable]]: true,
    [[Configurable]]: false
  }
}
```

示例

```javascript
let S = new String('abc')

S.length // 3
S[3] // undefined

for (let i in S) {
  console.log(i)
}
// 0
// 1
// 2

S[0] = 1 // 严格模式抛出TypeError
delet S[0] // 严格模式抛出TypeError
```
