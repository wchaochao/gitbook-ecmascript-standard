# 循环语句

标签（空格分隔）： ECMAScript规范

---

```
IterationStatement :
  do Statement while ( Expression ); // do/while语句
  while ( Expression ) Statement // while语句
  for ( ExpressionNoIn(opt) ; Expression(opt) ; Expression(opt) ) Statement // for语句 无声明
  for ( var VariableDeclarationListNoIn ; Expression(opt) ; Expression(opt) ) Statement // for语句 有声明
  for ( LeftHandSideExpression in Expression ) Statement  // for/in语句 无声明
  for ( var VariableDeclarationNoIn in Expression ) Statement // for/in语句 有声明
```

## do/while语句

```javascript
IterationStatement : do Statement while ( Expression );

/**
 * 执行Statement，结果为s
 * s.type为normal, 获取Expression的值
 *    转换为false，返回Completion(normal, V, empty)
 *    转换为true，继续下次循环
 * s.type为continue
 *    s.target在循环语句的标签组内，相当于s.type为normal
 *    s.target不在循环语句的标签组内，返回s
 * s.type为break
 *    s.target在循环语句的标签组内，返回Completion(normal, V, empty)
 *    s.target不在循环语句的标签组内，返回s
 * s.type为其他，返回s
 */
let V = empty
repeat {
  let s = Statement
  if (s.value !== empty) {
    V = s.value
  }
  if (s.type !== continue || !IterationStatement.labels.include(s.target)) {
    if (s.type === break && IterationStatement.labels.include(s.target)) {
      return Completion(normal, V, empty)
    }
    
    if (s.type !== normal) {
      return s
    }
  }
  
  let value = GetValue(Expression)
  if (ToBoolean(value) === false) {
    return Completion(normal, V, empty)
  }
}
```

示例

```javascript
// normal
let i = 0
do {
  i++
} while (i < 3)

// continue
let i = 0
do {
  i++
  if (i === 1) {
    continue
  }
} while (i < 3)

// continue label
outer: for (let j = 0; j < 3; j++) {
  let i = 0
  do {
    i++
    if (i === 1) {
      continue outer
    }
  } while (i < 3)
}

// break
let i = 0
do {
  i++
  if (i === 1) {
    break
  }
} while (i < 3)

// break label
outer: for (let j = 0; j < 3; j++) {
  let i = 0
  do {
    i++
    if (i === 1) {
      break outer
    }
  } while (i < 3)
}

// throw
let i = 0
do {
  i++
  if (i === 1) {
    throw i
  }
} while (i < 3)
```

## while语句

```javascript
IterationStatement : while ( Expression ) Statement

/**
 * 获取Expression的值
 * 转换为false，返回Completion(normal, V, empty)
 * 转换为true，执行Statement，结果为s
 *     s.type为normal, 继续下次循环
 *     s.type为continue
 *         s.target在循环语句的标签组内，继续下次循环
 *         s.target不在循环语句的标签组内，返回s
 *     s.type为break
 *         s.target在循环语句的标签组内，返回Completion(normal, V, empty)
 *         s.target不在循环语句的标签组内，返回s
 *     s.type为其他，返回s
 */
let V = empty
repeat {
  let value = GetValue(Expression)
  if (ToBoolean(value) === false) {
    return Completion(normal, V, empty)
  }

  let s = Statement
  if (s.value !== empty) {
    V = s.value
  }
  if (s.type !== continue || !IterationStatement.labels.include(s.target)) {
    if (s.type === break && IterationStatement.labels.include(s.target)) {
      return Completion(normal, V, empty)
    }
    
    if (s.type !== normal) {
      return s
    }
  }
}
```

示例

```javascript
// normal
let i = 0
while (i < 3) {
  i++
} 

// continue
let i = 0
while (i < 3) {
  i++
  if (i === 1) {
    continue
  }
} 

// continue label
outer: for (let j = 0; j < 3; j++) {
  let i = 0
  while (i < 3) {
    i++
    if (i === 1) {
      continue outer
    }
  } 
}

// break
let i = 0
while (i < 3) {
  i++
  if (i === 1) {
    break
  }
} 

// break label
outer: for (let j = 0; j < 3; j++) {
  let i = 0
  while (i < 3) {
    i++
    if (i === 1) {
      break outer
    }
  } 
}

// throw
let i = 0
while (i < 3) {
  i++
  if (i === 1) {
    throw i
  }
} 
```

## for语句

### 无声明

