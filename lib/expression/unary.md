# 一元表达式

标签（空格分隔）： ECMAScript规范

---

```
UnaryExpression :
  PostfixExpression // 后缀表达式
  delete UnaryExpression // delete
  void UnaryExpression // void
  typeof UnaryExpression // typeof
  ++ UnaryExpression // 前置自增
  -- UnaryExpression // 前置自减
  + UnaryExpression // 取正
  - UnaryExpression // 取负
  ~ UnaryExpression // 按位取反
  ! UnaryExpression // 逻辑取反
```

## delete

```javascript
UnaryExpression : delete UnaryExpression

/*
 * UnaryExpression不为引用类型，直接返回true
 * UnaryExpression为引用类型
 *     Unresolvable引用
 *         严格模式，抛出SyntaxError
 *         非严格模式，返回true
 *     属性引用，将base value转换为对象，调用对象的[[Delete]]方法
 *     环境引用
 *         严格模式，抛出SyntaxError
 *         非严格模式，调用base value的DeleteBinding方法
 */
let ref = UnaryExpression
if (Type(ref) !== Reference) {
  return true
}

if (IsUnresolvableReference(ref)) {
  if (IsStrictReference(ref)) {
    throw SyntaxError
  } else {
    return true
  }
}

if (IsPropertyReference(ref)) {
  let o = ToObject(GetBase(ref))
  return o.[[Delete]](GetReferenceName(ref), IsStrictReference(ref))
} else {
  if (IsStrictReference(ref)) {
    throw SyntaxError
  } else {
    let base = GetBase(ref)
    return base.DeleteBinding(GetReferenceName(ref))
  }
}
```

示例

```javascript
// 非引用类型
delete 1 // true

// Unresolvable引用
delete a // 严格模式抛出SyntaxError, 非严格模式返回true

// 属性引用
let o = {a: 1}
delete o.b // true
delete o.a // true

delete 'a'.length // 严格模式抛出TypeError, 非严格模式返回false

// 环境引用
let a = 1
delete a // 严格模式抛出SyntaxError, 非严格模式返回false
```

## void

```javascript
UnaryExpression : void UnaryExpression

/*
 * 获取UnaryExpression值并返回undefined
 */
GetValue(UnaryExpression)
return undefined
```

示例

```javascript
let a = 1
void(a++) // 返回undefined, a = 2
```

## typeof

```javascript
UnaryExpression : typeof UnaryExpression

/*
 * UnaryExpression为Unresolvalbe引用，返回'undefined'
 * 获取UnaryExpression值，判断值的类型
 *     类型为Undefined, 返回'undefined'
 *     类型为Null, 返回'object'
 *     类型为Boolean, 返回'boolean'
 *     类型为Number, 返回'number'
 *     类型为String, 返回'string'
 *     类型为Object, 函数返回'function', 其他返回'object'
 */
let val = UnaryExpression
if (Type(val) === Reference) {
  if (IsUnresolvableReference(val)) {
    return undefined
  } else {
    val = GetValue(val)
  }
}

switch (Type(val)) {
  case Undefined:
    return 'undefined'
  case Null:
    return 'object'
  case Boolean:
    return 'boolean'
  case Number:
    return 'number'
  case String:
    return 'string'
  case Object:
    if (IsCallable(val)) {
      return 'function'
    } else {
      return 'object'
    }
}
```

| Type of val | Result |
| --- | --- |
| Undefined | 'undefined' |
| Null | 'object' |
| Boolean | 'boolean' |
| Number | 'number' |
| String | 'string' |
| Object (native and does not implement [[Call]]) | 'object' |
| Object (native or host and does implement [[Call]]) | 'function' |


示例

```javascript
typeof a // 'undefined'
typeof undefined // 'undefined'
typeof null // 'object'
typeof true // 'boolean'
typeof 1 // 'number'
typeof 'a' // 'string'
typeof /a/i // 'object'
function foo () {}
typeof foo // 'function'
```

## 前置自增

```javascript
UnaryExpression : ++ UnaryExpression

/*
 * UnaryExpression为引用类型
 *     该引用为环境引用，且引用名为eval或arguments，且为严格模式时，抛出SyntaxError
 *     其他情况设置引用值为原引用值+1，并返回新引用值
 */
let expr = UnaryExpression

if (Type(expr) === Reference && Type(GetBase(expr)) === Environment Record && (GetReferenceName(expr) === 'eval' || GetReferenceName(expr) === 'arguments') && IsStrictReference(expr) = true) {
  throw SyntaxError
}

let oldValue = ToNumber(GetValue(expr))
let newValue = oldValue + 1
PutValue(expr, newValue)
return newValue
```

示例

```javascript
let a = 1
++a + ++a // 2 + 3, a = 3
```

## 前置自减

```javascript
UnaryExpression : -- UnaryExpression

/*
 * UnaryExpression为引用类型
 *     该引用为环境引用，且引用名为eval或arguments，且为严格模式时，抛出SyntaxError
 *     其他情况设置引用值为原引用值-1，并返回新引用值
 */
let expr = UnaryExpression

if (Type(expr) === Reference && Type(GetBase(expr)) === Environment Record && (GetReferenceName(expr) === 'eval' || GetReferenceName(expr) === 'arguments') && IsStrictReference(expr) = true) {
  throw SyntaxError
}

let oldValue = ToNumber(GetValue(expr))
let newValue = oldValue - 1
PutValue(expr, newValue)
return newValue
```

示例

```javascript
let a = 1
--a + --a // 0 + (-1), a = -1
```

## 取正

```javascript
UnaryExpression : + UnaryExpression

/*
 * 获取UnaryExpression的值并转换为Number类型
 */
let value = GetValue(UnaryExpression)
return ToNumber(value)
```

示例

```javascript
+undefined // NaN
+null // +0
+true // 1
+false // 0
+' \n ' // 0
+' 0123.45 ' // 123.45
+' 0x12 ' // 18
+({}) // NaN
```

## 取负

```javascript
UnaryExpression : - UnaryExpression

/*
 * 获取UnaryExpression的值并转换为Number类型，再取负值
 */
let expr = UnaryExpression
let oldValue = ToNumber(GetValue(expr))
if (SameValue(oldValue, NaN)) {
  return NaN
} else {
  return negative(oldValue)
}
```

示例

```javascript
-undefined // NaN
-null // -0
-true // -1
-false // -0
-' \n ' // -0
-' 0123.45 ' // -123.45
-' 0x12 ' // -18
-({}) // NaN
```

## 按位取反

```javascript
UnaryExpression : ~ UnaryExpression

/*
 * 获取UnaryExpression的值并转换为Int32类型
 * 按位取反，结果为Int32类型
 */
let expr = UnaryExpression
let oldValue = ToInt32(GetValue(expr))
return bitwiseComplement(oldValue)
```

示例

```javascript
~NaN // -1
~Infinity // -1
~-Infinity // -1
~+0 // -1
~-0 // -1
~15 // -16
~-15 // 16
~(Math.pow(2, 31) + 1) // 2^31 - 2
~(-Math.pow(2, 31) - 1) // -2^31
```

## 逻辑取反

```javascript
UnaryExpression : ! UnaryExpression

/*
 * 获取UnaryExpression的值并转换为Boolean类型，再取反
 */
let expr = UnaryExpression
let oldValue = ToBoolean(GetValue(expr))
if (oldValue === true) {
  return false
} else {
  return true
}
```

示例

```javascript
!undefined // true
!null // true
!0 // true
!NaN // true
!'' // true
!({}) // false
```
