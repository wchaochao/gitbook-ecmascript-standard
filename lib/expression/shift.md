# 移位表达式

标签（空格分隔）： ECMAScript规范

---

```
ShiftExpression :
  AdditiveExpression // 加法表达式
  ShiftExpression << AdditiveExpression // 左移
  ShiftExpression >> AdditiveExpression // 有符号右移
  ShiftExpression >>> AdditiveExpression // 无符号右移
```

## 左移

```javascript
ShiftExpression : ShiftExpression << AdditiveExpression

/*
 * 获取ShiftExpression的值并转换为Int32类型
 * 获取AdditiveExpression的值并转换为UInt32类型，取最后五位二进制为位移数
 * 对左值进行左移，右补0，结果为Int32类型
 */
let leftValue = ToInt32(GetValue(ShiftExpression))
let rightValue = ToUInt32(GetValue(AdditiveExpression))
let shiftCount = rightValue & 0x1F

return leftShift(leftValue, shiftCount)
```

示例

```javascript
NaN << 2 // 0
Infinity << 2 // 0
0 << 2 // 0
32 << 2 // 128
-32 << 2 // -128
32 << 34 // 32 << 2, 128
32 << -2 // 32 << 30, 0
1 << -1 // 1 << 31, -2^31
(1 << -1) << 1 // 0
```

## 有符号右移

```javascript
ShiftExpression : ShiftExpression >> AdditiveExpression

/*
 * 获取ShiftExpression的值并转换为Int32类型
 * 获取AdditiveExpression的值并转换为UInt32类型，取最后五位二进制为位移数
 * 对左值进行右移，左补符号位，结果为Int32类型
 */
let leftValue = ToInt32(GetValue(ShiftExpression))
let rightValue = ToUInt32(GetValue(AdditiveExpression))
let shiftCount = rightValue & 0x1F

return signedRightShift(leftValue, shiftCount)
```

示例

```javascript
NaN >> 2 // 0
Infinity >> 2 // 0
0 >> 2 // 0
32 >> 2 // 8
-32 >> 2 // -8
32 >> 34 // 32 >> 2, 8
32 >> -2 // 32 >> 30, 0
1 >> -1 // 1 >> 31, 0
(1 << -1) >> -1 // -1
```

## 无符号右移

```javascript
ShiftExpression : ShiftExpression >>> AdditiveExpression

/*
 * 获取ShiftExpression的值并转换为UInt32类型
 * 获取AdditiveExpression的值并转换为UInt32类型，取最后五位二进制为位移数
 * 对左值进行右移，左补0，结果为UInt32类型
 */
let leftValue = ToUInt32(GetValue(ShiftExpression))
let rightValue = ToUInt32(GetValue(AdditiveExpression))
let shiftCount = rightValue & 0x1F

return unsignedRightShift(leftValue, shiftCount)
```

示例

```javascript
NaN >>> 2 // 0
Infinity >>> 2 // 0
0 >>> 2 // 0
32 >>> 2 // 8
-32 >>> 2 // 2^30 - 8
32 >>> 34 // 32 >>> 2, 8
32 >>> -2 // 32 >>> 30, 0
1 >>> -1 // 1 >>> 31, 0
(1 << -1) >>> -1 // 1
```