```javascript
IterationStatement : for ( ExpressionNoIn(opt) ; Expression(opt) ; Expression(opt) ) Statement

/**
 * 如果ExpressionNoIn存在，获取ExpressionNoIn的值
 * 如果第一个Expression存在，获取第一个Expression的值
 * 转换为false，返回Completion(normal, V, empty)
 * 转换为true，执行Statement，结果为s
 *     s.type为normal, 如果第二个Expression存在，获取第二个Expression的值并继续下次循环
 *     s.type为continue
 *         s.target在循环语句的标签组内，相当于s.type为normal
 *         s.target不在循环语句的标签组内，返回s
 *     s.type为break
 *         s.target在循环语句的标签组内，返回Completion(normal, V, empty)
 *         s.target不在循环语句的标签组内，返回s
 *     s.type为其他，返回s
 */
if (IsPresent(ExpressionNoIn)) {
  GetValue(ExpressionNoIn)
}
let V = empty
Repeat {
  if (IsPresent(firstExpression)) {
    let value = GetValue(firstExpression)
    if (ToBoolean(value) === false) {
      return Completion(normal, V, empty)
    }
  }
  
  let s = Statement
  if (s.value !== empty) {
    V = s.value
  }
  if (s.type !== continue || !IterationStatement.labels.include(s.target)) {
    if (s.type === break && IterationStatement.labels.include(s.target)) {
      return Completion(normal, V, empty)
    }
    
    if (s.type !== normal) {
      return s
    }
  }
  
  if (IsPresent(secondExpression)) {
    GetValue(secondExpression)
  }
}
```

示例

```javascript
// normal
let i, count = 0
for (i = 0; i < 3; i++) {
  count++
}

// continue
let i, count = 0
for (i = 0; i < 3; i++) {
  count++
  if (count === 1) {
    continue
  }
}

// continue label
outer: for (let j = 0; j < 3; j++) {
  let i, count = 0
  for (i = 0; i < 3; i++) {
    count++
    if (count === 1) {
      continue outer
    }
  }
}

// break
let i, count = 0
for (i = 0; i < 3; i++) {
  count++
  if (count === 1) {
    break
  }
}

// break label
outer: for (let j = 0; j < 3; j++) {
  let i, count = 0
  for (i = 0; i < 3; i++) {
    count++
    if (count === 1) {
      break outer
    }
  }
}

// throw
let i, count = 0
for (i = 0; i < 3; i++) {
  count++
  if (count === 1) {
    throw 1
  }
}
```

### 有声明

```javascript
IterationStatement : for ( var VariableDeclarationListNoIn ; Expression(opt) ; Expression(opt) ) Statement

/**
 * 执行VariableDeclarationListNoIn
 * 如果第一个Expression存在，获取第一个Expression的值
 * 转换为false，返回Completion(normal, V, empty)
 * 转换为true，执行Statement，结果为s
 *     s.type为normal, 如果第二个Expression存在，获取第二个Expression的值并继续下次循环
 *     s.type为continue
 *         s.target在循环语句的标签组内，相当于s.type为normal
 *         s.target不在循环语句的标签组内，返回s
 *     s.type为break
 *         s.target在循环语句的标签组内，返回Completion(normal, V, empty)
 *         s.target不在循环语句的标签组内，返回s
 *     s.type为其他，返回s
 */
exec(VariableDeclarationListNoIn)
let V = empty
Repeat {
  if (IsPresent(firstExpression)) {
    let value = GetValue(firstExpression)
    if (ToBoolean(value) === false) {
      return Completion(normal, V, empty)
    }
  }
  
  let s = Statement
  if (s.value !== empty) {
    V = s.value
  }
  if (s.type !== continue || !IterationStatement.labels.include(s.target)) {
    if (s.type === break && IterationStatement.labels.include(s.target)) {
      return Completion(normal, V, empty)
    }
    
    if (s.type !== normal) {
      return s
    }
  }
  
  if (IsPresent(secondExpression)) {
    GetValue(secondExpression)
  }
}
```

示例

```javascript
// normal
let count = 0
for (let i = 0; i < 3; i++) {
  count++
}

// continue
let count = 0
for (let i = 0; i < 3; i++) {
  count++
  if (count === 1) {
    continue
  }
}

// continue label
outer: for (let j = 0; j < 3; j++) {
  let count = 0
  for (let i = 0; i < 3; i++) {
    count++
    if (count === 1) {
      continue outer
    }
  }
}

// break
let count = 0
for (let i = 0; i < 3; i++) {
  count++
  if (count === 1) {
    break
  }
}

// break label
outer: for (let j = 0; j < 3; j++) {
  let count = 0
  for (let i = 0; i < 3; i++) {
    count++
    if (count === 1) {
      break outer
    }
  }
}

// throw
let count = 0
for (let i = 0; i < 3; i++) {
  count++
  if (count === 1) {
    throw 1
  }
}
```

## for/in语句

### 无声明

