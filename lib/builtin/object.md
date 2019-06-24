# Object构造器

标签（空格分隔）： ECMAScript规范

---

## 构造函数

### Object([value])

作为函数使用，将value转换为Object类型

```javascript
/**
 * value为null、undefined或未提供，返回new Object(value)
 * value为其他，返回ToObject(value)
 */
if (value === null || value === undefined || value === empty) {
  return new Object(value)
} else {
  return ToObject(value)
}
```

示例

```javascript
Object() // {}
Object(undefined) // {}
Object(null) // {}
Object(true) // Boolean(true)
Object(1) // Number{1}
Object('a') // String{'a'}
let o = {}
Object(o) // o
```

### new Object([value])

作为构造器使用，创建Object对象

```javascript
/**
 * value为null、undefined或未提供，返回空对象
 * value为Object类型，返回value本身
 * value为Boolean、Number、String类型，返回value对应的包装对象
 */
if (value === null || value === undefined || value === empty) {
  return {}
} else if (Type(value) === Object) {
  return value
} else if (Type(value) === Boolean || Type(value) === Number || Type(value) === String) {
  return ToObject(value)
}
```

示例

```javascript
new Object() // {}
new Object(undefined) // {}
new Object(null) // {}
new Object(true) // Boolean(true)
new Object(1) // Number{1}
new Object('a') // String{'a'}
new let o = {}
new Object(o) // o
```

### 静态属性

```javascript
Object.[[Class]] = 'Object'
Object.[[Prototype]] = Function.Prototype
Object.[[Extensible]] = true

Object.[[DefineOwnProperty]]('length', {
  [[Value]]: 1,
  [[Writable]]: false,
  [[Enumerable]]: false,
  [[Configurable]]: false
}, false)
Object.[[DefineOwnProperty]]('prototype', {
  [[Value]]: {constructor: Object,...},
  [[Writable]]: false,
  [[Enumerable]]: false,
  [[Configurable]]: false
}, false)
```

### 静态方法

**原型方法**

#### Object.getPrototypeOf(O)

获取对象的原型

```javascript
/**
 * O转换为Object类型，返回[[Prototype]]属性
 */
let obj = ToObject(O)
return obj.[[Prototype]]
```

示例

```javascript
Object.getPrototypeOf() // 抛出TypeError
Object.getPrototypeOf(true) // Boolean.prototype
Object.getPrototypeOf(1) // Number.prototype
Object.getPrototypeOf('a') // String.prototype
Object.getPrototypeOf({}) // Object.prototype
Object.getPrototypeOf([]) // Array.prototype
```

**属性方法**

#### Object.getOwnPropertyNames(O)

获取对象自身属性名组成的数组

```javascript
/**
 * O转换为Object类型，遍历O自身的属性，将属性名存放到一个数组中
 */
let obj = ToObject(O)
let array = new Array()
let n = 0
while (true) {
  let P = nextOwnProperty(obj)
  if (!P) {
    return array
  }
  array.[[DefineOwnProperty]](ToString(n), {
    [[Value]]: P,
    [[Writable]]: true,
    [[Enumerable]]: true,
    [[Configurable]]: true
  }, false)
  n++
}
```

示例

```javascript
Object.getOwnPropertyNames() // 抛出TypeError
Object.getOwnPropertyNames(1) // []
Object.getOwnPropertyNames('abc') // ['0', '1', '2', length]
Object.getOwnPropertyNames({a: 1}) // ['a']
```

#### Object.keys(O)

获取对象自身可枚举属性名组成的数组

```javascript
/**
 * O转换为Object类型，遍历O自身可枚举的属性，将属性名存放到一个数组中
 */
let obj = ToObject(O)
let array = new Array()
let n = 0
while (true) {
  let P = nextOwnEnumerableProperty(obj)
  if (!P) {
    return array
  }
  array.[[DefineOwnProperty]](ToString(n), {
    [[Value]]: P,
    [[Writable]]: true,
    [[Enumerable]]: true,
    [[Configurable]]: true
  }, false)
  n++
}
```

