# 中断语句

标签（空格分隔）： ECMAScript规范

---

## label语句

标签语句，用于continue label和break label

* 语句默认有个空的标签组
* 循环语句和switch语句默认有个包含单个元素 empty 的标签组

```
LabelledStatement :
  Identifier : Statement
```

### 算法

```javascript
Identifier : Statement

/**
 * Statement添加标签并执行，返回Statement结果
 */
Statement.labels.add(Identifier)
let s = Statement
return s
```

示例

```javascript
outer: for (let j = 0; j < 3; j++) {
  let count = 0
  for (let i = 0; i < 3; i++) {
    count++
    if (count === 1) {
      break outer
    }
  }
}
```

## continue语句

进入下一次循环

* 出现在循环语句中
* Identifier在循环语句的标签组中

```
ContinueStatement :
  continue ;
  continue [no LineTerminator here] Identifier ;
```

### continue

```javascript
ContinueStatement : continue ;

/**
 * 返回Completion(continue, empty, empty)
 */
return Completion(continue, empty, empty)
```

示例

```javascript
let count = 0
for (let i = 0; i < 3; i++) {
  count++
  if (count === 1) {
    continue
  }
}
```

### continue label

```javascript
ContinueStatement : continue [no LineTerminator here] Identifier ;

/**
 * 返回Completion(continue, empty, Identifier)
 */
return Completion(continue, empty, Identifier)
```

示例

```javascript
outer: for (let j = 0; j < 3; j++) {
  let count = 0
  for (let i = 0; i < 3; i++) {
    count++
    if (count === 1) {
      continue outer
    }
  }
}
```

## break语句

跳出循环或switch语句

* 出现在循环语句或switch语句中
* Identifier在语句的标签组中

```
BreakStatement :
  break ;
  break [no LineTerminator here] Identifier ;
```

### break

```javascript
BreakStatement : break ;

/**
 * 返回Completion(break, empty, empty)
 */
return Completion(break, empty, empty)
```

示例

```javascript
let count = 0
for (let i = 0; i < 3; i++) {
  count++
  if (count === 1) {
    break
  }
}
```

### break label

```javascript
BreakStatement : break [no LineTerminator here] Identifier ;

/**
 * 返回Completion(break, empty, Identifier)
 */
return Completion(break, empty, Identifier)
```

示例

```javascript
outer: for (let j = 0; j < 3; j++) {
  let count = 0
  for (let i = 0; i < 3; i++) {
    count++
    if (count === 1) {
      break outer
    }
  }
}
```

## return语句

返回值

* 出现在函数中
* 停止函数执行并把值返回给caller

```
ReturnStatement :
  return ;
  return [no LineTerminator here] Expression ;
```

### return

```javascript
ReturnStatement : return ;

/**
 * 返回Completion(return, undefined, empty)
 */
return Completion(return, undefined, empty)
```

示例

```javascript
function foo () {
  return
}

let a = foo() // undefined
```

### return value

```javascript
ReturnStatement : return [no LineTerminator here] Expression ;

/**
 * 返回Completion(return, V, empty)
 */
let V = GetValue(Expression)
return Completion(return, V, empty)
```

示例

```javascript
function foo () {
  return 1
}

let a = foo() // 1
```

## throw语句

抛出错误

```
ThrowStatement :
  throw [no LineTerminator here] Expression ;
```

### 算法

```javascript
ThrowStatement : throw [no LineTerminator here] Expression ;

/**
 * 获取Expression的值，并返回Completion(throw, V, empty)
 */
let V = GetValue(Expression)
return Completion(throw, V, empty)
```

示例

```javascript
throw 1
throw new Error('error')
```

## debugger语句

```javascript
DebuggerStatement :
  debugger ;
```

### 算法

```javascript
DebuggerStatement : debugger ;

/**
 * 支持debugger, 返回实现的debuggerCompletion
 * 不支持debugger, 返回Completion(normal, empty, empty)
 */
if (debuggerImplementation) {
  return debuggerCompletion
} else {
  return Completion(normal, empty, empty)
}
```

示例

```javascript
function foo () {
  debugger
}
foo()
```
