# 其他语句

标签（空格分隔）： ECMAScript规范

---

## with语句

```
WithStatement :
  with ( Expression ) Statement
```

### 算法

```javascript
WithStatement : with ( Expression ) Statement

/**
 * 获取Expression值并转换为对象
 * 基于当前上下文创建一个对象式词法环境，绑定上述对象，provideThis设为true
 * 在对象式词法环境中执行Statement，执行完后回归以前的词法环境
 */

let obj = ToObject(GetValue(Expression))
let oldEnv = currentContext.LexicalEnvironment
let newEnv = new NewObjectEnvironment(obj, oldEnv)
newEnv.provideThis = true
currentContext.LexicalEnvironment = newEnv
let s = Statement
currentContext.LexicalEnvironment = oldEnv
return s
```

示例

```javascript
let o = {
  a: 1
}
with (o) {
  a = 2 // o.a = 2
}
```

### 严格模式限制

```javascript
/**
 * 严格模式，直接抛出SyntaxError
 */
 
if (strict) {
  throw SyntaxError
}
```

## try语句

```
TryStatement :
  try Block Catch // try/catch
  try Block Finally // try/finally
  try Block Catch Finally // try/catch/finally

Catch :
  catch ( Identifier ) Block // catch语句

Finally :
  finally Block // finally语句
```

### try/catch

```javascript
TryStatement : try Block Catch

/**
 * 执行Block，结果为B
 * B.type不是throw，直接返回B
 * B.type是throw，将B.value传给Catch，获取返回值
 */
let B = Block
if (B.type !== throw) {
  return B
} else {
  return exec(Catch, B.value)
}
```

示例

```javascript
// 不是throw
try {
  let a = 1
} catch (e) {
  console.log(e) // 不执行
}

// 是throw
try {
  throw 1
} catch (e) {
  console.log(e) // e = 1
}
```

### catch

传入参数C

```javascript
Catch : catch ( Identifier ) Block

/**
 * 基于当前上下文创建一个声明式词法环境，创建可变绑定Identifier，值为传入的参数C
 * 在新建的声明式词法环境中执行Statement，执行完后回归以前的词法环境
 */
let oldEnv = currentContext.LexicalEnvironment
let catchEnv = new NewDeclarativeEnvironment(oldEnv)
let envRec = catchEnv.environmentRecord
envRec.CreateMutableBinding(Identifier)
envRec.SetMutableBinding(Identifier, C, false)

currentContext.LexicalEnvironment = catchEnv
let s = Statement
currentContext.LexicalEnvironment = oldEnv
return s
```

严格模式限制

```javascript
/**
 * 严格模式下，Identifier为eval或argument，抛出SyntaxError
 */
if (strict) {
  if (<identifier-name-string> === 'eval' || <identifier-name-string> === 'argument') {
    throw SyntaxError
  }
}
```

### try/finally

```javascript
TryStatement : try Block Finally

/**
 * 执行Block，结果为B
 * 执行Finally，结果为F
 * F.type是normal，返回B
 * F.type不是normal，返回F
 */
let B = Block
let F = Finally
if (F.type === normal) {
  return B
} else {
  return F
}
```

示例

```javascript
// F.type为normal
function foo () {
  try {
    return 1
  } finally {
    let a = 1
  }
}

foo() // 1

// F.type不为normal
function foo () {
  try {
    return 1
  } finally {
    return 2
  }
}

foo() // 2
```

### finally

```javascript
Finally : finally Block

/**
 * 执行Block，返回结果
 */
let F = Block
return F
```

### try/catch/finally

```javascript
 TryStatement : try Block Catch Finally
 
/**
 * 先执行try/catch，再执行try/finally
 */
 let B = Block
 let C
 if (B.type !== throw) {
   C = B
 } else {
   C = exec(Catch, B.value)
 }
 
 let F = Finally
 if (F.type === normal) {
   return C
 } else {
   return F
 }
```

示例

```javascript
try {
  let a = 1
} catch (e) {
  console.log(e) // 不执行
} finally {
  let b = 2
}

try {
  throw 1
} catch (e) {
  console.log(e) // e = 1
  throw 2 // 不抛出
} finally {
  throw 3 // 抛出
}
```
