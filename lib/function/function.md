# 函数

标签（空格分隔）： ECMAScript规范

---

## 定义

```
FunctionDeclaration : // 函数声明
  function Identifier ( FormalParameterList(opt) ) { FunctionBody }

FunctionExpression : // 函数表达式
  function Identifier(opt) ( FormalParameterList(opt) ) { FunctionBody }

FormalParameterList : // 形参列表
  Identifier // 单参数
  FormalParameterList , Identifier // 多参数

FunctionBody : // 函数体
  SourceElements(opt) // 源代码
```

### 函数声明

```javascript
FunctionDeclaration : function Identifier ( FormalParameterList(opt) ) { FunctionBody }

/**
 * 创建函数对象，形参为FormalParameterList，函数体为FunctionBody，作用域为currentContext.VariableEnvironment，严格模式为strict
 * 将创建的函数对象绑定到Identifier上
 */
let fun = createFunction(FormalParameterList, FunctionBody, currentContext. VariableEnvironment, strict)
bind(Identifier, func)
```

示例

```javascript
function foo () {}
```

### 函数表达式

#### 匿名函数

```javascript
FunctionExpression : function ( FormalParameterList(opt) ) { FunctionBody }

/**
 * 创建函数对象，形参为FormalParameterList，函数体为FunctionBody，作用域为currentContext.LexicalEnvironment ，严格模式为strict
 * 返回该函数对象
 */
let fun = createFunction(FormalParameterList, FunctionBody, currentContext. LexicalEnvironment, strict)
return func
```

示例

```javascript
let foo = function () {}
```

#### 命名函数

```javascript
FunctionExpression : function Identifier ( FormalParameterList(opt) ) { FunctionBody }

/**
 * 基于当前上下文创建一个声明式词法环境funEnv
 * 创建函数对象，形参为FormalParameterList，函数体为FunctionBody，作用域为funEnv ，严格模式为strict
 * funEnv创建不可变绑定Identifier，值为创建的函数对象
 * 返回创建的函数对象
 */
let funEnv = new NewDeclarativeEnvironment(currentContext.LexicalEnvironment)
let closure = createFunction(FormalParameterList, FunctionBody, funEnv, strict)

let envRec = funEnv.environmentRecord
envRec.CreateImmutableBinding(Identifier)
envRec.InitializeImmutableBinding(Identifier, closure)

return closure
```

示例

```javascript
let foo = function foo () {}
```

### 函数体

```javascript
FunctionBody : SourceElements(opt)

/**
 * 包含在严格模式代码中或开头有严格模式指令，则为严格模式
 * 执行SourceElements，结果为s
 *     s.type为return或throw，返回s
 *     s.type为其他，返回Completion(normal, undefined, empty)
 */
if (IsContainedInStrictModeCode || beginWithUserStrictDirective) {
  let strict = true
}
let s = SourceElements
if (s.type === return || s.type === throw) {
  return s
} else {
  return Completion(normal, undefined, empty)
}
```

示例

```javascript
// 严格模式
function foo () {
  'use strict'
}

// return
function foo () {
  return 1
}

// throw
function foo () {
  throw 1
}

// normal
function foo () {}
```

### 严格模式限制

```javascript
/**
 * 严格模式下
 * 形参列表中有eval或arguments，抛出SyntaxError
 * 形参列表有重复的形参，抛出SyntaxError
 */
if (strict) {
  if (FormalParameterList.has('eval') || FormalParameterList.has('arguments')) {
    throw SyntaxError
  }
  
  if (FormalParameterList.hasRepeatIndentifier()) {
    throw SyntaxError
  }
}
```

示例

```javascript
// 参数为eval
function foo (eval) {
  'use strict'
}

// 重复参数
function foo (a, a) {
  'use strict'
}
```

## 创建

```javascript
createFunction(FormalParameterList, FunctionBody, Scope, Strict)
```

### 属性