示例

```javascript
Object.keys() // 抛出TypeError
Object.keys(1) // []
Object.keys('abc') // ['0', '1', '2']
Object.keys({a: 1}) // ['a']
```

#### Object.getOwnPropertyDescriptor(O, P)

获取对象自身属性的属性描述对象

```javascript
/**
 * O转换为Object类型，P转换为String类型
 * 获取O自身属性的属性描述符，并转换为属性描述对象
 */
let obj = ToObject(O)
let key = ToString(P)
let desc = obj.[[GetOwnProperty]](key)
return FromPropertyDescriptor(desc)
```

示例

```javascript
Object.getOwnPropertyDescriptor(undefined, 'toString') // 抛出TypeError
Object.getOwnPropertyDescriptor(1, 'toString') // undefined
Object.getOwnPropertyDescriptor({a: 1}, 'a') // {value: 1, writable: true, enumerable: true, configurable: true}
```

#### Object.defineProperty(O, P, Attributes)

设置对象自身属性的属性描述符

```javascript
/**
 * O不为Object类型，抛出TypeError
 * O为Object类型，P转换为String类型，Attributes转换为属性描述符，调用[[DefineOwnProperty]]方法设置属性描述符
 */
if (Type(O) !== Object) {
  throw TypeError
}
let name = ToString(P)
let desc = ToPropertyDescriptor(Attributes)
O.[[DefineOwnProperty]](name, desc, true)
return O
```

示例

```javascript
Object.defineProperty(1, 'a', {value: 1}) // 抛出TypeError

let o = {a: 1}
Object.defineProperty(o, 'b', {get: function () {return 2}}) // {a: 1, b: 2}
Object.defineProperty(o, 'a', {value: 3}) // {a: 3, b: 2}
```

#### Object.defineProperties(O, Properties)

设置多个对象自身属性的属性描述符

```javascript
/**
 * O不为Object类型，抛出TypeError
 * O为Object类型，Properties转换为Object类型，遍历Properties自身的可枚举属性，属性值转换为属性描述符，调用[[DefineOwnProperty]]方法设置属性描述符
 */
if (Type(O) !== Object) {
  throw TypeError
}
let props = ToObject(Properties)
while (true) {
  let P = nextOwnEnumerableProperty(props)
  if (!P) {
    return O
  }
  let descObj = props.[[Get]](P)
  let desc = ToPropertyDescriptor(descObj)
  O.[[DefineOwnProperty]](P, desc, true)
}
```

示例

```javascript
Object.defineProperties(1, {a: {value: 1}}) // 抛出TypeError

let o = {a: 1}
Object.defineProperties(o, { // {a: 3, b: 2}
  a: {value: 3},
  b: {get: function () {return 2}}
})
```

#### Object.create(O [, Properties])

以指定对象为原型创建对象

```javascript
/**
 * O不为Object或Null类型，抛出TypeError
 * O为Object或Null类型，创建新对象，[[Prototype]]属性设为O，并设置多个自身属性的属性描述符
 */
if (Type(O) !== Object && Type(O) !== Null) {
  throw TypeError
}
let obj = new Object()
obj.[[Prototype]] = O
if (Properties !== empty && Properties !== undefined) {
  Object.defineProperties(obj, Properties)
}
return obj
```

示例

```javascript
Object.create() // 抛出TypeError
Object.create(null) // {}
Object.create({a: 1}, { // {b: 2, c: 3, __proto__: {a: 1}}
  b: {
    value: 2
  },
  c: {
    get: function () {
      return 3
    }
  }
})
```

**状态方法**

#### Object.preventExtensions(O)

阻止对象扩张，不能添加属性

```javascript
/**
 * O不为Object类型，直接返回O
 * O为Object类型
 *    设置[[Extensible]]属性为false
 */
if (Type(O) !== Object) {
  return O
}
O.[[Extensible]] = false
return O
```

