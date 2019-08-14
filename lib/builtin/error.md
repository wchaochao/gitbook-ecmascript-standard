# Error构造器

标签（空格分隔）： ECMAScript规范

---

## 构造函数

### Error(message)

作为函数使用，等同于作为构造器使用

```javascript
/**
 * 调用new Error
 */
return new Error(message)
```

### new Error(message)

作为构造器使用，创建Error对象

```javascript
/**
 * 参数处理
 *   message为undefined时，置为''
 *   message不为undefined时，转换为String类型
 * 
 * 创建Error对象
 * [[Class]]属性为'Error'
 * [[Prototype]]属性为Error.prototype
 * [[Extensible]]属性为true
 * message属性为message
 */
message = message === undefined ? '' : ToString(message)

let err = new Object()
err.[[Class]] = 'Error'
err.[[Prototype]] = Error.prototype
err.[[Extensible]] = true
err.message = message
```

示例

```javascript
new Error() // Error
new Error('err') // Error: err
```

### 静态属性

```javascript
Error.[[Class]] = 'Function'
Error.[[Prototype]] = Function.Prototype
Error.[[Extensible]] = true

Error.[[DefineOwnProperty]]('length', {
  [[Value]]: 1,
  [[Writable]]: false,
  [[Enumerable]]: false,
  [[Configurable]]: false
}, false)
Error.[[DefineOwnProperty]]('prototype', {
  [[Value]]: {constructor: Error,...},
  [[Writable]]: false,
  [[Enumerable]]: false,
  [[Configurable]]: false
}, false)
```

## 原型对象

Error对象的原型

```javascript
Error.prototype
```

### 原型属性

```javascript
Error.prototype.[[Class]] = 'Error'
Error.prototype.[[Prototype]] = Object.prototype
Error.prototype.[[Extensible]] = true

Error.prototype.constructor = Error
Error.prototype.name = 'Error'
Error.prototype.message = ''
```

### 原型方法

#### Error.prototype.toString( )

获取Error对象的字符串表示

```javascript
/**
 * this不为对象时，抛出TypeError
 * 获取this的name属性
 *   name为undefined时，置为'Error'
 *   name不为undefined时，转换为String类型
 * 获取this的message属性
 *   message为undefined时，置为''
 *   message不为undefined时，转换为String类型
 * name为空字符串时，返回message
 * name不为空字符串时
 *   message为空字符串时，返回name
 *   message不为空字符串时，返回name + ': ' + message
 */
let O = this
if (Type(O) !== Object) {
  throw TypeError
}

let name = O.[[Get]]('name')
name = name === undefined ? 'Error' : ToString(name)
let msg = O.[[Get]]('message')
msg = msg === undefined ? '' : ToString(msg)

if (name === '') {
  return msg
}
if (msg === '') {
  return name
}
return name + ': ' + message
```

示例

```javascript
Error.prototype.toString.call(1) // 抛出TypeError
Error.prototype.toString.call({name: '', message: 'message'}) // 'message'
Error.prototype.toString.call({}) // 'Error'
Error.prototype.toString.call({message: 'message'}) // 'Error: message'
new Error('this is a error').toString() // 'Error: this is a error'
```

## 实例对象

[[Class]]为Error的对象

```javascript
// 通过构造器Error创建
new Error('err')
```

### 实例属性

```javascript
err.[[Class]] = 'Error'
err.[[Prototype]] = Error.prototype
err.[[Extensible]] = true

err.name = 'Error'
err.message = message
```

## NativeError

Error的子类

* `SyntaxError`: 语法错误
* `ReferenceError`: 引用错误
* `RangeError`: 超出有效范围
* `TypeError`: 类型错误
* `URIError`: URI编码/解码错误
* `EvalError`: Eval执行错误

### 实例属性

```javascript
nativeErr.[[Class]] = 'Error'
nativeErr.[[Prototype]] = NativeError.prototype
nativeErr.[[Extensible]] = true

nativeErr.name = NativeErrorName
nativeErr.message = message
```

示例

```javascript
new RangeError('This is a range error') // RangeError: This is a range error
```
