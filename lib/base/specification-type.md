# 规范类型

标签（空格分隔）： ECMAScript规范

---

只存在于规范里的抽象类型

* `Reference`
* `List`
* `Completion`
* `Property Descriptor` 
* `Property Identifier`
* `Environment Record`
* `Lexical Environment`

## Reference类型

引用类型

* 标识符、对象属性的环境描述

### 组成

| 组成 | 类型 | 描述 |
| --- | --- | --- |
| base value | Undefined、Boolean、String、Number、Object、Environment Record | 基值 |
| referenced name | String | 引用名称 |
| strict reference  | Boolean | 是否是严格引用 |

示例

```javascript
// Unresolvable引用
// 未声明的标识符
console.log(typeof a)

Reference {
  base: undefined,
  name: 'a',
  strict: false
}

// 环境引用
// 声明的标识符
var a = 1
console.log(a)

Reference {
  base: global Environment Record,
  name: 'a',
  strict: false
}

// 属性引用
// Boolean方法
console.log(true.toString)

Reference {
  base: true,
  name: 'toString',
  strict: false
}

// Number方法
console.log((1).toString)

Reference {
  base: 1,
  name: 'toString',
  strict: false
}

// String方法
console.log('a'.toString)

Reference {
  base: 'a',
  name: 'toString',
  strict: false
}

// 对象方法
var obj = {
  name: 'a',
  getName () {
    return this.name
  }
}
console.log(obj.getName)

reference = {
  base: obj,
  name: 'getName',
  strict: false
}
```

### 操作

| 方法 | 描述 |
| --- | --- |
| GetBase(V)  | 获取基值 |
| GetReferencedName(V)  | 获取引用名称 |
| IsStrictReference(V)  | 是否是严格引用 |
| IsUnresolvableReference(V)  | 是否是Unresolvable引用，即基值是否是Undefined |
| IsPropertyReference(V)  | 是否是属性引用，即基值是否是Boolean、String、Number、Object |
| HasPrimitiveBase(V)  | 基值是否是原始值，即Boolean、String、Number |

#### GetValue(V)

获取引用的值

```javascript
/*
 * 非引用类型，直接返回
 * 引用类型
 *     Unresolvable引用，抛出ReferenceError
 *     属性引用
 *         base value为对象，调用对象的[[Get]]方法
 *         base value不为对象，转换为对象，类似于调用对象的[[Get]]方法
 *     环境引用，调用环境记录项的GetBindingValue方法
 */
if (Type(V) !== Reference) {
  return V
}

let base = getBase(V)

if (IsUnresolvableReference(V)) {
  throw ReferenceError
}

if (IsPropertyReference(V)) {
  if (!HasPrimitiveBase(V)) { // base value为对象
    return base.[[Get]](GetReferencedName(V))
  } else { // base value为原始值
    let O = ToObject(base)
    let P = GetReferencedName(V)
    let desc = O.[[GetProperty]](P)

    if (desc === undefined) { // 属性不存在
      return undefined
    } else if (IsDataDescriptor(desc)) { // 数据属性
      return desc.[[Value]]
    } else { // 访问器属性
      let getter = desc.[[Get]]
      if (getter === undefined) {
        return undefined
      } else {
        return getter.[[Call]](base)
      }
    }
  }
}

return base.GetBindingValue(GetReferencedName(V), IsStrictReference(V))
```

示例

```javascript
// 非引用类型
GetValue(null) // null

// Unresolvable引用
GetValue(a) // 抛出ReferenceError

// 环境引用
var a = 1
GetValue(a) // 1

// 属性引用
GetValue(({a: 1}).a) // 1
GetValue(true.a) // undefined
```

#### PutValue(V, W)

设置引用的值

```javascript
/*
 * 非引用类型，抛出ReferenceError
 * 引用类型
 *     Unresolvable引用
 *         严格模式，抛出ReferenceError
 *         非严格模式，调用全局对象的[[Put]]方法
 *     属性引用
 *         base value为对象，调用对象的[[Put]]方法
 *         base value不为对象，转换为对象
 *              访问器属性，调用[[Set]]方法
 *              数据属性或属性不存在，严格模式抛出ReferenceError
 *     环境引用，调用环境记录项的SetMutableBinding方法
 */
if (Type(V) !== Reference) {
  throw ReferenceError
}

let base = getBase(V)

if (IsUnresolvableReference(V)) {
  if (IsStrictReference(V)) { // 严格模式，抛出ReferenceError
    throw ReferenceError
  } else { // 非严格模式，设置全局属性
    global.[[Put]](GetReferencedName(V), W, false)
    return
  }
}

/*
 * 是属性引用，通过[[Put]]方法设置
 */
if (IsPropertyReference(V)) {
  if (!HasPrimitiveBase(V)) { // base value为对象
    base.[[Put]](GetReferencedName(V), W, IsStrictReference(V))
    return
  } else { // base value为原始值
    let O = ToObject(base)
    let P = GetReferencedName(V)
    let Throw = IsStrictReference(V)
    
    if (!O.[[CanPut]](P)) { // 不能设置
      if (Throw) {
        throw ReferenceError
      } else {
        return
      }
    }
    
    let desc = O.[[GetProperty]](P)
    
    if (IsAccessorDescriptor(desc)) { // 访问器属性
      let setter = desc.[[Set]]
      if (setter === undefined) {
        return
      } else {
        setter.[[Call]](base, W)
        return
      }
    } else { // 数据属性或属性不存在
      if (Throw) {
        throw ReferenceError
      } else {
        return
      }
    }
  }
}

/*
 * base value是Environment Record，通过 SetMutableBinding 方法设置
 */
base.SetMutableBinding(GetReferencedName(V), W, IsStrictReference(V))
return
```

