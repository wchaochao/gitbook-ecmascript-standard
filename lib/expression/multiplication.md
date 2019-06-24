# 乘法表达式

标签（空格分隔）： ECMAScript规范

---

```
MultiplicativeExpression:
  UnaryExpression // 一元表达式
  MultiplicativeExpression * UnaryExpression // 乘法
  MultiplicativeExpression / UnaryExpression // 除法
  MultiplicativeExpression % UnaryExpression // 求模
```

## 算法

```javascript
MultiplicativeExpression : MultiplicativeExpression @ UnaryExpression // @为* / %

/*
 * 获取MultiplicativeExpression的值并转换为Number类型
 * 获取UnaryExpression的值并转换为Number类型
 * 对两值进行乘法、除法、求模运算
 */
let leftValue = ToNumber(GetValue(MultiplicativeExpression))
let rightValue = ToNumber(GetValue(UnaryExpression))

return leftValue @ rightValue
```

## 乘法

```javascript
MultiplicativeExpression: oneValue * anotherValue
MultiplicativeExpression: anotherValue * oneValue

/*
 * 有一值为NaN，返回NaN
 * 有一值为无穷大
 *     另一值为0，返回NaN
 *     另一值不为0，返回无穷大，符号为两个数的符号相乘
 * 其他情况进行乘法运算
 */
if (SameValue(oneValue, NaN)) {
  return NaN
}

if (abs(oneValue) === Infinity) {
  if (abs(anotherValue) === 0) {
    return NaN
  } else {
    return multiply(sign(oneValue), sign(anotherValue), Infinity)
  }
}

return multiply(oneValue, anotherValue)  
```

示例

```javascript
NaN * 0 // NaN
Infinity * 0 // NaN
Infinity * 1 // Infinity
Infinity * -1 // -Infinity
3 * 4 // 12
3 * (-4) // -12
```

## 除法

```javascript
MultiplicativeExpression: leftValue / rightValue

/*
 * 被除数或除数为NaN，返回NaN
 * 被除数为无穷大
 *     除数为无穷大，返回NaN
 *     除数不为无穷大，返回无穷大，符号为两个数的符号相乘
 * 被除数不为无穷大
 *     除数为无穷大，返回0，符号为两个数的符号相乘
 * 被除数为0
 *     除数为0，返回NaN
 *     除数不为0，返回0，符号为两个数的符号相乘
 * 被除数不为0
 *     除数为0，返回无穷大，符号为两个数的符号相乘
 * 其他情况进行除法运算
 */
if (SameValue(leftValue, NaN) || SameValue(rightValue, NaN)) {
  return NaN
}

if (abs(leftValue) === Infinity) {
  if (abs(rightValue) === Infinity) {
    return NaN
  } else {
    return multiply(sign(leftValue), sign(rightValue), Infinity)
  }
} else if (abs(rightValue) === Infinity) {
  return multiply(sign(leftValue), sign(rightValue), 0)
}

if (abs(leftValue) === 0) {
  if (abs(rightValue) === 0) {
    return NaN
  } else {
    return multiply(sign(leftValue), sign(rightValue), 0)
  }
} else if (abs(rightValue) === 0) {
  return multiply(sign(leftValue), sign(rightValue), Infinity)
}

return divide(leftValue, rightValue)
```

示例

```javascript
NaN / 1 // NaN
1 / NaN // NaN
Infinity / Infinity // NaN
Infinity / +1 // Infinity
Infinity / -1 // -Infinity
+1 / Infinity // +0
-1 / Infinity // -0
+0 / -0 // NaN
+0 / 1 // +0
-0 / 1 // -0
1 / +0 // Infinity
1 / -0 // -Infinity
12 / 3 // 4
12 / (-3) // -4
```

## 求模

```javascript
MultiplicativeExpression: leftValue % rightValue

/*
 * 被除数或除数为NaN，返回NaN
 * 被除数为无穷大或除数为0，返回NaN
 * 被除数为0或除数为无穷大，返回被除数
 * 其他情况进行求余运算，结果与被除数符号相同
 */
if (SameValue(leftValue, NaN) || SameValue(rightValue, NaN)) {
  return NaN
}

if (abs(leftValue) === Infinity || abs(rightValue) === 0) {
  return NaN
}

if (abs(leftValue) === 0 || abs(rightValue) === Infinity) {
  return leftValue
}

return module(leftValue, rightValue)
```

示例

```javascript
NaN % 1 // NaN
1 % NaN // NaN
Infinity / 1 // NaN
1 / 0 // NaN
0 / 1 // 0
1 / Infinity // 1
-1 / Infinity // -1
12 / 5 // 2
-12 / 5 // -2
```