```javascript
IterationStatement : for ( LeftHandSideExpression in Expression ) Statement

/**
 * 获取Expression的值
 * 不能转换为对象，返回Completion(normal, empty, empty)
 * 能转换为对象，转换为对象
 * 获取对象的下一个可枚举属性名
 * 不存在，返回Completion(normal, V, empty)
 * 存在，将属性名赋给LeftHandSideExpression，执行Statement，结果为s
 *     s.type为normal, 继续下次循环
 *     s.type为continue
 *         s.target在循环语句的标签组内，继续下次循环
 *         s.target不在循环语句的标签组内，返回s
 *     s.type为break
 *         s.target在循环语句的标签组内，返回Completion(normal, V, empty)
 *         s.target不在循环语句的标签组内，返回s
 *     s.type为其他，返回s
 */
let value = GetValue(Expression)
let obj
if (value === undefined || value === null) {
  return Completion(normal, empty, empty)
} else {
  obj = ToObject(value)
}

let V = empty
Repeat {
  let P = nextEnumerableProperty(obj)
  if (!P) {
    return Completion(normal, V, empty)
  }
  
  let lhs = LeftHandSideExpression
  PutValue(lhs, P)

  let s = Statement
  if (s.value !== empty) {
    V = s.value
  }
  if (s.type !== continue || !IterationStatement.labels.include(s.target)) {
    if (s.type === break && IterationStatement.labels.include(s.target)) {
      return Completion(normal, V, empty)
    }
    
    if (s.type !== normal) {
      return s
    }
  }
}
```

示例

```javascript
// normal
let o = {a: 1}
let i, count = 0
for (i in o) {
  count++
}

// continue
let o = {a: 1}
let i, count = 0
for (i in o) {
  count++
  if (count === 1) {
    continue
  }
}

// continue label
outer: for (let i = 0; i < 3; i++) {
  let o = {a: 1}
  let i, count = 0
  for (i in o) {
    count++
    if (count === 1) {
      continue outer
    }
  } 
}

// break
let o = {a: 1}
let i, count = 0
for (i in o) {
  count++
  if (count === 1) {
    break
  }
}

// break label
outer: for (let i = 0; i < 3; i++) {
  let o = {a: 1}
  let i, count = 0
  for (i in o) {
    count++
    if (count === 1) {
      break outer
    }
  } 
}

// throw
let o = {a: 1}
let i, count = 0
for (i in o) {
  count++
  if (count === 1) {
    throw 1
  }
}
```

### 有声明

```javascript
IterationStatement : for ( var VariableDeclarationNoIn in Expression ) Statement

/**
 * 执行VariableDeclarationNoIn，获取变量名
 * 获取Expression的值
 * 不能转换为对象，返回Completion(normal, empty, empty)
 * 能转换为对象，转换为对象
 * 获取对象的下一个可枚举属性名
 * 不存在，返回Completion(normal, V, empty)
 * 存在，将属性名赋给变量，执行Statement，结果为s
 *     s.type为normal, 继续下次循环
 *     s.type为continue
 *         s.target在循环语句的标签组内，继续下次循环
 *         s.target不在循环语句的标签组内，返回s
 *     s.type为break
 *         s.target在循环语句的标签组内，返回Completion(normal, V, empty)
 *         s.target不在循环语句的标签组内，返回s
 *     s.type为其他，返回s
 */
let varName = exec(VariableDeclarationNoIn)
let value = GetValue(Expression)
let obj
if (value === undefined || value === null) {
  return Completion(normal, empty, empty)
} else {
  obj = ToObject(value)
}

let V = empty
Repeat {
  let P = nextEnumerableProperty(obj)
  if (!P) {
    return Completion(normal, V, empty)
  }
  
  let varRef = ToReference(varName)
  PutValue(varRef, P)

  let s = Statement
  if (s.value !== empty) {
    V = s.value
  }
  if (s.type !== continue || !IterationStatement.labels.include(s.target)) {
    if (s.type === break && IterationStatement.labels.include(s.target)) {
      return Completion(normal, V, empty)
    }
    
    if (s.type !== normal) {
      return s
    }
  }
}
```

示例

```javascript
// normal
let o = {a: 1}
let count = 0
for (let i in o) {
  count++
}

// continue
let o = {a: 1}
let count = 0
for (let i in o) {
  count++
  if (count === 1) {
    continue
  }
}

// continue label
outer: for (let i = 0; i < 3; i++) {
  let o = {a: 1}
  let count = 0
  for (let i in o) {
    count++
    if (count === 1) {
      continue outer
    }
  } 
}

// break
let o = {a: 1}
let count = 0
for (let i in o) {
  count++
  if (count === 1) {
    break
  }
}

// break label
outer: for (let i = 0; i < 3; i++) {
  let o = {a: 1}
  let count = 0
  for (let i in o) {
    count++
    if (count === 1) {
      break outer
    }
  } 
}

// throw
let o = {a: 1}
let count = 0
for (let i in o) {
  count++
  if (count === 1) {
    throw 1
  }
}
```