示例

```javascript
// 非引用类型
PutValue(null, 2) // ReferenceError

// Unresolvable引用
PutValue(a, 2) // 严格模式，抛出ReferenceError
PutValue(a, 2) // 非严格模式，global.a = 2

// 环境引用
var a = 1
PutValue(a, 2) // a = 2

// 属性引用
PutValue(({a: 1}).a, 2) // {a: 2}
PutValue(true.a, 1) // 严格模式抛出ReferenceError
```

## List类型

列表类型

* 用于处理形参列表、实参列表

```javascript
function foo (a, b) {}
foo(1, 2, 3)

// 实参列表List{1, 2, 3}
// 形参列表List{a, b}
```

## Completion类型

完结类型

* 语句执行都返回一个Completion类型

### 组成

Completion(type, value, target)

| 组成 | 描述 |
| --- | --- |
| type |  类型，normal, break, continue, return, throw 之一 |
| value |  值，任意类型或empty |
| target  |  目标，任一标识符或empty |

示例

```javascript
// normal
1 + 1 // Completion(normal, 2, empty)

// break
for (let i = 0; i < 3; i++) {
  if (i === 2) {
    break // Completion(break, empty, empty)
  }
}

outer: for (let j = 0; j < 3; j++) {
  for (let i = 0; i < 3; i++) {
    if (i === 2) {
      break outer // Completion(break, empty, outer)
    }
  }
}

// continue
for (let i = 0; i < 3; i++) {
  if (i === 2) {
    continue // Completion(continue, empty, empty)
  }
}

outer: for (let j = 0; j < 3; j++) {
  for (let i = 0; i < 3; i++) {
    if (i === 2) {
      continue outer // Completion(continue, empty, outer)
    }
  }
}

// return
function foo () {
  return 1 // Completion(return, 1, empty)
}

// throw
throw new Error('exception') // Completion(throw, error, empty)
```

## Property Descriptor类型

属性描述符类型

* 对属性的描述，用于解释属性特性
* 分为数据属性描述符和访问器属性描述符

```javascript
// 数据属性描述符
{
  [[Value]]: 1,
  [[Writable]]: true,
  [[Enumerable]]: true,
  [[Configurable]]: true
}

// 访问器属性描述符
{
  [[Get]]: function () { return 1 },
  [[Set]]: function () { console.log('set') },
  [[Enumerable]]: true,
  [[Configurable]]: true
}
```

### 操作

#### IsDataDescriptor(Desc)

是否是数据属性描述符

```javascript
if (Desc === undefined) {
  return false
} else if (Desc.[[Value]] === empty && Desc.[[Writable]] === empty) {
  return false
} else {
  return true
}
```

#### IsAccessorDescriptor(Desc)

是否是访问器属性描述符

```javascript
if (Desc === undefined) {
  return false
} else if (Desc.[[Get]] === empty && Desc.[[Set]] === empty) {
  return false
} else {
  return true
}
```

#### IsGenericDescriptor(Desc)

是否是通用属性描述符

```javascript
if (Desc === undefined) {
  return false
} else if (!IsDataDescriptor(Desc) && !IsAccessorDescriptor(Desc)) {
  return true
} else {
  return false
}
```

#### FromPropertyDescriptor(Desc)

将属性描述符转换为对象

