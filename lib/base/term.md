# 术语

标签（空格分隔）： ECMAScript规范

---

## 严格模式

为容易出错的地方施加限制

### 全局中

```javascript
"use strict"; // 在代码中间时不起作用

// statement
```

### 函数中

```javascript
function foo(){
  "use strict"; // 在函数代码中间时不起作用

  // statement
}
```

### eval中

```javascript
eval('"use strict"; var a = 2;')
```

## 数据类型

* 原始值类型：`Undefined, Null, Number, String, Boolean`，存储值
* 引用类型：`Object`，存储对象的引用

```javascript
// 存储值
var a = 1;
var b = 1;
console.log(a === b); // true

// 存储引用
var c = {}
var d = {}
console.log(c === d); // false
```

## 运算符

进行某种运算

## 表达式

由数据和运算符组成，进行某种运算，返回运算结果

## 语句

执行某个操作

* 空语句：`;`
* 单语句：以分号结尾，不加分号时按规则自动插入分号
* 多语句：组合到BlockStatement中

## 函数

封装任意条语句，实现某个功能

## 对象

属性的集合，所有数据都可以视为对象

### Object对象

普通对象

### Function对象

函数，可调用的对象

### 数组

按次序排列的一组值

* 数组长度和数组元素动态关联

### 类数组对象

具有`length`属性的对象，可通过`call(), apply()`调用数组的一些方法

* String对象
* arguments对象
* DOM元素集合

### 基本包装对象

`Number, String, Boolean`类型对应的包装对象

* `Number, String, Boolean`类型当作对象使用时，会被包装成相应包装对象，使用完后销毁该对象
  * `Number, String, Boolean`类型当作对象使用时，相应的包装对象是只读的，无法设置属性
  * `Number`字面量当作对象使用时，其后的点会被解释成小数点，需要使用圆括号括起来
* `Undefined, Null`类型没有相应的包装对象，不能当作对象使用，会抛出`TypeError`

```javascript
'1'.toSting() // '1'
(1).toString() // '1'
'1'.toString = 2 // 严格模式抛出TypeError
null.a // 抛出TypeError
```

### Date对象

用于处理日期和时间

* 使用自1970年1月1日00:00:00(UTC时间)开始经过的毫秒数来保存日期，范围为该时间的前后1亿天
* UTC时间：世界协调时间，UTC时间 = GMT时间
* 本地时间：本地时区时间，UTC时间 + 时区 = 本地时间

### RegExp对象

正则表达式，用于模式匹配

### Error对象

用于错误处理

### 单体内置对象

无须实例化，直接提供的对象

* Global对象：提供全局属性和全局方法
* Math对象：用于数学运算
* JSON对象：用于解析和转换JSON字符串

## 构造器

创建和初始化对象的构造函数

## 原型

用来实现继承和共享属性的对象

* 使用`new`创建对象时原型为`constructor.prototype`
* 使用`Object.create()`方法创建对象时原型为指定的原型
