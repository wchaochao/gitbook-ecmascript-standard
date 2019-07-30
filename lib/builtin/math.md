# Math对象

标签（空格分隔）： ECMAScript规范

---

数学对象，用于数学运算

## 属性

```javascript
Math.[[Class]] = 'Math'
Math.[[Prototype]] = 'Object.prototype'
Math.[[Extensible]] = true

// 内置属性的属性描述符默认为
{
  [[Writable]]: false,
  [[Enumerable]]: false,
  [[Configurable]]: false
}
```

* `Math.PI`: `π`, 约为 3.14159
* `Math.SQRT2`: `根号2`, 约为 1.414
* `Math.SQRT1_2`: `根号1/2`, 约为 0.707
* `Math.E`: `常量e`, 约为 2.718
* `Math.LN2`: `ln2`, 约为 0.693
* `Math.LN10`: `ln10`, 约为 2.303
* `Math.LOG2E`: `log2(e)`, 约为 1.443
* `Math.LOG10E`: `log10(e)`, 约为 0.434

## 方法

* 参数都转换为Number类型
* 参数为NaN时，返回NaN（pow方法有例外）

**最值方法**

### Math.min([value1 [ , value2 [ , … ] ] ] )

取最小

```javascript
/**
 * 取参数中的最小值，默认为Infinity，其中 +0 > -0
 */
let min = Infinity

for (let i = 0; i < arguments.length; i++) {
  let arg = arguments[i]
  if (arg < min || (SameValue(arg, -0) && SameValue(min, +0))) {
    min = arg
  }
}

return min
```

示例

```javascript
Math.min.length // 2

Math.min(NaN, 1) // NaN
Math.min() // Infinity
Math.min(+0, -0) // -0
Math.min(-1, 0, 1) // -1
```

### Math.max([value1 [ , value2 [ , … ] ] ] )

取最大

```javascript
/**
 * 取参数中的最大值，默认为-Infinity，其中 +0 > -0
 */
let max = -Infinity

for (let i = 0; i < arguments.length; i++) {
  let arg = arguments[i]
  if (arg > max || (SameValue(arg, +0) && SameValue(max, -0))) {
    max = arg
  }
}

return max
```

示例

```javascript
Math.max.length // 2

Math.max(NaN, 1) // NaN
Math.max() // -Infinity
Math.max(+0, -0) // +0
Math.max(-1, 0, 1) // 1
```

**舍入方法**

### Math.floor(x)

向下取整

```javascript
/**
 * x的绝对值为0或Infinity, 返回x
 * x > 0 且 x < 1, 返回+0
 * 其他，返回floor(x)
 */
if (abs(x) === 0 || abs(x) === Infinity) {
  return x
} else if (x > 0 && x < 1){
  return +0
} else {
  return floor(x)
}
```

示例

```javascript
Math.floor(x) === -Math.ceil(-x)

Math.floor(NaN) // NaN
Math.floor(+0) // +0
Math.floor(-0) // -0
Math.floor(Infinity) // Infinity
Math.floor(-Infinity) // -Infinity
Math.floor(0.1) // +0
Math.floor(1) // 1
Math.floor(1.2) // 1
Math.floor(-1.2) // -2
```

### Math.ceil(x)

向上取整

```javascript
/**
 * x的绝对值为0或Infinity, 返回x
 * x > -1 且 x < 0, 返回-0
 * 其他，返回ceil(x)
 */
if (abs(x) === 0 || abs(x) === Infinity) {
  return x
} else if (x > -1 && x < 0){
  return -0
} else {
  return ceil(x)
}
```

示例

```javascript
Math.ceil(x) === -Math.floor(-x)

Math.ceil(NaN) // NaN
Math.ceil(+0) // +0
Math.ceil(-0) // -0
Math.ceil(Infinity) // Infinity
Math.ceil(-Infinity) // -Infinity
Math.ceil(-0.1) // -0
Math.ceil(1) // 1
Math.ceil(1.2) // 2
Math.ceil(-1.2) // -1
```

### Math.round(x)

取最接近的整数，同样接近时取大值

```javascript
/**
 * x的绝对值为0或Infinity, 返回x
 * x > 0 且 x < 0.5, 返回+0
 * x >= -0.5 且 x < 0, 返回-0
 * 其他，返回round(x)
 */
if (abs(x) === 0 || abs(x) === Infinity) {
  return x
} else if (x > 0 && x < 0.5){
  return +0
} else if (x >= -0.5 && x < 0){
  return -0
} else {
  return round(x)
}
```

