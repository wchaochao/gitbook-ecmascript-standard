# 主值表达式

标签（空格分隔）： ECMAScript规范

---

```
PrimaryExpression :
  this // this
  Identifier // 标识符
  Literal // 字面量
  ArrayLiteral // 数组字面量
  ObjectLiteral // 对象字面量
  ( Expression ) // 分组表达式
```

## this

```javascript
PrimaryExpression : this

/*
 * 返回当前上下文的ThisBinding
 *     全局上下文，this为全局对象
 *     函数上下文
 *         严格模式，this为函数的thisArg
 *             单纯调用函数，thisArg为undefined
 *             调用对象方法，thisArg为该对象
 *             new构造，thisArg为实例对象
 *             调用call()、apply()、bind()，thisArg为绑定的this值
 *         非严格模式，thisArg转换为对象，不能转换时为全局对象
 *     eval上下文
 *         间接调用，this为全局对象
 *         直接调用，this为调用上下文的ThisBinding
 */
return runningContext.ThisBinding
```

示例

```javascript
// 全局this
console.log(this) // window对象

// 函数this
// 单纯调用函数
function func () {
  'use strict';
  return this
}
console.log(func()) // undefined

// 调用对象方法
var obj = {
  name: 'a',
  getName () {
    return this.name
  }
}
console.log(obj.getName()) // 'a'

// new构造
function Rect (width, height) {
  this.width = width
  this.heigth = height
}
console.log(new Rect(10, 20)) // {width: 10, height: 20}

// call调用
function func () {
  return Array.prototype.slice.call(arguments)
}
console.log(func(1, 2, 3)) // [1, 2, 3]
```

## 标识符

```javascript
PrimaryExpression : Identifier

/*
 * 返回一个引用
 *     引用的base value为绑定标识符的环境记录项，没有为undefined
 *     reference name为标识符字符串
 *     strict为是否为严格模式
 */
let env = runningContext.LexicalEnvironment
return GetIdentifierReference(env, Identifier, strict)
```

示例

```javascript
// 未声明的标识符
console.log(typeof a)

Reference {
  base: undefined,
  name: 'a',
  strict: false
}

// 全局上下文的标识符
var a = 1
console.log(a)

Reference {
  base: global Environment Record,
  name: 'a',
  strict: false
}

// 函数上下文的标识符
function foo (a) {
  console.log(a)
  
  Reference {
    base: function Environment Record,
    name: 'a',
    strict: false
  }
}
```

## 字面量

```
PrimaryExpression : Literal

Literal ::
  NullLiteral
  BooleanLiteral
  NumericLiteral
  StringLiteral
  RegularExpressionLiteral
```

示例

```javascript
// Null字面量
null

// 布尔字面量
true
false

// 数字字面量
10
0x12ab
10.12
1.23e+3

// 字符串字面量
'abc'
'\n'

// 正则字面量
/a/i
```

## 数组字面量

```
PrimaryExpression : ArrayLiteral

ArrayLiteral : // 数组字面量
  [ Elision(opt) ] // 全空位
  [ ElementList ] // 元素列表
  [ ElementList , Elision(opt) ] // 元素列表 + 后置空位

ElementList : // 元素列表
  Elision(opt) AssignmentExpression // 一个 前置空位 + 元素
  ElementList , Elision(opt) AssignmentExpression // 多个 前置空位 + 元素

Elision : // 空位
  , // 一个逗号
  Elision , // 多个逗号
```

### 全空位

```javascript
ArrayLiteral : [ Elision(opt) ]

/*
 * 创建一个数组
 *    length属性为空位个数
 */
let array = new Array()
let pad = Elision.size || 0
array.[[Put]]('length', pad, false)
return array
```

示例

```javascript
let arr = [,,,]
console.log(arr) // [empty × 3]
```

### 元素列表

```javascript
ArrayLiteral : [ ElementList ]

ElementList : Elision(opt) AssignmentExpression
ElementList : ElementList , Elision(opt) AssignmentExpression
```

#### 一个 前置空位 + 元素

