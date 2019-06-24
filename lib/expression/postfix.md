# 后缀表达式

标签（空格分隔）： ECMAScript规范

---

```
PostfixExpression : // 后缀表达式
  LeftHandSideExpression // 左值表达式
  LeftHandSideExpression [no LineTerminator here] ++ // 后缀自增
  LeftHandSideExpression [no LineTerminator here] -- // 后缀自减
```

## 后缀自增

```javascript
PostfixExpression : LeftHandSideExpression [no LineTerminator here] ++

/*
 * LeftHandSideExpression为引用类型
 *     该引用为环境引用，且引用名为eval或arguments，且为严格模式时，抛出SyntaxError
 *     其他情况设置引用值为原引用值+1，并返回原引用值
 */
let lhs = LeftHandSideExpression

if (Type(lhs) === Reference && Type(GetBase(lhs)) === Environment Record && (GetReferenceName(lhs) === 'eval' || GetReferenceName(lhs) === 'arguments') && IsStrictReference(lhs) = true) {
  throw SyntaxError
}

let oldValue = ToNumber(GetValue(lhs))
let newValue = oldValue + 1
PutValue(lhs, newValue)
return oldValue
```

示例

```javascript
let a = 1
a++ + a++ // 1 + 2, a = 3
```

## 后缀自减

```javascript
PostfixExpression : LeftHandSideExpression [no LineTerminator here] --

/*
 * LeftHandSideExpression为引用类型
 *     该引用为环境引用，且引用名为eval或arguments，且为严格模式时，抛出SyntaxError
 *     其他情况设置引用值为原引用值-1，并返回原引用值
 */
let lhs = LeftHandSideExpression

if (Type(lhs) === Reference && Type(GetBase(lhs)) === Environment Record && (GetReferenceName(lhs) === 'eval' || GetReferenceName(lhs) === 'arguments') && IsStrictReference(lhs) = true) {
  throw SyntaxError
}

let oldValue = ToNumber(GetValue(lhs))
let newValue = oldValue - 1
PutValue(lhs, newValue)
return oldValue
```

示例

```javascript
let a = 1
a-- + a-- // 1 + 0, a = -1
```
