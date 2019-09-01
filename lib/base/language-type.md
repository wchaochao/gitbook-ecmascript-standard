# 语言类型

标签（空格分隔）： ECMAScript规范

---

直接操作的值类型

* `Undefined`
* `Null`
* `Boolean`
* `String`
* `Number`
* `Object`

## Undefined类型

未定义

### 类型值

```
undefined
```

示例

```javascript
// 变量被声明但未赋值
let i // undefined

// 不存在的对象属性
let o = {}
console.log(o.p) // undefined

// 没有实参传入的形参
function f(x) {
  console.log(x)
}
f() // undefined

// 函数无返回值, 默认返回undefined
function f() {}
var a = f() // undefined
```

## Null类型

空值

### 类型值

```
null
```

## Boolean类型

布尔值

### 类型值

```
true
false
```

## String类型

字符串

### 类型值

16 位无符号整数值的有序序列

* 每个元素占有一个序列位置，索引从0开始
* 元素的个数为字符串长度，最多2^53 - 1个

Unicode字符转换为String类型值

* 使用`UTF-16`编码
 * 单字节字符，无影响
 * 多字节字符，需要用两个元素来表示，高位为[0xD800, 0xDBFF]，低位为[0xDC00, 0xDFFF]

## Number类型

数值

### 类型值

IEEE-754 格式 64 位双精度数值

#### 组成

1位数符 + 11位阶码 + 52位尾数

* 数符S：表示正负，0为正，1为负
* 阶码E：指数部分，阶码 = 阶码真值 + 偏移量1023
* 尾数：小数部分

示例

```
十进制 -125.125
转换为二进制表示：-1111101.001
科学计数法表示：-1.111101001 * 2^6

数符S：1
阶码E：000 00000110 (阶码真值) + 011 11111111 (偏移量) = 1  00 00000101
尾数M：11110100 10000000 00000000 00000000 00000000 00000000 0000
最终值：1 + 100 00000101 + 11110100 10000000 00000000 00000000 00000000 00000000 0000
```

#### 表示值

* 有限值：2^64 - 2^53个，一半正数、一半负数
* 无限值：Infinity、-Infinity、NaN

```
V = (-1)^S*2^(E-1023)*1.M

规格化（M省掉了小数点前的1）
S  +  (E!=0 && E!=2047) + 1.M
  E为2046且M全为1时为能表示的最大的数，用Number.MAX_VALUE表示，大于该数时为无穷大
  E为1023 ~ 1023 + 52时为整数，范围为[Number.MIN_SAFE_INTEGER, Number.MAX_SAFE_INTEGER]
  M为52位，总共53为有效数字，最小精度为Number.EPSILON

非规格化（M未省掉小数点前的1）
S  +  (E=0) + M
  M全为0时表示0
     S为正，表示+0
     S为负，表示-0
  M不全为0时表示接近0的数
     M为1时为能表示的最接近0的数，用Number.MIN_VALUE表示，小于该数时为0
S  +  (E=2047) + (M全为0)     
  无穷大
     S为正，表示Infinity
     S为负，表示-Infinity
S  +  (E=2047) + (M不全为0)
  NaN
```

精确数字 x 对应Number类型值

* 选择Number类型值中最接近x的值
 * 只有一个，就是该值
 * 有两个，选择M为偶数的
 * 选择值为2^1024时，为正无穷
 * 选择值为-2^1024时，为负无穷
 * 选择值为+0且x大于0时，为正零
 * 选择值为+0且x小于0时，为负零

可能会存在误差

```javascript
0.1 + 0.2 === 0.3 // false

0.3 / 0.1 // 2.9999999999999996

(0.3 - 0.2) === (0.2 - 0.1) // false
```

## Object类型

对象

### 类型值

各类对象

* 内置对象：ECMAScript里内置的对象，如`Object, Function, Array, Number, String, Boolean, Date, RegExp, Error`等构造函数，`Global, Math, JSON`等单体对象
* 原生对象：通过ECMAScript创建的对象
* 宿主对象：由宿主环境提供的对象，用于完善执行环境，如浏览器提供的`window, document`等对象