示例

```javascript
Math.round(NaN) // NaN
Math.round(+0) // +0
Math.round(-0) // -0
Math.round(Infinity) // Infinity
Math.round(-Infinity) // -Infinity
Math.round(0.1) // +0
Math.round(-0.5) // -0
Math.round(1) // 1
Math.round(1.5) // 2
Math.round(-1.8) // -2
Math.round(-1.5) // -1
```

**随机方法**

### Math.random()

获取[0, 1)之间的随机数

```javascript
/**
 * 随机返回[0, 1)之间的数
 */
return random()
```

示例

```javascript
// 两数之间的随机整数
function getRandom(min, max) {
  min = Math.ceil(min);
  max = Math.floor(max);

  return Math.floor(Math.random() * (max - min + 1) + min);
}
```

**幂方法**

### Math.abs(x)

绝对值

```javascript
/**
 * 返回x的绝对值
 */
return abs(x)
```

示例

```javascript
Math.abs(NaN) // NaN
Math.abs(-0) // +0
Math.abs(-Infinity) // Infinity
Math.abs(-2) // 2
Math.abs(2) // 2
```

### Math.pow(x, y)

幂

```javascript
/**
 * y为NaN，返回NaN
 * y的绝对值为0，返回1，即使x为NaN
 * x为NaN且y不为0，返回NaN
 * x的绝对值大于1
 *    y为Infintity，返回Infinity
 *    y为-Infinity，返回+0
 * x的绝对值等于1
 *    y的绝对值为Infintity，返回NaN
 * x的绝对值小于1
 *    y为Infintity，返回+0
 *    y为-Infinity，返回Infinity
 * x为Infinity
 *    y > 0，返回Infinity
 *    y < 0，返回+0
 * x为-Infinity
 *    y > 0
 *       y为奇数，返回-Infinity
 *       y为其他，返回Infinity
 *    y < 0
 *       y为奇数，返回-0
 *       y为其他，返回+0
 * x为+0
 *    y > 0，返回+0
 *    y < 0，返回Infinity
 * x为-0
 *    y > 0
 *       y为奇数，返回-0
 *       y为其他，返回+0
 *    y < 0
 *       y为奇数，返回-Infinity
 *       y为其他，返回Infinity
 * x < 0且y不为整数，返回NaN
 * 其他，返回pow(x, y)
 */
if (SameValue(y, NaN)) {
  return NaN
}
if (abs(y) === 0) {
  return 1
}
if (SameValue(x, NaN)) {
  return NaN
}

if (abs(x) > 1) {
  if (y === Infinity) {
    return Infinity
  }
  if (y === -Infinity) {
    return +0
  }
}
if (abs(x) === 1) {
  if (abs(y) === Infinity) {
    return NaN
  }
}
if (abs(x) < 1) {
  if (y === Infinity) {
    return +0
  }
  if (y === -Infinity) {
    return Infinity
  }
}

if (x === Infinity) {
  if (y > 0) {
    return Infinity
  }
  if (y < 0) {
    return +0
  }
}
if (x === -Infinity) {
  if (y > 0) {
    if (IsOddInteger(y)) {
      return -Infinity
    } else {
      return Infinity
    }
  }
  if (y < 0) {
    if (IsOddInteger(y)) {
      return -0
    } else {
      return +0
    }
  }
}

if (x === +0) {
  if (y > 0) {
    return +0
  }
  if (y < 0) {
    return Infinity
  }
}
if (x === -0) {
  if (y > 0) {
    if (IsOddInteger(y)) {
      return -0
    } else {
      return +0
    }
  }
  if (y < 0) {
    if (IsOddInteger(y)) {
      return -Infinity
    } else {
      return +Infinity
    }
  }
}

if (x < 0 && !IsInteger(y)) {
  return NaN
} else {
  return pow(x, y)
}
```

示例

```javascript
Math.pow(1, NaN) // NaN
Math.pow(NaN, +0) // 1
Math.pow(NaN, 1) // NaN
Math.pow(2, Infinity) // Infinity
Math.pow(-2, -Infinity) // +0
Math.pow(1, Infinity) // NaN
Math.pow(0.2, Infinity) // +0
Math.pow(-0.2, -Infinity) // Infinity
Math.pow(Infinity, 2) // Infinity
Math.pow(Infinity, -2) // +0
Math.pow(-Infinity, 1) // -Infinity
Math.pow(-Infinity, 1.2) // Infinity
Math.pow(-Infinity, -1) // -0
Math.pow(-Infinity, -1.2) // +0
Math.pow(+0, 2) // +0
Math.pow(+0, -2) // Infinity
Math.pow(-0, 1) // -0
Math.pow(-0, 1.2) // +0
Math.pow(-0, -1) // -Infinity
Math.pow(-0, -1.2) // Infinity
Math.pow(-1, 1.2) // NaN
Math.pow(-1, 1) // -1
```