示例

```javascript
'use strict'

Object.preventExtensions(1) // 1

let o = {a: 1}
Object.preventExtensions(o) // {a: 1}
o.b = 2 // 抛出TypeError
Object.defineProperty(o, 'a', {enumerable: false}) // a不可枚举
o.a = 2 // 2
delete o.a // true
```

#### Object.seal(O)

密封对象，不能添加属性、配置、删除属性

```javascript
/**
 * O不为Object类型，直接返回O
 * O为Object类型
 *    设置[[Extensible]]属性为false
 *    遍历O自身的属性，设置属性描述符[[Configurable]]为false
 */
if (Type(O) !== Object) {
  return O
}
O.[[Extensible]] = false
while (true) {
  let P = nextOwnProperty(O)
  if (!P) {
    return O
  }
  let desc = O.[[GetOwnProperty]](P)
  if (desc.[[Configurable]] === true) {
    desc.[[Configurable]] = false
  }
  O.[[DefineOwnProperty]](P, desc, true)
}
```

示例

```javascript
'use strict'

Object.seal(1) // 1

let o = {a: 1}
Object.seal(o) // {a: 1}
o.b = 2 // 抛出TypeError
Object.defineProperty(o, 'a', {enumerable: false}) // 抛出TypeError
delete o.a // 抛出TypeError
o.a = 2 // 2
```

#### Object.freeze(O)

冻结对象，不能添加属性、配置、删除、修改自身属性

```javascript
/**
 * O不为Object类型，直接返回O
 * O为Object类型
 *    设置[[Extensible]]属性为false
 *    遍历O自身的属性，设置属性描述符[[Configurable]]、[[Writable]]为false
 */
if (Type(O) !== Object) {
  return O
}
O.[[Extensible]] = false
while (true) {
  let P = nextOwnProperty(O)
  if (!P) {
    return O
  }
  let desc = O.[[GetOwnProperty]](P)
  if (IsDataDescriptor(desc) && desc.[[Writable]] === true) {
    desc.[[Writable]] = false
  }
  if (desc.[[Configurable]] === true) {
    desc.[[Configurable]] = false
  }
  O.[[DefineOwnProperty]](P, desc, true)
}
```

示例

```javascript
'use strict'

Object.freeze(1) // 1

let o = {a: 1}
Object.freeze(o) // {a: 1}
o.b = 2 // 抛出TypeError
Object.defineProperty(o, 'a', {enumerable: false}) // 抛出TypeError
delete o.a // 抛出TypeError
o.a = 2 // 抛出TypeError
```

#### Object.isExtensible(O)

对象是否可扩展

```javascript
/**
 * O不为Object类型，直接返回false
 * O为Object类型，返回O.[[Extensible]]
 */
if (Type(O) !== Object) {
  return false
} else {
  return O.[[Extensible]]
}
```

示例

```javascript
Object.isExtensible(1) // false

let o = {a: 1}
Object.isExtensible(o) // true

Object.preventExtensions(o)
Object.isExtensible(o) // false

Object.seal(o)
Object.isExtensible(o) // false

Object.freeze(o)
Object.isExtensible(o) // false
```

#### Object.isSealed(O)

对象是否密封

```javascript
/**
 * O不为Object类型，直接返回true
 * O为Object类型
 *    [[Extensible]]属性为false且所有自身属性的[[Configurable]]为false时，返回true
 *    其他情况返回false
 */
if (Type(O) !== Object) {
  return true
}
if (O.[[Extensible]] === true) {
  return false
}
while (true) {
  let P = nextOwnProperty(O)
  if (!P) {
    return true
  }
  let desc = O.[[GetOwnProperty]](P)
  if (desc.[[Configurable]] === true) {
    return false
  }
}
```

示例

