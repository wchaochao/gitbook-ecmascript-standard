# 条件语句

标签（空格分隔）： ECMAScript规范

---

## if语句

```
IfStatement :
  if ( Expression ) Statement else Statement // if/else
  if ( Expression ) Statement // 单if
```

### if/else

```javascript
IfStatement : if ( Expression ) Statement else Statement

/**
 * 获取Expression的值
 * 转换为true，执行第一个语句并返回结果
 * 转换为false，执行第二个语句并返回结果
 */
let value = GetValue(Expression)
if (ToBoolean(value) === true) {
  return firstStatement
} else {
  return secondStatement
}
```

示例

```javascript
let a = 10

if (a > 5) {
  a++
} else {
  a--
}

if (a > 5) {
  a++
} else if (a > 0) {
  a--
} else {
  a++
}
```

### 单If

```javascript
IfStatement : if ( Expression ) Statement

/**
 * 获取Expression的值
 * 转换为true，执行Statement并返回结果
 * 转换为false，返回Completion(normal, empty, empty)
 */
let value = GetValue(Expression)
if (ToBoolean(value) === true) {
  return Statement
} else {
  return Completion(normal, empty, empty)
}
```

示例

```javascript
let a = 10

if (a > 5) {
  a++
}
```

## switch语句

```
SwitchStatement :
  switch ( Expression ) CaseBlock // switch

CaseBlock : // case块
  { CaseClauses(opt) } // case项 不含default case
  { CaseClauses(opt) DefaultClause CaseClauses(opt) } // case项 含default case

CaseClauses : // case项
  CaseClause // 单个case项
  CaseClauses CaseClause // 多个case项

CaseClause : // 单个case项
  case Expression : StatementList(opt)

DefaultClause :
  default : StatementList(opt)
```

### switch

```javascript
SwitchStatement : switch ( Expression ) CaseBlock

/**
 * 获取Expression的值并传给CaseBlock执行获得R
 * R.type为break且R.target在当前的labels内，返回Completion(normal, R.value, empty)
 * 其他情况，返回R
 */
let value = GetValue(Expression)
let R = exec(CaseBlock, value)
if (R.type === break && currentLabels.include(s.target)) {
  return Completion(normal, R.value, empty)
} else {
  return R
}
```

示例

```javascript
// break
let a = 1
let count = 0
switch (a) { // count = 1
  case 1: 
    count += 1
    break
  case 2:
    count += 2
    break
}

// 非break
let a = 1
let count = 0
switch (a) { // count = 3
  case 1: 
    count += 1
  case 2:
    count += 2
}
```

### case块

传入参数input

#### 无default

```javascript
CaseBlock : { CaseClauses(opt) }

/**
 * 遍历case项，匹配input
 * 有匹配，执行匹配case项的statementList
 *     结果为abrupt completion, 返回Completion(R.type, V, R.target)
 *     结果为normal completion, 执行下一个case项的statementList，同上
 * 无匹配，返回Completion(normal, V, empty)
 */
let V = empty
let A = CaseClauses
let C

let found = false
while (C = A.nextCaseClause) {
  if (found === false) {
    let causeSelector = GetValue(C.Expression)
    if (causeSelector === input) {
      found = true
    }
  }
  if (found === true) {
    if (C.statementList !== empty) {
      let R = C.statementList
      V = R.value !== empty ? R.value : V
      if (R.type !== normal) {
        return Completion(R.type, V, R.target)
      }
    }
  }
}

return Completion(normal, V, empty)
```

示例

```javascript
// 有匹配且中断
let a = 1
let count = 0
switch (a) { // count = 1
  case 1: 
    count += 1
    break
  case 2:
    count += 2
    break
}

// 有匹配且不中断
let a = 1
let count = 0
switch (a) { // count = 3
  case 1: 
    count += 1
  case 2:
    count == 2
}

// 无匹配
let a = '1'
let count = 0
switch (a) { // count = 0
  case 1: 
    count += 1
    break
  case 2:
    count += 2
    break
}
```

#### 有default