```javascript
if (Desc === undefined) {
  return undefined
}

let obj = new Object()

if (IsDataDescriptor(Desc)) { // 数据属性描述符，设置value, writable属性
  obj.[[DefineOwnProperty]]('value', {
    [[Value]]: Desc.[[Value]],
    [[Writable]]: true,
    [[Enumerable]]: true,
    [[Configurable]]: true
  }, false)
  obj.[[DefineOwnProperty]]('writable', {
    [[Value]]: Desc.[[Writable]],
    [[Writable]]: true,
    [[Enumerable]]: true,
    [[Configurable]]: true
  }, false)
} else { // 访问器属性描述符，设置get, set属性
  obj.[[DefineOwnProperty]]('get', {
    [[Value]]: Desc.[[Get]],
    [[Writable]]: true,
    [[Enumerable]]: true,
    [[Configurable]]: true
  }, false)
  obj.[[DefineOwnProperty]]('set', {
    [[Value]]: Desc.[[Set]],
    [[Writable]]: true,
    [[Enumerable]]: true,
    [[Configurable]]: true
  }, false)
}

// 设置enumerable, configurable属性
obj.[[DefineOwnProperty]]('enumerable', {
  [[Value]]: Desc.[[Enumerable]],
  [[Writable]]: true,
  [[Enumerable]]: true,
  [[Configurable]]: true
}, false)
obj.[[DefineOwnProperty]]('configurable', {
  [[Value]]: Desc.[[Configurable]],
  [[Writable]]: true,
  [[Enumerable]]: true,
  [[Configurable]]: true
}, false)

return obj
```

#### ToPropertyDescriptor(Obj)

将对象转换为属性描述符

```javascript
/*
 * Obj不是Object，抛出TypeError
 */
if (Type(Obj) !== Object) {
  throw TypeError
}

let desc = new Property Descriptor()

// 设置[[Enumerable]]
if (Obj.[[HasProperty]]('enumerable')) {
  let enum = Obj.[[Get]]('enumerable')
  desc.[[Enumerable]] = ToBoolean(enum)
}

// 设置[[Configurable]]
if (Obj.[[HasProperty]]('configurable')) {
  let conf = Obj.[[Get]]('configurable')
  desc.[[Configurable]] = ToBoolean(conf)
}

// 设置[[Value]]
if (Obj.[[HasProperty]]('value')) {
  let value = Obj.[[Get]]('value')
  desc.[[Value]] = value
}

// 设置[[Writable]]
if (Obj.[[HasProperty]]('writable')) {
  let writable  = Obj.[[Get]]('writable')
  desc.[[Writable]] = ToBoolean(writable )
}

// 设置[[Get]]
if (Obj.[[HasProperty]]('get')) {
  let getter = Obj.[[Get]]('get')
  if (getter !== undefined && !IsCallable(getter)) {
    throw TypeError
  } else {
    desc.[[Get]] = getter
  }
}

// 设置[[Set]]
if (Obj.[[HasProperty]]('set')) {
  let setter  = Obj.[[Get]]('set')
  if (setter !== undefined && !IsCallable(setter)) {
    throw TypeError
  } else {
    desc.[[Set]] = getter
  }
}

/*
 * 数据属性描述符和访问器属性描述符混合时，抛出TypeError
 */
if ((desc.[[Value]] !== empty || desc.[[Writable]] !== empty) && (desc.[[Get]] !== empty || desc.[[Set]] !== empty)) {
  throw TypeError
}

return desc
```

## Property Identifier类型

属性标识符类型

* 用于关联属性名称与属性描述符，描述属性键值对

### 组成

| 组成 | 类型 | 描述 |
| --- | --- | --- |
| name | String | 属性名 |
| descriptor | Property Descriptor | 属性描述符 |

示例

```javascript
// 数据属性
var o = {
  a: 1
}

PropertyIdentifier {
  name: 'a',
  descriptor: {
    [[Value]]: 1,
    [[Writable]]: true,
    [[Enumerable]]: true,
    [[Configurable]]: true
  }
}

// 访问器属性
var o = {
  get a () { return 1 },
  set a (val) { console.log(val) }
}

PropertyIdentifier {
  name: 'a',
  descriptor: {
    [[Get]]: getter,
    [[Set]]: setter,
    [[Enumerable]]: true,
    [[Configurable]]: true
  }
}
```

## Environment Record类型

环境记录项类型

* 用于记录绑定的标识符情形

### 通用操作

| 方法 | 描述 |
| --- | --- |
| HasBinding(N)  | 是否绑定了指定标识符 |
| CreateMutableBinding(N, D)  | 创建可变绑定，绑定值为undefined，D表示该绑定是否可删除 |
| SetMutableBinding(N, V, S)  | 设置绑定值，S表示是否抛出错误 |
| GetBindingValue(N, S)  | 获取绑定值，S表示是否抛出错误 |
| DeleteBinding(N)  | 删除绑定 |
| ImplicitThisValue()  | 获取绑定的函数对象的this值 |

## Lexical Environment类型

词法环境类型

* 关联环境记录项形成作用域链

### 组成

| 组成 | 类型 | 描述 |
| --- | --- | --- |
| environment record | Environment Record | 环境记录项 |
| outer environment reference | Lexical Environment | 外部词法环境 |

示例

```javascript
// 全局词法环境
Lexical Environment {
  envRec: global Environment Record,
  outer: null
}

// 函数词法环境
Lexical Environment {
  envRec: function Environment Record,
  outer: F.[[Scope]]
}
```
