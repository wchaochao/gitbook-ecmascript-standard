# 其他表达式

标签（空格分隔）： ECMAScript规范

---

## 位运算表达式

```
BitwiseANDExpression : // 按位与
  EqualityExpression // 相等表达式
  BitwiseANDExpression & EqualityExpression

BitwiseXORExpression : // 按位异或
  BitwiseANDExpression
  BitwiseXORExpression ^ BitwiseANDExpression

BitwiseORExpression : // 按位或
  BitwiseXORExpression
  BitwiseORExpression | BitwiseXORExpression
```

### 算法

```javascript
A : A @ B // @为&, ^, |

/*
 * 获取A的值并转换为Int32类型
 * 获取B的值并转换为Int32类型
 * 执行相应的位运算，结果为Int32类型
 */
let leftValue = ToInt32(GetValue(A))
let rightValue = ToInt32(GetValue(B))
return bitwise(leftValue, rightValue)
```

示例

```javascript
// 按位与
1 & 0 // 0
1 & -1 // 1
7 & 4 // 4

// 按位异或
1 ^ 0 // 1
1 ^ -1 // -2

// 按位或
1 | 0 // 1
1 | -1 // -1
```

## 逻辑表达式

```
LogicalANDExpression: // 逻辑与
  BitwiseORExpression
  LogicalANDExpression && BitwiseORExpression
  
LogicalORExpression : // 逻辑或
  LogicalANDExpression
  LogicalORExpression || LogicalANDExpression
```

### 逻辑与

```javascript
LogicalANDExpression : LogicalANDExpression && BitwiseORExpression

/**
 * 获取LogicalANDExpression的值
 * LogicalANDExpression转换为false，返回LogicalANDExpression的值
 * LogicalANDExpression转换为true，返回BitwiseORExpression的值
 */
let leftValue = GetValue(LogicalANDExpression)
if (ToBoolean(leftValue) === false) {
  return leftValue
} else {
  return GetValue(BitwiseORExpression)
}
```

示例

```javascript
let o
o && o.a // undefined
'' && 1 // ''
1 && 'a' && NaN && true // NaN
```

### 逻辑或

```javascript
LogicalORExpression : LogicalORExpression || LogicalANDExpression

/**
 * 获取LogicalORExpression的值
 * LogicalORExpression转换为true，返回LogicalORExpression的值
 * LogicalORExpression转换为false，返回LogicalANDExpression的值
 */
let leftValue = GetValue(LogicalANDExpression)
if (ToBoolean(leftValue) === true) {
  return leftValue
} else {
  return GetValue(BitwiseORExpression)
}
```

示例

```javascript
let o = {a: 1}
o || o.a // {a: 1}
'' || 1 // 1
1 || 'a' || NaN || true // 1
```

## 条件表达式

```
ConditionalExpression :
  LogicalORExpression
  LogicalORExpression ? AssignmentExpression : AssignmentExpression
```

### 算法

```javascript
ConditionalExpression : LogicalORExpression ? AssignmentExpression : AssignmentExpression

/**
 * 获取LogicalORExpression的值
 * LogicalORExpression转换为true，返回第一个AssignmentExpression的值
 * LogicalORExpression转换为false，返回第二个AssignmentExpression的值
 */
let leftValue = GetValue(LogicalORExpression)
if (ToBoolean(leftValue) === true) {
  return GetValue(FirstAssignmentExpression)
} else {
  return GetValue(SecondAssignmentExpression)
}
```

示例

```javascript
true ? 1 : 0 // 1
false ? 1 : 0 // 0
```

## 赋值表达式

```
AssignmentExpression :
  ConditionalExpression
  LeftHandSideExpression = AssignmentExpression // 简单赋值
  LeftHandSideExpression AssignmentOperator AssignmentExpression // 复合赋值

AssignmentOperator : one of
  *=  /=  %=  +=  -=  <<=  >>=  >>>=  &=  ^=  |=
```

### 简单赋值

```javascript
AssignmentExpression : LeftHandSideExpression = AssignmentExpression

/*
 * LeftHandSideExpression为引用类型
 *     该引用为环境引用，且引用名为eval或arguments，且为严格模式时，抛出SyntaxError
 *     其他情况设置引用值为AssignmentExpression的值，并返回AssignmentExpression的值
 */
let lhs = LeftHandSideExpression
let rightValue = GetValue(AssignmentExpression)

if (Type(lhs) === Reference && Type(GetBase(lhs)) === Environment Record && (GetReferenceName(lhs) === 'eval' || GetReferenceName(lhs) === 'arguments') && IsStrictReference(lhs) = true) {
  throw SyntaxError
}

PutValue(lhs, rightValue)
return rightValue
```

示例

```javascript
let a = 1
let b
b = true
```

### 复合赋值

```javascript
AssignmentExpression : LeftHandSideExpression @= AssignmentExpression

/* 
 * 获取LeftHandSideExpression和AssignmentExpression的值并执行@操作，结果为r
 * LeftHandSideExpression为引用类型
 *     该引用为环境引用，且引用名为eval或arguments，且为严格模式时，抛出SyntaxError
 *     其他情况设置引用值为r，并返回r
 */
let lhs = LeftHandSideExpression
let leftValue = GetValue(LeftHandSideExpression)
let rightValue = GetValue(AssignmentExpression)
let r = @(leftValue, rightValue)

if (Type(lhs) === Reference && Type(GetBase(lhs)) === Environment Record && (GetReferenceName(lhs) === 'eval' || GetReferenceName(lhs) === 'arguments') && IsStrictReference(lhs) = true) {
  throw SyntaxError
}

PutValue(lhs, r)
return r
```

示例

```javascript
let a = 32
a *= 2 // 64

let a = 32
a /= 2 // 16

let a = 32
a %= 10 // 2

let a = 32
a += 2 // 34

let a = 32
a -= 2 // 30

let a = 32
a <<= 2 // 128

let a = 32
a >>= 2 // 8

let a = 32
a >>>= 2 // 8

let a = 32
a &= 0 // 0

let a = 32
a ^= 0 // 32

let a = 32
a |= 0 // 32
```

## 逗号表达式

```
Expression :
  AssignmentExpression
  Expression , AssignmentExpression
```

### 算法

```javascript
Expression : Expression , AssignmentExpression

/* 
 * 获取Expression和AssignmentExpression的值，并返回AssignmentExpression的值
 */
GetValue(Expression)
return GetValue(AssignmentExpression)
```

示例

```javascript
1, true // true
```