```javascript
CaseBlock : { CaseClauses(opt) DefaultClause CaseClauses(opt) }

/**
 * 遍历第一个case块，匹配input
 * 有匹配，执行匹配case项的statementList
 *     结果为abrupt completion, 返回Completion(R.type, V, R.target)
 *     结果为normal completion, 执行下一个case项（A、default、B）的statementList，同上
 * 无匹配，遍历第二个case块，匹配input
 *    有匹配，执行匹配case项的statementList
 *        结果为abrupt completion, 返回Completion(R.type, V, R.target)
 *        结果为normal completion, 执行下一个case项（B）的statementList，同上
 *    无匹配，匹配default case
 *        结果为abrupt completion, 返回Completion(R.type, V, R.target)
 *        结果为normal completion, 执行下一个case项（B）的statementList，同上
 */
let V = empty
let A = firstCaseClauses
let B = secondCaseClauses
let C

let found = false
while (C = A.nextCaseClause) {
  if (found === false) {
    let causeSelector = GetValue(C.Expression)
    if (causeSelector === input) {
      found = true
    }
  }
  if (found === true) {
    if (C.statementList !== empty) {
      let R = C.statementList
      V = R.value !== empty ? R.value : V
      if (R.type !== normal) {
        return Completion(R.type, V, R.target)
      }
    }
  }
}

let foundInB = false
if (found === false) {
  while (foundInB === false) {
    C = B.nextCaseClause
    let causeSelector = GetValue(C.Expression)
    if (causeSelector === input) {
      foundInB = true
      
      if (C.statementList !== empty) {
        let R = C.statementList
        V = R.value !== empty ? R.value : V
        if (R.type !== normal) {
          return Completion(R.type, V, R.target)
        }
      }
    }
  }
}

if (foundInB === false) {
  let R = defaultCase.statementList
  V = R.value !== empty ? R.value : V
  if (R.type !== normal) {
    return Completion(R.type, V, R.target)
  }
}

while (true) {
  let C = B.nextCaseClause
  if (C === empty) {
    return Completion(normal, V, empty)
  }
  
  if (C.statementList !== empty) {
    let R = C.statementList
    V = R.value !== empty ? R.value : V
    if (R.type !== normal) {
      return Completion(R.type, V, R.target)
    }
  }
}
```

匹配第一个case块，且中断

```javascript
let a = 2
let count = 0
switch (a) { // count = 2
  case 1: 
    count+=1
    break
  case 2:
    count+=2
    break
  default: 
    count+=3
    break
  case 4: 
    count+=4
    break
  case 5:
    count+=5
    break
}
```

匹配第一个case块，且不中断

```javascript
let a = 2
let count = 0
switch (a) { // count = 5
  case 1: 
    count+=1
    break
  case 2:
    count+=2
  default: 
    count+=3
    break
  case 4: 
    count+=4
    break
  case 5:
    count+=5
    break
}
```

匹配第二个case块，且中断

```javascript
let a = 4
let count = 0
switch (a) { // count = 4
  case 1: 
    count+=1
    break
  case 2:
    count+=2
    break
  default: 
    count+=3
    break
  case 4: 
    count+=4
    break
  case 5:
    count+=5
    break
}
```

匹配第二个case块，且不中断

```javascript
let a = 4
let count = 0
switch (a) { // count = 9
  case 1: 
    count+=1
    break
  case 2:
    count+=2
    break
  default: 
    count+=3
    break
  case 4: 
    count+=4
  case 5:
    count+=5
    break
}
```

匹配default，且中断

```javascript
let a = 3
let count = 0
switch (a) { // count = 3
  case 1: 
    count+=1
    break
  case 2:
    count+=2
    break
  default: 
    count+=3
    break
  case 4: 
    count+=4
    break
  case 5:
    count+=5
    break
}
```

匹配default，且不中断

```javascript
let a = 3
let count = 0
switch (a) { // count = 7
  case 1:
    count+=1
    break
  case 2:
    count+=2
    break
  default:
    count+=3
  case 4:
    count+=4
    break
  case 5:
    count+=5
    break
}
```
