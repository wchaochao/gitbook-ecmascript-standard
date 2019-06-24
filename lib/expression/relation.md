# 关系表达式

标签（空格分隔）： ECMAScript规范

---

```
RelationalExpression :
  ShiftExpression // 移位表达式
  RelationalExpression < ShiftExpression // 小于
  RelationalExpression > ShiftExpression // 大于
  RelationalExpression <= ShiftExpression // 小于等于
  RelationalExpression >= ShiftExpression // 大于等于
  RelationalExpression instanceof ShiftExpression // instanceof
  RelationalExpression in ShiftExpression // in
```

## 算法

```javascript
/**
 * 比较x, y，leftFirst表示先计算x还是先计算y, 默认为true
 *     返回true，x小于y
 *     返回false，x不小于y
 *     返回undefined，x或y至少一值为NaN
 */
comparison(x, y, leftFirst)

/**
 * 根据leftFirst按顺序将x, y转换为原始值类型
 * x, y都为String类型
 *     y是x的前缀，返回false
 *     x是y的前缀，返回true
 *     不是前缀，比较x, y第一个不同字符的Unicode码
 * x, y不都为String类型，转换为Number类型
 *     有一值为NaN，返回undefined
 *     两值相等，返回false
 *     有无穷大，按-Infinity < 有限数字 < Infintiy比较
 *     有限数字比较大小
 */
let px
let py
if (leftFirst) {
  px = ToPrimitive(x, 'Number')
  py = ToPrimitive(y, 'Number')
} else {
  py = ToPrimitive(y, 'Number')
  px = ToPrimitive(x, 'Number')
}

if (Type(px) === String && Type(py) === String) {
  if (isPrefix(py, px)) { // px = py + r, py是px的前缀
    return false
  }
  
  if (isPrefix(px, py)) { // py = px + r, px是py的前缀
    return true
  }
  
  let k = diffPosition(px, py) // px, py开始不同时的位置
  let m = charCode(px[k]) // 获取Unicode码
  let n = charCode(py[k])
  return m < n
} else {
  nx = ToNumber(x)
  ny = ToNumber(y)
  if (SameValue(nx, NaN) || SameValue(ny, NaN)) {
    return undefined
  }
  
  if (nx === ny) {
    return false
  }
  
  if (nx === -Infinity || ny === Infinity) {
    return true
  }
  
  if (nx === Infinity || ny === -Infinity) {
    return false
  }
  
  return lessThan(nx, ny)
}
```

示例

```javascript
comparison('123', '123') // false
comparison('123a', '123') // false
comparison('123', '123a') // true
comparison('abcd', 'abd') // true
comparison('abfd', 'abd') // false

comparison(undefined, null) // undefined
comparison(null, false) // false
comparison(+0, -0) // false
comparison(-Infinity, 1) // true
comparison(Infinity, 1) // false
comparison(1, Infinity) // true
comparison(1, -Infinity) // false
comparison(1, 2) // true
comparison(2, 1) // false
```

## 小于

```javascript
RelationalExpression : RelationalExpression < ShiftExpression

/**
 * 获取RelationalExpression和ShiftExpression的值
 * 执行抽象比较算法comparison(leftValue, rightValue)
 *    结果为undefined或false, 返回false
 *    结果为true, 返回true
 */
let leftValue = GetValue(RelationalExpression)
let rightValue = GetValue(ShiftExpression)
let r = comparison(leftValue, rightValue)
if (r === undefined || r === false) {
  return false
} else {
  return true
}
```

示例

```javascript
'123' < '123' // false
'123a' < '123' // false
'123' < '123a' // true
'abcd' < 'abd' // true
'abfd' < 'abd' // false

undefined < null // false
null < false // false
+0 < -0 // false
-Infinity < 1 // true
Infinity < 1 // false
1 < Infinity // true
1 < -Infinity // false
1 < 2 // true
2 < 1 // false
```

## 大于

```javascript
RelationalExpression : RelationalExpression > ShiftExpression

/**
 * 获取RelationalExpression和ShiftExpression的值
 * 执行抽象比较算法comparison(rightValue, leftValue, false)
 *    结果为undefined或false, 返回false
 *    结果为true, 返回true
 */
let leftValue = GetValue(RelationalExpression)
let rightValue = GetValue(ShiftExpression)
let r = comparison(leftValue, rightValue)
if (r === undefined || r === false) {
  return false
} else {
  return true
}
```