```javascript
ElementList : Elision(opt) AssignmentExpression

/*
 * 创建一个数组
 *     设置数组第一个不为空位的元素值，索引为空位个数，值为AssignmentExpression值
 */
let array = new Array()
let firstIndex = Elision.size || 0
let initValue = GetValue(AssignmentExpression)
array.[[DefineOwnProperty]](ToString(firstIndex), {
   [[Value]]: initValue,
   [[Writable]]: true,
   [[Enumerable]]: true,
   [[Configurable]]: true
}, false)
return array
```

示例

```javascript
let arr = [,,1]
console.log(arr) // [empty × 2, 1]
```

#### 多个 前置空位 + 元素

```javascript
ElementList : ElementList , Elision(opt) AssignmentExpression

/*
 * 根据元素列表创建一个数组
 *     设置数组最后一个不为空位的元素值，索引为ToUint32(数组长度 + 空位个数)，值为AssignmentExpression值
 */
let array = ArrayLiteral : [ ElementList ]
let len = array.[[Get]]('length')
let pad = Elision.size || 0
let firstIndex = ToUint32(len + pad)
let initValue = GetValue(AssignmentExpression)
array.[[DefineOwnProperty]](ToString(firstIndex), {
   [[Value]]: initValue,
   [[Writable]]: true,
   [[Enumerable]]: true,
   [[Configurable]]: true
}, false)
return array
```

示例

```javascript
let arr = [,,1,,2]
console.log(arr) // [empty × 2, 1, empty, 2]
```

### 元素列表 + 后置空位

```javascript
ArrayLiteral : [ ElementList , Elision(opt) ]

/*
 * 根据元素列表创建一个数组
 *    设置该数组的length属性为ToUint32(数组长度 + 空位个数)
 */
let array = ArrayLiteral : [ ElementList ]
let len = array.[[Get]]('length')
let pad = Elision.size || 0
array.[[Put]]('length', ToUnit32(len + pad), false)
```

示例

```javascript
let arr = [,,1,,2,,]
console.log(arr) // [empty × 2, 1, empty, 2, empty]
```

## 对象字面量

```
PrimaryExpression : ObjectLiteral

ObjectLiteral : // 对象字面量
  { } // 空对象
  { PropertyNameAndValueList } // 属性列表
  { PropertyNameAndValueList , } // 属性列表 + ,

PropertyNameAndValueList : // 属性列表
  PropertyAssignment // 单个属性
  PropertyNameAndValueList , PropertyAssignment // 多个属性

PropertyAssignment : // 单个属性
  PropertyName : AssignmentExpression // 键值对
  get PropertyName ( ) { FunctionBody } // get方法
  set PropertyName ( PropertySetParameterList ) { FunctionBody } // set方法

PropertyName : // 属性名
  IdentifierName // 标识符名
  StringLiteral // 字符串字面量
  NumericLiteral // 数字字面量

PropertySetParameterList : // set方法参数列表
  Identifier // 标识符
```

### 空对象

```javascript
ObjectLiteral : { }

/*
 * 创建一个对象并返回
 */
return new Object()
```

示例

```javascript
let o = {}
```

### 元素列表

```javascript
ObjectLiteral : { PropertyNameAndValueList }
ObjectLiteral : { PropertyNameAndValueList , }

PropertyNameAndValueList : PropertyAssignment
PropertyNameAndValueList : PropertyNameAndValueList , PropertyAssignment
```

#### 单个属性

```javascript
PropertyNameAndValueList : PropertyAssignment

/*
 * 创建一个对象
 *     设置该对象的属性为指定属性
 */
let obj = new Object()
let prop = ToPropertyIdentifier(PropertyAssignment)
obj.[[DefineOwnProperty]](prop.name, prop.descriptor, false)
return obj
```

示例

```javascript
// 数据属性
let o = {
  a: 1
}

// 访问器属性
let o = {
  get p () { return 1 },
  set p (val) { console.log(val) }
}
```

#### 多个属性