```javascript
Object.isSealed(1) // true

let o = {a: 1}
Object.isSealed(o) // false

Object.preventExtensions(o)
Object.isSealed(o) // false

Object.seal(o)
Object.isSealed(o) // true

Object.freeze(o)
Object.isSealed(o) // true
```

#### Object.isFrozen(O)

对象是否冻结

```javascript
/**
 * O不为Object类型，直接返回true
 * O为Object类型
 *    [[Extensible]]属性为false且所有自身属性的[[Configurable]]、[[Writable]]为false时，返回true
 *    其他情况返回false
 */
if (Type(O) !== Object) {
  return true
}
if (O.[[Extensible]] === true) {
  return false
}
while (true) {
  let P = nextOwnProperty(O)
  if (!P) {
    return true
  }
  let desc = O.[[GetOwnProperty]](P)
  if (IsDataDescriptor(desc) && desc.[[Writable]] === true) {
    return false
  }
  if (desc.[[Configurable]] === true) {
    return false
  }
}
```

示例

```javascript
Object.isFrozen(1) // true

let o = {a: 1}
Object.isFrozen(o) // false

Object.preventExtensions(o)
Object.isFrozen(o) // false

Object.seal(o)
Object.isFrozen(o) // false

Object.freeze(o)
Object.isFrozen(o) // true
```

## 原型对象

Object对象的原型

```javascript
Object.prototype
```

### 原型属性

```javascript
Object.prototype.[[Class]] = 'Object'
Object.prototype.[[Prototype]] = null
Object.prototype.[[Extensible]] = true

Object.prototype.constructor = Object
```

### 原型方法

**转换方法**

#### Object.prototype.toString( )

获取对象的字符串表示

```javascript
/**
 * this为undefined, 返回'[object Undefined]'
 * this为null, 返回'[object Null]'
 * this为其他，转换为Object类型，返回`[object ${O.[[Class]]}]`
 */
if (this === undefined) {
  return '[object Undefined]'
}

if (this === null) {
  return '[object Null]'
}

let O = ToObject(this)
return `[object ${O.[[Class]]}]`
```

示例

```javascript
Object.prototype.toString.call(undefined) // '[object Undefined]'
Object.prototype.toString.call(null) // '[object Null]'
Object.prototype.toString.call(true) // '[object Boolean]'
Object.prototype.toString.call(1) // '[object Number]'
Object.prototype.toString.call('a') // '[object String]'
Object.prototype.toString.call({}) // '[object Object]'
Object.prototype.toString.call([]) // '[object Array]'
Object.prototype.toString.call(function () {}) // '[object Function]'
Object.prototype.toString.call(new Date()) // '[object Date]'
Object.prototype.toString.call(/a/) // '[object RegExp]'
Object.prototype.toString.call(new Error()) // '[object Error]'
Object.prototype.toString.call(Math) // '[object Math]'
Object.prototype.toString.call(JSON) // '[object JSON]'
```

#### Object.prototype.toLocaleString( )

获取对象的本地字符串表示

```javascript
/**
 * this转换为Object类型O，调用O的toString()方法
 */
let O = ToObject(O)
return O.toString(O)
```

示例

```javascript
Object.prototype.toLocaleString.call(undefined) // 抛出TypeError
Object.prototype.toLocaleString.call(null) // 抛出TypeError
Object.prototype.toLocaleString.call(true) // 'true'
Object.prototype.toLocaleString.call(1) // '1'
Object.prototype.toLocaleString.call('a') // 'a'
Object.prototype.toLocaleString.call({}) // '[object Object]'
Object.prototype.toLocaleString.call([]) // ''
Object.prototype.toLocaleString.call(function () {}) // 'function () {}'
Object.prototype.toLocaleString.call(new Date()) // 'Thu Jun 06 2019 14:42:02 GMT+0800 (中国标准时间)'
Object.prototype.toLocaleString.call(/a/) // '/a/'
Object.prototype.toLocaleString.call(new Error('aaa')) // 'Error: aaa'
Object.prototype.toLocaleString.call(Math) // '[object Math]'
Object.prototype.toLocaleString.call(JSON) // '[object JSON]'
```