示例

```javascript
'123' > '123' // false
'123a' > '123' // true
'123' > '123a' // false
'abcd' > 'abd' // false
'abfd' > 'abd' // true

undefined > null // false
null > false // false
+0 > -0 // false
-Infinity > 1 // false
Infinity > 1 // true
1 > Infinity // false
1 > -Infinity // true
1 > 2 // false
2 > 1 // true
```

## 小于等于

```javascript
RelationalExpression : RelationalExpression <= ShiftExpression

/**
 * 获取RelationalExpression和ShiftExpression的值
 * 执行抽象比较算法comparison(rightValue, leftValue, false)
 *    结果为undefined或true, 返回false
 *    结果为false, 返回true
 */
let leftValue = GetValue(RelationalExpression)
let rightValue = GetValue(ShiftExpression)
let r = comparison(rightValue, leftValue, false)
if (r === undefined || r === true) {
  return false
} else {
  return true
}
```

示例

```javascript
'123' <= '123' // true
'123a' <= '123' // false
'123' <= '123a' // true
'abcd' <= 'abd' // true
'abfd' <= 'abd' // false

undefined <= null // false
null <= false // true
+0 <= -0 // true
-Infinity <= 1 // true
Infinity <= 1 // false
1 <= Infinity // true
1 <= -Infinity // false
1 <= 2 // true
2 <= 1 // false
```

## 大于等于

```javascript
RelationalExpression : RelationalExpression >= ShiftExpression

/**
 * 获取RelationalExpression和ShiftExpression的值
 * 执行抽象比较算法comparison(leftValue, rightValue)
 *    结果为undefined或true, 返回false
 *    结果为true, 返回false
 */
let leftValue = GetValue(RelationalExpression)
let rightValue = GetValue(ShiftExpression)
let r = comparison(leftValue, rightValue)
if (r === undefined || r === true) {
  return false
} else {
  return true
}
```

示例

```javascript
'123' >= '123' // true
'123a' >= '123' // true
'123' >= '123a' // false
'abcd' >= 'abd' // false
'abfd' >= 'abd' // true

undefined >= null // false
null >= false // true
+0 >= -0 // true
-Infinity >= 1 // false
Infinity >= 1 // true
1 >= Infinity // false
1 >= -Infinity // true
1 >= 2 // false
2 >= 1 // true
```

## instanceof

```javascript
RelationalExpression : RelationalExpression instanceof ShiftExpression

/**
 * 获取RelationalExpression和ShiftExpression的值
 * ShiftExpression不是Object或没有[[HasInstance]]内部方法时抛出TypeError
 * ShiftExpression符合条件时，执行[[HasInstance]]方法
 *     ShiftExpression在RelationalExpression的原型链上，返回true
 *     ShiftExpression不在RelationalExpression的原型链上，返回false
 */
let leftValue = GetValue(RelationalExpression)
let rightValue = GetValue(ShiftExpression)

if (Type(rightValue) !== Object || typeof rightValue.[[HasInstance]] !== 'function') {
  throw TypeError
}

return rightValue.[[HasInstance]](leftValue)
```

示例

```javascript
1 instanceof 2 // TypeError
1 instanceof ({}) // TypeError
1 instanceof Number // false
let a = new Number(1)
a instanceof Number // true
a instanceof Object // true
```

## in

```javascript
RelationalExpression : RelationalExpression in ShiftExpression

/**
 * 获取RelationalExpression和ShiftExpression的值
 * ShiftExpression不是Object时抛出TypeError
 * ShiftExpression符合条件时，执行[[HasProperty]]方法
 */
let leftValue = GetValue(RelationalExpression)
let rightValue = GetValue(ShiftExpression)

if (Type(rightValue) !== Object) {
  throw TypeError
}

return rightValue.[[HasProperty]](ToString(leftValue))
```

示例

```javascript
1 in 1 // TypeError
'a' in ({a: 1}) // true
'toString' in ({a: 1}) // true
1 in [,1,] // true
0 in [,1,] // false
```