```javascript
/**
 * 创建函数对象
 * [[Class]]属性为'Function'
 * [[Prototype]]属性为Function.prototype
 * [[Extensible]]属性为true
 * [[FormalParameters]]属性为FormalParameterList
 * [[Code]]属性为FunctionBody
 * [[Scope]]属性为Scope
 * length属性为形参个数
 * prototype属性为原型对象，有constructor属性指向F
 * 严格模式下，访问caller、arguments属性抛出TypeError
 */
let F = new Object()
F.[[Class]] = 'Function'
F.[[Prototype]] = Function.prototype
F.[[Extensible]] = true
F.[[FormalParameters]] = FormalParameterList
F.[[Code]] = FunctionBody
F.[[Scope]] = Scope

let len = FormalParameterList.length
F.[[DefineOwnProperty]]('length', {
  [[Value]]: len,
  [[Writable]]: false,
  [[Enumerable]]: false,
  [[Configurable]]: false
}, false)

let proto = new Object()
proto.[[DefineOwnProperty]]('constructor', {
  [[Value]]: F,
  [[Writable]]: true,
  [[Enumerable]]: true,
  [[Configurable]]: true
}, false)
F.[[DefineOwnProperty]]('prototype', {
  [[Value]]: proto,
  [[Writable]]: true,
  [[Enumerable]]: false,
  [[Configurable]]: false
}, false)

if (strict) {
  let thrower = [[ThrowTypeError]]
  F.[[DefineOwnProperty]]('caller', {
    [[Get]]: thrower,
    [[Set]]: thrower,
    [[Enumerable]]: false,
    [[Configurable]]: false
  }, false)
  F.[[DefineOwnProperty]]('arguments', {
    [[Get]]: thrower,
    [[Set]]: thrower,
    [[Enumerable]]: false,
    [[Configurable]]: false
  }, false)
}

return F
```

示例

```javascript
function foo (a, b) {
  'use strict'
  console.log(a, b)
}

// foo对象
{
  [[Class]]: 'Function',
  [[Prototype]]: Function.prototype,
  [[Extensible]]: true,
  [[FormalParameters]]: List{a, b}
  [[Code]]: FunctionBody,
  [[Scope]]: Global Environment,
  length: 2,
  prototype: {
    constructor: foo
  },
  caller: thrower,
  arguments: thrower
}
```

### 方法

#### F.[[Call]] (thisArg, argList)

执行函数

```javascript
/**
 * 根据thisArg、F.[[Scope]]创建函数上下文，压入上下文堆栈并初始化
 * 执行F.[[Code]]，退出函数上下文
 *     s.type为throw，抛出s.value
 *     s.type为return，返回s.value
 *     s.type为normal，返回undefined
 */
let funcCtx = createFunctionContext(thisArg, F.[[Scope]])
let s = exec(F.[[Code]])
funcCtx.exit()

if (s.type === throw) {
  throw s.value
} else if (s.type === return) {
  return s.value
} else if (s.type === normal) {
  return undefined
}
```

示例

```javascript
// throw
functon foo () {
  throw 1
}
foo()

// return
functon bat () {
  return 1
}
bat()

// normal
functon cat () {
  let a = 1
  console.log(a)
}
cat()
```

#### F.[[Construct]] (argList)

创建实例对象

```javascript
/**
 * 创建实例对象
 * [[Class]]属性为'Object'
 * [[Extensible]]属性为true
 * 获取F的prototype属性
 *     F的prototype属性是对象类型，[[Prototype]]属性为F的prototype属性
 *     F的prototype属性不是对象类型，[[Prototype]]属性为Object.prototype
 * 调用F的[[Call]]方法，thisArg为obj，参数列表为argList，结果为result
 *     result是Object类型，返回result
 *     result不是Object类型，返回obj
 */
let obj = new Object()
obj.[[Class]] = 'Object'
obj.[[Extensible]] = true

let proto = F.[[Get]]('prototype')
if (Type(proto) === Object) {
  obj.[[Prototype]] = proto
} else {
  obj.[[Prototype]] = Object.prototype
}

let result = F.[[Call]](obj, argList)
if (Type(result) === Object) {
  return result
} else {
  return obj
}
```

示例

```javascript
functon Rect (width, height) {
  this.width = width
  this.height = height
}
let r = new Rect(20, 10)

// 实例对象
{
  [[Class]]: 'Object',
  [[Extensible]]: true,
  [[Prototype]]: Rect.[[Get]]('prototype'),
  width: 20,
  height: 10
}
```

#### F.[[HasInstance]] (V)

判断V是否是F的实例对象

```javascript
/**
 * V不为对象，返回false
 * 获取F的prototype属性O，不为对象时，抛出TypeError
 * 遍历V的原型链
 *     O在V的原型链上，返回true
 *     O不在V的原型链上，返回false
 */
if (Type(V) !== Object) {
  return false
}

let O = F.[[Get]]('prototype')
if (Type(O) !== Object) {
  throw TypeError
}

while (true) {
  V = V.[[prototype]]
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
1 instanceof Number // false
let a = new Number(1)
a instanceof Number // true
a instanceof Object // true
```

## 内部函数

### [[ThrowTypeError]]

抛出TypeError函数

```javascript
let F = createFunction(List{}, 'throw TypeError', GlobalEnvironment, true)
F.[[Extensible]] = false
return F
```
