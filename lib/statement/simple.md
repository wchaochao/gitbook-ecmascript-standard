# 简单语句

标签（空格分隔）： ECMAScript规范

---

## 语句

执行并返回一个Completion类型

```
Statement : // 语句
  EmptyStatement // 空语句
  Block // 块语句
  VariableStatement // 变量语句
  ExpressionStatement // 表达式语句
  LabelledStatement // label语句
  ContinueStatement // continue语句
  BreakStatement // break语句
  ReturnStatement // return语句
  ThrowStatement // throw语句
  DebuggerStatement // debugger语句
  IfStatement // if语句
  SwitchStatement // switch语句
  IterationStatement // 循环语句
  WithStatement // with语句
  TryStatement // try语句
```

## 空语句

```javascript
EmptyStatement :
  ;
```

### 算法

```javascript
EmptyStatement : ;

/**
 * 返回Completion(normal, empty, empty)
 */
return Completion(normal, empty, empty)
```

## 块语句

```
Block : // 块语句
  { StatementList(opt) } // { 语句列表 }
  
StatementList : // 语句列表
  Statement // 单语句
  StatementList Statement // 多语句
```

### 语句列表

```javascript
Block : { StatementList(opt) }

/**
 * 无StatementList，返回Completion(normal, empty, empty)
 * 有StatementList，执行StatementList并返回结果
 */
if (!IsPresent(StatementList)) {
  return Completion(normal, empty, empty)
} else {
  let s = StatementList
  return s
}
```

示例

```javascript
if (true) {
} // Completion(normal, empty, empty)
```

### 单语句

```javascript
StatementList : Statement

/**
 * 执行Statement
 * 有异常抛出，返回Completion(throw, exception, empty)
 * 无异常抛出，返回Statement结果
 */
let s = Statement
if (exception) {
  return Completion(throw, exception, empty)
} else {
  return s
}
```

示例

```javascript
delete o.a // Completion(throw, ReferenceError, empty)

let o = {}
delete o.a // Completion(normal, true, empty)
```

### 多语句

```javascript
StatementList : StatementList Statement

/**
 * 执行StatementList
 * 结果为abrupt completion, 返回StatementList结果
 * 结果不为abrupt completion，执行Statement，返回Completion(s.type, s.value || sl.value, s.target)
 */
let sl = StatementList
if (sl.type !== normal) {
  return sl
}

let s = Statement
let V = s.value === empty ? sl.value : s.value
return Completion(s.type, V, s.target)
```

示例

```javascript
{
  delete o.a // Completion(throw, ReferenceError, empty)
  let a = 1 // 不执行
}

{
  let a = 1
  delete o.a // Completion(throw, ReferenceError, empty)
}

{
  let a = 1
  a++ // Completion(normal, 1, empty)
}

{1;;;;;} // Completion(normal, 1, empty)
```

## 变量语句

变量声明

```
VariableStatement : // 变量语句
  var VariableDeclarationList ; // var 变量声明列表;

VariableDeclarationList : // 变量声明列表
  VariableDeclaration // 单个变量声明
  VariableDeclarationList , VariableDeclaration // 多个变量声明

VariableDeclaration : // 单个变量声明
  Identifier Initialiser(opt) // 标识符 变量初始化

Initialiser : // 变量初始化
  = AssignmentExpression // = 赋值语句
```

### 变量声明列表

```javascript
VariableStatement : var VariableDeclarationList

/**
 * 执行VariableDeclarationList，返回Completion(normal, empty, empty)
 */
exec(VariableDeclarationList)
return Completion(normal, empty, empty)
```

示例

```javascript
var a
var b = 1

var c,
    d = 2
```

### 单个变量声明

```javascript
VariableDeclarationList : VariableDeclaration
```

#### 声明

```javascript
VariableDeclaration : Identifier

/**
 * 返回标识符字符串
 */
return <identifier-name-string>
```

示例

```javascript
var a
```

#### 初始化

```javascript
VariableDeclaration : Identifier Initialiser

/**
 * 对标识符进行赋值并返回标识符字符串
 */
let lhs = Identifier
let rightValue = GetValue(Initialiser)
PutValue(lhs, rightValue)
return <identifier-name-string>
```

示例

```javascript
var a = 1
```

### 多个变量声明

```javascript
VariableDeclarationList : VariableDeclarationList , VariableDeclaration

/**
 * 执行VariableDeclarationList
 * 执行VariableDeclaration并返回结果
 */
exec(VariableDeclarationList)
return exec(VariableDeclaration)
```

示例

```javascript
var c,
    d = 2
```

### 严格模式限制

```javascript
/**
 * 严格模式下，标识符为eval或arguments时抛出SyntaxError
 */
if (strict) {
  if (<identifier-name-string> === 'eval' || <identifier-name-string> === 'arguments') {
    throw SyntaxError
  }
}
```

示例

```javascript
'use strict'
let eval = 1 // 抛出SyntaxError
```

## 表达式语句

执行运算

```
ExpressionStatement :
  [lookahead ∉ {{, function}] Expression ; // 表达式前不为{或function
```

### 算法

```javascript
ExpressionStatement : [lookahead ∉ {{, function}] Expression ;

/**
 * 获取Expression的值，返回Completion(normal, V, empty)
 */
let V = GetValue(Expression)
return Completion(normal, V, empty)
```

示例

```javascript
1; // Completion(normal, 1, empty)

let a
a = 1 // Completion(normal, 1, empty)

let o = {}
o.a = 1 // Completion(normal, 1, empty)
delete o.a // Completion(normal, true, empty)

let foo = function () {}
foo() // Completion(normal, undefined, empty)
```