### Math.sqrt(x)

根号

```javascript
/**
 * x < 0, 返回NaN
 * x的绝对值为0, 返回x
 * x为Infinity, 返回Infinity
 * 其他，返回sqrt(x)
 */
if (x < 0) {
  return NaN
} else if (abs(x) === 0) {
  return x
} else if (x === Infinity) {
  return Infinity
} else {
  return sqrt(x)
}
```

示例

```javascript
Math.sqrt(NaN) // NaN
Math.sqrt(-1) // NaN
Math.sqrt(+0) // +0
Math.sqrt(-0) // -0
Math.sqrt(Infinity) // Infinity
Math.sqrt(1) // 1
```

### Math.exp(x)

指数

```javascript
/**
 * x的绝对值为0, 返回1
 * x为Infinity, 返回Infinity
 * x为-Infinity, 返回+0
 * 其他，返回exp(x)
 */
if (abs(x) === 0) {
  return 1
} else if (x === Infinity) {
  return Infinity
} else if (x === -Infinity) {
  return +0
} else {
  return exp(x)
}
```

示例

```javascript
Math.exp(NaN) // NaN
Math.exp(+0) // 1
Math.exp(-0) // 1
Math.exp(Infinity) // Infinity
Math.exp(-Infinity) // +0
Math.exp(1) // e
```

### Math.log(x)

对数

```javascript
/**
 * x < 0, 返回NaN
 * x的绝对值为0, 返回-Infinity
 * x为1, 返回+0
 * x为Infinity, 返回Infinity
 * 其他，返回ln(x)
 */
if (x < 0) {
  return NaN
} else if (abs(x) === 0) {
  return -Infinity
} else if (x === 1) {
  return +0
} else if (x === Infinity) {
  return Infinity
} else {
  return ln(x)
}
```

示例

```javascript
Math.log(NaN) // NaN
Math.log(-1) // NaN
Math.log(+0) // -Infinity
Math.log(-0) // -Infinity
Math.log(1) // +0
Math.log(Infinity) // Infinity
Math.log(Math.E) // 1
```

**三角方法**

### Math.sin(x)

正弦

```javascript
/**
 * x的绝对值为0, 返回x
 * x的绝对值为Infinity, 返回NaN
 * 其他，返回sin(x)
 */
if (abs(x) === 0) {
  return x
} else if (abs(x) === Infinity) {
  return NaN
} else {
  return sin(x)
}
```

示例

```javascript
Math.sin(NaN) // NaN
Math.sin(+0) // +0
Math.sin(-0) // -0
Math.sin(Infinity) // NaN
Math.sin(-Infinity) // NaN
Math.sin(Math.PI / 2) // 1
```

### Math.cos(x)

余弦

```javascript
/**
 * x的绝对值为0, 返回1
 * x的绝对值为Infinity, 返回NaN
 * 其他，返回cos(x)
 */
if (abs(x) === 0) {
  return 1
} else if (abs(x) === Infinity) {
  return NaN
} else {
  return cos(x)
}
```

示例

```javascript
Math.cos(NaN) // NaN
Math.cos(+0) // 1
Math.cos(-0) // 1
Math.cos(Infinity) // NaN
Math.cos(-Infinity) // NaN
Math.cos(Math.PI) // -1
```

### Math.tan(x)

正切

```javascript
/**
 * x的绝对值为0, 返回x
 * x的绝对值为Infinity, 返回NaN
 * 其他，返回tan(x)
 */
if (abs(x) === 0) {
  return x
} else if (abs(x) === Infinity) {
  return NaN
} else {
  return tan(x)
}
```

示例

```javascript
Math.tan(NaN) // NaN
Math.tan(+0) // +0
Math.tan(-0) // -0
Math.tan(Infinity) // NaN
Math.tan(-Infinity) // NaN
Math.tan(Math.PI / 4) // 0.9999999999999999
```

### Math.asin(x)

反正弦

```javascript
/**
 * x < -1 或 x > 1，返回NaN
 * x的绝对值为0, 返回x
 * 其他，返回arcsin(x)
 */
if (x < -1 || x > 1) {
  return NaN
} else if (abs(x) === 0) {
  return x
} else {
  return arcsin(x)
}
```

