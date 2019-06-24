# 相等表达式

标签（空格分隔）： ECMAScript规范

---

```
EqualityExpression :
  RelationalExpression // 关系表达式
  EqualityExpression == RelationalExpression // 相等
  EqualityExpression != RelationalExpression // 不等
  EqualityExpression === RelationalExpression // 全等
  EqualityExpression !== RelationalExpression // 不全等
```

## 相等比较

```javascript
equality(x, y)
equality(y, x)

/*
 * 两值类型相同
 *     有一值为NaN，返回false
 *     一值为+0，一值为-0，返回true
 *     其他情况按照SameValue算法计算
 * 两值类型不同
 *     一值为undefined, 一值为null, 返回true
 *     一值Number类型，一值为String类型，转换为Number类型进行相等比较
 *     一值为Boolean类型，将该值转换为Number类型与另一值进行相等比较
 *     一值为Object类型，将该值转换为Primitive类型与另一值进行相等比较
 */
if (Type(x) === Type(y)) {
  if (Type(x) === Number) {
    if (SameValue(x, NaN)) {
      return false
    }
    
    if (SameValue(x, +0) && SameValue(y, -0)) {
      return true
    }
  }
  
  return SameValue(x, y)
}

if (Type(x) === Undefined && Type(y) === Null) {
  return true
}

if (Type(x) === Number && Type(y) === String) {
  return x === ToNumber(y)
}

if (Type(x) === Boolean) {
  return ToNumber(x) === y
}

if (Type(x) === Object) {
  return ToPrimitive(x) === y
}

return false
```

示例

```javascript
// 类型相同
equality(NaN, NaN) // false
equality(+0, -0) // true
equality(undefined, undefined) // true
equality(null, null) // true
equality(true, true) // true
equality('1', '\x31') // true
equality(({}, {}) // false
equality(1, 0x1) // true

// 类型不同
equality(undefined, null) // true
equality(undefined, 1) // false
equality(1, '1') // true
equality('1', true) // true
equality(true, new Number(1)) // true
```

## 相等

```javascript
EqualityExpression : EqualityExpression == RelationalExpression

/**
 * 获取EqualityExpression和RelationalExpression的值
 * 执行相等比较算法equality(leftValue, rightValue)
 */
let leftValue = GetValue(EqualityExpression)
let rightValue = GetValue(RelationalExpression)
return equality(leftValue, rightValue)
```

示例

```javascript
// 类型相同
NaN == NaN // false
+0 == -0 // true
undefined == undefined // true
null == null // true
true == true // true
'1' == '\x31' // true
({}) == ({}) // false
1 == 0x1 // true

// 类型不同
undefined == null // true
undefined == 1 // false
1 == '1' // true
'1' == true // true
true == new Number(1) // true
```

## 不相等

```javascript
EqualityExpression : EqualityExpression != RelationalExpression

/**
 * 获取EqualityExpression和RelationalExpression的值
 * 执行相等比较算法equality(leftValue, rightValue)，逻辑取反
 */
let leftValue = GetValue(EqualityExpression)
let rightValue = GetValue(RelationalExpression)
return !equality(leftValue, rightValue)
```

示例

```javascript
// 类型相同
NaN != NaN // true
+0 != -0 // false
undefined != undefined // false
null != null // false
true != true // false
'1' != '\x31' // false
({}) != ({}) // true
1 != 0x1 // false

// 类型不同
undefined != null // false
undefined != 1 // true
1 != '1' // false
'1' != true // false
true != new Number(1) // false
```

## 严格相等比较

```javascript
strictEquality(x, y)
strictEquality(y, x)

/*
 * 两值类型不同，返回false
 * 两值类型相同
 *     有一值为NaN，返回false
 *     一值为+0，一值为-0，返回true
 *     其他情况按照SameValue算法计算
 */
if (Type(x) !== Type(y)) {
  return false
} else {
  if (Type(x) === Number) {
    if (SameValue(x, NaN)) {
      return false
    }
    
    if (SameValue(x, +0) && SameValue(y, -0)) {
      return true
    }
  }
  
  return SameValue(x, y)
}
```

示例

```javascript
strictEquality(undefined, null) // false
strictEquality(NaN, NaN) // false
strictEquality(+0, -0) // true
strictEquality(undefined, undefined) // true
strictEquality(null, null) // true
strictEquality(true, true) // true
strictEquality('1', '\x31') // true
strictEquality(({}, {}) // false
strictEquality(1, 0x1) // true
```

## 严格相等

```javascript
EqualityExpression : EqualityExpression === RelationalExpression

/**
 * 获取EqualityExpression和RelationalExpression的值
 * 执行严格相等比较算法strictEquality(leftValue, rightValue)
 */
let leftValue = GetValue(EqualityExpression)
let rightValue = GetValue(RelationalExpression)
return strictEquality(leftValue, rightValue)
```

示例

```javascript
undefined === null // false
NaN === NaN // false
+0 === -0 // true
undefined === undefined // true
null === null // true
true === true // true
'1' === '\x31' // true
({}) === ({}) // false
1 === 0x1 // true
```

## 严格不相等

```javascript
EqualityExpression : EqualityExpression !== RelationalExpression

/**
 * 获取EqualityExpression和RelationalExpression的值
 * 执行严格相等比较算法strictEquality(leftValue, rightValue)，逻辑取反
 */
let leftValue = GetValue(EqualityExpression)
let rightValue = GetValue(RelationalExpression)
return !strictEquality(leftValue, rightValue)
```

示例

```javascript
undefined !== null // true
NaN !== NaN // true
+0 !== -0 // false
undefined !== undefined // false
null !== null // false
true !== true // false
'1' !== '\x31' // false
({}) !== ({}) // true
1 !== 0x1 // false
```