#### Object.prototype.valueOf( )

获取对象的值表示

```javascript
/**
 * this转换为Object类型并返回
 */
let O = ToObject(this)
return O
```

示例

```javascript
Object.prototype.valueOf.call(undefined) // 抛出TypeError
Object.prototype.valueOf.call(null) // 抛出TypeError
Object.prototype.valueOf.call(true) // Boolean{true}
Object.prototype.valueOf.call(1) // Number{1}
Object.prototype.valueOf.call('a') // String{'a'}
Object.prototype.valueOf.call({}) // {}
Object.prototype.valueOf.call([]) // []
Object.prototype.valueOf.call(function () {}) // function () {}
Object.prototype.valueOf.call(new Date()) // new Date()
Object.prototype.valueOf.call(/a/) // /a/
Object.prototype.valueOf.call(new Error()) // new Error()
Object.prototype.valueOf.call(Math) // Math
Object.prototype.valueOf.call(JSON) // JSON
```

**检测方法**

#### Object.prototype.hasOwnProperty(V)

判断V是否是对象自身的属性

```javascript
/**
 * V转换为String类型P，this转换为Object类型O
 * 获取对象O自身属性P的属性描述符
 *     属性描述符不存在，返回false
 *     属性描述符存在，返回true
 */
let P = ToString(V)
let O = ToObject(this)
let desc = O.[[GetOwnProperty]](P)
if (desc === undefined) {
  return false
} else {
  return true
}
```

示例

```javascript
Object.prototype.hasOwnProperty.call(undefined, 'toString') // 抛出TypeError
true.hasOwnProperty('toString') // false
'a'.hasOwnProperty('0') // true
'a'.hasOwnProperty('length') // true
({a: 1}).hasOwnProperty('a') // true
```

#### Object.prototype.propertyIsEnumerable(V)

判断V是否是对象自身的可枚举属性

```javascript
/**
 * V转换为String类型P，this转换为Object类型O
 * 获取对象O自身属性P的属性描述符
 *     属性描述符不存在，返回false
 *     属性描述符存在，返回desc.[[Enumerable]]
 */
let P = ToString(V)
let O = ToObject(P)
let desc = O.[[GetOwnProperty]](P)
if (desc === undefined) {
  return false
} else {
  return desc.[[Enumerable]]
}
```

示例

```javascript
Object.prototype.propertyIsEnumerable.call(undefined, 'toString') // 抛出TypeError
true.propertyIsEnumerable('toString') // false
'a'.propertyIsEnumerable('0') // true
'a'.propertyIsEnumerable('length') // false
({a: 1}).propertyIsEnumerable('a') // true
```

#### Object.prototype.isPrototypeOf(V)

判断对象是否在V的原型链上

```javascript
/**
 * V不为Object类型，直接返回false
 * V为Object类型，this转换为Object类型O
 *     O在V的原型链上，返回true
 *     O不在V的原型链上，返回false
 */
if (Type(V) !== Object) {
  return false
}
let O = ToObject(this)
while (true) {
  V = V.[[Prototype]]
  if (V === null) {
    return false
  }
  if (V === O) {
    return true
  }
}
```

示例

```javascript
Object.prototype.isPrototypeOf.call(undefined, 1) // 返回false
Object.prototype.isPrototypeOf.call(undefined, {a: 1}) // 抛出TypeError
Array.prototype.isPrototypeOf([]) // true
Object.prototype.isPrototypeOf([]) // true
```

## 实例对象

[[Class]]为Object的对象

```javascript
// 通过对象直接量创建
{a: 1}

// 通过构造器Object创建
new Object()

// 通过Object.create()创建
Object.create(null)
```

### 实例属性

```javascript
obj.[[Class]] = 'Object'
obj.[[Prototype]] = Object.prototype
obj.[[Extensible]] = true
```