```javascript
/*
 * 根据属性列表创建一个对象
 *     指定属性可以设置，设置属性
 *     指定属性不可设置，抛出SyntaxError
 *         属性重复且满足下列条件时不可设置
 *            属性的描述符发生改变
 *            属性同为数据属性，且在严格模式
 *            属性同为访问器属性，且[[Get]]或[[Set]]发生变化
 */
PropertyNameAndValueList : PropertyNameAndValueList , PropertyAssignment

let obj = ObjectLiteral : { PropertyNameAndValueList }
let prop = ToPropertyIdentifier(PropertyAssignment)
let previous = obj.[[GetOwnProperty]](prop.name)

if (previous !== undefined) {
  let invalid = false
  let desc = prop.descriptor
  if (IsDataDescriptor(previous)) {
    if (IsDataDescriptor(desc)) {
      if (strict) {
        invalid = true
      }
    } else {
      invalid = true
    }
  } else {
    if (IsDataDescriptor(desc)) {
      invalid = true
    } else {
      if ((previous.[[Get]] && desc.[[Get]]) || (previous.[[Set]] && desc.[[Set]])) {
        invalid = true
      }
    }
  }
  if (invalid) {
    throw SyntaxError
  }
}

obj.[[DefineOwnProperty]](prop.name, prop.descriptor, false)
return obj
```

示例

```javascript
// 属性不重复
let o = {
  a: 1,
  get p () { return 1 },
  set p (val) { console.log(val) }
}

// 数据属性重复，严格模式下抛出Syntax错误
let o = {
  a: 1,
  b: 2,
  a: 2
}

// 访问器属性重复，抛出Syntax错误
let o = {
  a: 1,
  get p () { return 1 },
  get p () { return 2 }
}

// 属性描述符类型发送改变，抛出Syntax错误
let o = {
  a: 1,
  get a () { return 1 }
}
```

### 属性解析

#### 键值对

```javascript
PropertyAssignment : PropertyName : AssignmentExpression

/*
 * 创建属性标识符
 *     属性名为键名
 *     属性描述符为数据属性描述符，[[Value]]为键值
 */
let propName = ToString(PropertyName)
let propValue = GetValue(AssignmentExpression)
let desc = {
  [[Value]]: propValue,
  [[Writable]]: true,
  [[Enumerable]]: true,
  [[Configurable]]: true
}
return new Property Identifier(propName, desc)
```

示例

```javascript
let o = {
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
```

#### get方法

```javascript
PropertyAssignment : get PropertyName ( ) { FunctionBody }

/*
 * 创建属性标识符
 *     属性名为键名
 *     属性描述符为访问器属性描述符，[[Get]]为封装后的get方法
 */
let propName = ToString(PropertyName)
let closure = createFunction(new List(), get.FunctionBody, currentContext.LexicalEnvironment, strict)
let desc = {
  [[Get]]: closure,
  [[Enumerable]]: true,
  [[Configurable]]: true
}
return new Property Identifier(propName, desc)
```

示例

```javascript
let o = {
  get p () { return 1 }
}

PropertyIdentifier {
  name: 'p',
  descriptor: {
    [[Get]]: getter,
    [[Set]]: undefined,
    [[Enumerable]]: true,
    [[Configurable]]: true
  }
}
```

#### set方法

```javascript
PropertyAssignment : set PropertyName ( PropertySetParameterList ) { FunctionBody }

/*
 * 创建属性标识符
 *     属性名为键名
 *     属性描述符为访问器属性描述符，[[Set]]为封装后的set方法
 */
let propName = ToString(PropertyName)
let closure = createFunction(PropertySetParameterList , set.FunctionBody, currentContext.LexicalEnvironment, strict)
let desc = {
  [[Set]]: closure,
  [[Enumerable]]: true,
  [[Configurable]]: true
}
return new Property Identifier(propName, desc)
```

示例

```javascript
let o = {
  set p (val) { console.log(val) }
}

PropertyIdentifier {
  name: 'p',
  descriptor: {
    [[Get]]: undefined,
    [[Set]]: setter,
    [[Enumerable]]: true,
    [[Configurable]]: true
  }
}
```

## 分组表达式

```javascript
PrimaryExpression : ( Expression )

/*
 * 优先运算Expression
 */
return Expression
```

示例

```javascript
(1 + 1) * 2 // 2 * 2

1 + 1 * 2 // 1 + 2
```