示例

```javascript
Math.asin(NaN) // NaN
Math.asin(-2) // NaN
Math.asin(2) // NaN
Math.asin(+0) // +0
Math.asin(-0) // -0
Math.asin(1) // +π/2
```

### Math.acos(x)

反余弦

```javascript
/**
 * x < -1 或 x > 1，返回NaN
 * x = 1, 返回+0
 * 其他，返回arccos(x)
 */
if (x < -1 || x > 1) {
  return NaN
} else if (x === 1) {
  return +0
} else {
  return arccos(x)
}
```

示例

```javascript
Math.acos(NaN) // NaN
Math.acos(-2) // NaN
Math.acos(2) // NaN
Math.acos(1) // +0
Math.acos(0) // +π/2
```

### Math.atan(x)

反正切

```javascript
/**
 * x的绝对值为0, 返回x
 * x的绝对值为Infinity, 返回sign(x) * π / 2
 * 其他，返回arctan(x)
 */
if (abs(x) === 0) {
  return x
} else if (abs(x) === Infinity){
  return sign(x) * PI / 2
} else {
  return arctan(x)
}
```

示例

```javascript
Math.atan(NaN) // NaN
Math.atan(+0) // +0
Math.atan(-0) // -0
Math.atan(Infinity) // +π/2
Math.atan(-Infinity) // -π/2
Math.atan(1) // +π/4
```

### Math.atan2(y, x)

象限切线

```javascript
/**
 * x的绝对值为0, y的绝对值不为0, 返回sign(y) * π / 2
 * y的绝对值为0
 *    x > 0 或 x === +0, 返回sign(y) * (+0)
 *    x < 0 或 x === -0, 返回sign(y) * (+π)
 * x的绝对值为Infinity, y的绝对值不为Infinity
 *    x === Infinity, 返回sign(y) * (+0)
 *    x === -Infinity, 返回sign(y) * (+π)
 * x的绝对值不为Infinity, y的绝对值为Infinity，返回sign(y) * π / 2
 * x的绝对值为Infinity, y的绝对值为Infinity
 *    x === Infinity, 返回sign(y) * (+π / 4)
 *    x === -Infinity, 返回sign(y) * (+3 * π / 4)
 * x > 0, 在一、四象限，返回arctan(y / x)
 * x < 0
 *    y > 0, 在第二象限，返回arctan(y / x) + π
 *    y < 0, 在第三象限，返回arctan(y / x) - π
 */
if (abs(x) === 0 && abs(y) !== 0) {
  return sign(y) * PI / 2
} else (abs(y) === 0) {
  if (x > 0 || x === +0) {
    return sign(y) * (+0)
  } else {
    return sign(y) * (+PI)
  }
}

if (abs(x) === Infinity && abs(y) !== Infinity) {
  if (x === Infinity) {
    return sign(y) * (+0)
  } else {
    return sign(y) * (+PI)
  }
} else if (abs(x) !== Infinity && abs(y) === Infinity) {
  return sign(y) * (+PI / 2)
} else if (abs(x) === Infinity && abs(y) === Infinity) {
  if (x === Infinity) {
    return sign(y) * (+PI / 4)
  } else {
    return sign(y) * (+3 * PI / 4)
  }
}

if (x > 0) {
  return arctan(y / x)
} else {
  if (y > 0) {
    return arctan(y / x) + PI
  } else {
    return arctan(y / x) - PI
  }
}
```

示例

```javascript
Math.atan2(NaN, x) // NaN
Math.atan2(1, +0) // +π/2
Math.atan2(-1, -0) // -π/2
Math.atan2(+0, 1) // +0
Math.atan2(-0, 1) // -0
Math.atan2(+0, -1) // +π
Math.atan2(-0, -1) // -π
Math.atan2(1, Infinity) // +0
Math.atan2(-1, Infinity) // -0
Math.atan2(1, -Infinity) // +π
Math.atan2(-1, -Infinity) // -π
Math.atan2(Infinity, 1) // +π/2
Math.atan2(-Infinity, -1) // -π/2
Math.atan2(Infinity, Infinity) // +π/4
Math.atan2(-Infinity, Infinity) // -π/4
Math.atan2(Infinity, -Infinity) // +3π/4
Math.atan2(-Infinity, -Infinity) // -3π/4
Math.atan2(1, 1) // +π/4
Math.atan2(-1, 1) // -π/4
Math.atan2(1, -1) // +3π/4
Math.atan2(-1, -1) // -3π/4
```
