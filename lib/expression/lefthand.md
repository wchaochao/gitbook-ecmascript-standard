# 左值表达式

标签（空格分隔）： ECMAScript规范

---

```
LeftHandSideExpression : // 左值表达式
  NewExpression // new表达式
  CallExpression // 调用表达式

NewExpression : // new表达式
  MemberExpression // 成员表达式
  new NewExpression // 对象构造

MemberExpression : // 成员表达式
  PrimaryExpression // 主值表达式
  FunctionExpression // 函数表达式
  MemberExpression [ Expression ] // 属性访问
  MemberExpression . IdentifierName // 属性访问
  new MemberExpression Arguments // 对象构造

CallExpression : // 调用表达式
  MemberExpression Arguments // 函数调用
  CallExpression Arguments // 函数调用
  CallExpression [ Expression ] // 属性访问
  CallExpression . IdentifierName // 属性访问

Arguments : // 参数
  ( ) // 空参数
  ( ArgumentList ) // 参数列表

ArgumentList : // 参数列表
  AssignmentExpression // 单个参数
  ArgumentList , AssignmentExpression // 多个参数
```

## 属性访问

```javascript
MemberExpression : MemberExpression [ Expression ]
CallExpression : CallExpression [ Expression ]
MemberExpression . IdentifierName 等价于 MemberExpression [ <identifier-name-string> ]
CallExpression . IdentifierName 等价于 CallExpression [ <identifier-name-string> ]

/*
 * 返回引用
 *     base value为MemberExpression值，要能转换为对象
 *     referenced name为Expression值，转换为字符串
 *     strice为是否是严格模式
 */
let baseValue = GetValue(MemberExpression)
CheckObjectCoercible(baseValue)

let propertyNameValue = GetValue(Expression)
let propertyNameString = ToString(propertyNameValue)

return new Reference({
  base: baseValue,
  name: propertyNameString,
  strict
})
```

示例

```javascript
// 访问未声明变量的属性
console.log(a.toString) // 抛出ReferenceError

// 访问Undefinecd、Null的属性
console.log(null.toString) // 抛出TypeError

// 访问Boolean、Number、String的属性
console.log(true.toString)

reference = {
  base: true,
  name: 'toString',
  strict: false
}

// 访问对象的属性
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

## new构造

### 无参数

```javascript
NewExpression : new NewExpression

/*
 * 调用构造器的[[Construct]]方法创建对象
 *     构造器为NewExpression值，无[[Construct]]方法时抛出TypeError
 *     参数为空
 */
let constructor = getValue(NewExpression)
if (Type(constructor) !== Object || !IsCallable(constructor.[[Construct]])) {
  throw TypeError
}

return constructor.[[Construct]]()
```

示例

```javascript
function Foo () {}
let o = new Foo // Foo.[[Construct]](), 返回{}
```

### 有参数

```javascript
MemberExpression : new MemberExpression Arguments

/*
 * 调用构造器的[[Construct]]方法创建对象
 *     构造器为NewExpression值，无[[Construct]]方法时抛出TypeError
 *     参数为Arguments
 */
let constructor = getValue(NewExpression)
let argList = Arguments
if (Type(constructor) !== Object || !IsCallable(constructor.[[Construct]])) {
  throw TypeError
}

return constructor.[[Construct]](argList)
```

示例

```javascript
function Foo (a) {
  this.a = a
}
let o = new Foo(1) // Foo.[[Construct]](List{1}), 返回{a: 1}
```

## 函数调用

```javascript
CallExpression : MemberExpression Arguments
CallExpression : CallExpression Arguments

/*
 * 调用函数的[[Call]]方法
 *     函数为MemberExpression值，无[[Call]]方法时抛出TypeError
 *     参数为Arguments
 *     MemberExpression不为引用时，this值为undefined
 *     MemberExpression为引用
 *         属性引用，this值为引用的base值
 *         环境引用，this值为环境的this值
 */
let func = getValue(MemberExpression)
let argList = Arguments
if (func !== Object || !IsCallable(func.[[Call]])) {
  throw TypeError
}

let thisValue
if (Type(ref) === Reference) {
  if (IsPropertyReference(ref)) {
    thisValue = GetBase(ref)
  } else {
    thisValue = GetBase(ref).ImplicitThisValue()
  }
} else {
  thisValue = undefined
}

return func.[[Call]](thisValue, argList)
```

示例

```javascript
// 调用普通函数
function foo () {}
foo()

foo: Reference {
  base: global environment record,
  name: 'foo',
  strict: false
}
thisArg: undefined

// 调用Boolean、Number、String方法
console.log(true.toString())

true.toString: Reference {
  base: true,
  name: 'toString',
  strict: false
}
thisArg: true

// 调用对象方法
let obj = {
  name: 'a',
  getName () {
    return this.name
  }
}
obj.getName()

obj.getName: Reference {
  base: obj,
  name: 'getName',
  strict: false
}
thisArg: obj
```

## 参数

### 空参数

```javascript
Arguments : ( )

/*
 * 创建一个列表并返回
 */
return new List()
```

示例

```javascript
function foo (a, b, c) {}
foo() // List{}
```

### 参数列表

```javascript
Arguments : ( ArgumentList )

ArgumentList : AssignmentExpression
ArgumentList : ArgumentList , AssignmentExpression
```

#### 单个参数

```javascript
ArgumentList : AssignmentExpression

/*
 * 创建一个列表
 *     元素为AssignmentExpression值
 */
let arg = GetValue(AssignmentExpression)
return new List(arg)
```

示例

```javascript
function foo (a, b, c) {}
foo(1) // List{1}
```

多个参数

```javascript
ArgumentList : ArgumentList , AssignmentExpression

/*
 * 根据参数列表创建一个列表
 *     添加一个元素，值为AssignmentExpression值
 */
let precedingArgs = Arguments : ( ArgumentList )
let arg = GetValue(AssignmentExpression)
precedingArgs.add(arg)
return precedingArgs
```

示例

```javascript
function foo (a, b, c) {}
foo(1, 2) // List{1, 2}
```

## 函数表达式

```javascript
MemberExpression : FunctionExpression

/*
 * 创建函数并返回
 */
return FunctionExpression
```

示例

```javascript
let fn = function foo () {}
```
