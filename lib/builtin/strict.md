# 严格模式

标签（空格分隔）： ECMAScript规范

---

严格模式下的限制

## 标识符

* FutureReservedWord不能为标识符
* eval、arguemnts不能为函数名、形参名、catch中错误名

```
FutureReservedWord :: one of
  class          enum       extends      super
  const          export   import
  implements  let       private      public    yield
  interface      package  protected  static
```

示例

```javascript
(function () {
  'use strict'
  
  let implements // Uncaught SyntaxError: Unexpected strict mode reserved word

  // SyntaxError: Unexpected eval or arguments in strict mode
  function foo(arguments) {} 
  function arguments() {}
  try {
  } catch (eval) {
  }
})()
```

## 数据

* 不能使用八进制数字
* 不能使用八进制转义系列

示例

```javascript
(function () {
  'use strict'
  
  let a = 012 // Uncaught SyntaxError: Octal literals are not allowed in strict mode.
  
  let b = '\123' // Uncaught SyntaxError: Octal escape sequences are not allowed in strict mode.
  
})()
```

## 表达式

* 变量未声明时不能赋值
* eval、argument不能出现在赋值左侧和递增、递减表达式中
* this不自动转换
* delete不能删除声明变量、声明函数

示例

```javascript
(function () {
  'use strict'
  
  x = 1 // Uncaught ReferenceError: x is not defined.
  
  eval = 1 // Uncaught SyntaxError: Unexpected eval or arguments in strict mode
  arguments++ // Uncaught SyntaxError: Unexpected eval or arguments in strict mode
  
  function fun() {
    return this;
  }

  fun() //undefined
  fun.call(2) // 2
  
  let a
  delete a // Uncaught SyntaxError: Delete of an unqualified identifier in strict mode.
})()
```

## 语句

* 禁止使用with语句
* 不在块中声明函数

示例

```javascript
(function () {
  'use strict'
  
  with(o) { // Uncaught SyntaxError: Strict mode code may not include a with statement
    v = 2
  }
  
  if (true) {
    function f1() {}
  }
  
})()
```

## 函数

* 形参列表不能重复
* 不能访问F.arguments、F.caller、arguments.caller、arguements.callee
* arguments对象与形参不再共享值

示例

```javascript
(function () {
  'use strict'
  
  // SyntaxError: Duplicate parameter name not allowed in this context
  function foo(a, a, b) {
    return a + b;
  }
  
  // Uncaught TypeError: 'caller', 'callee', and 'arguments' properties may not be accessed on strict mode functions or the arguments objects for calls to them  
  function f1() {
    f1.caller

    f1.arguments

    arguments.callee
  }
  f1()
  
  function f(a) {
    a = 2
    return [a, arguments[0]];
  }
  f(1); // [2, 1]
  
})()
```

## 对象

* 只读属性不能修改
* 不可扩展对象不能添加属性
* 不可配置属性不能删除
* 原始值类型不能设置属性

```javascript
(function () {
  'use strict'
  
  let obj = {}
  Object.defineProperty(obj, "x", {
    value: 1
  })
  obj.x = 2; // TypeError: Cannot assign to read only property 'x' of object '#<Object>'

  Object.defineProperty(obj, "y", {
    get: function () {
      return 1
    }
  })
  obj.y = 2 // TypeError: Cannot set property x of #<Object> which has only a getter
  
  Object.preventExtensions(o)
  o.z = 3 // Uncaught TypeError: Cannot add property z, object is not extensible
  
  delete obj.x // TypeError: Cannot delete property 'x' of #<Object>
  
  "sty".you = 1 // TypeError: Cannot create property 'you' on string
  
})()
```

## eval

调用eval时创建eval作用域

```javascript
(function () {
  'use strict'

  var x = 2

  console.log(eval('var x = 5; x')) // 5
  console.log(x) // 2

})()
```
