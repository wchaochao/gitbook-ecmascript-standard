# 词法环境

标签（空格分隔）： ECMAScript规范

---

## 环境记录项

记录绑定的标识符

### 声明式环境记录项

将标识符与语言值绑定，如函数上下文、catch语句

#### HasBinding(N)

```javascript
/*
 * 返回是否绑定了指定标识符
 */
let envRec = declarative environment record
return envRec.hasBindName(N)
```

#### CreateMutableBinding(N, D)

```javascript
/*
 * 创建可变绑定
 *     绑定可变
 *     绑定值为undefined
 *     绑定是否可删除由D指定
 */
let envRec = declarative environment record
assert(!envRec.HasBinding(N))
let binding = envRec.createBinding()
binding.mutable = true
binding.value = undefined
binding.deleted = D
```

示例

```javascript
// 函数上下文的形参、函数声明、变量声明创建可变绑定，不可删除
// 非严格模式函数上下文arguments创建可变绑定，不可删除
// 严格模式直接调用的eval上下文的函数声明、变量声明创建可变绑定，可删除
```

#### SetMutableBinding(N,V,S)

```javascript
/*
 * 设置绑定
 *     绑定可变时，设置绑定值
 *     绑定不可变时，由S确定是否抛出TypeError错误
 */
let envRec = declarative environment record
assert(envRec.HasBinding(N))
let binding = envRec.bindings.N
if (binding.mutable) {
  binding.value = V
} else {
  if (S) {
    throw TypeError
  }
}
```

示例

```javascript
// 函数上下文的形参、函数声明、变量声明绑定相应的值
// 非严格模式函数上下文arguments绑定相应的值
// 严格模式直接调用的eval上下文的函数声明、变量声明绑定相应的值
```

#### GetBindingValue(N,S)

```javascript
/*
 * 获取绑定值
 *     绑定不可变且未初始化时，由S确定是否抛出Reference错误或返回undefined
 *     其他情况返回绑定值
 */
let envRec = declarative environment record
assert(envRec.HasBinding(N))
let binding = envRec.bindings.N
if (!binding.mutable && binding.uninitialised) {
  if (S) {
    throw ReferenceError
  } else {
    return undefined
  }
} else {
  return bingind.value
}
```

示例

```javascript
// 获取函数上下文的形参、函数声明、变量声明绑定的值
// 获取非严格模式函数上下文arguments绑定的值
// 获取严格模式直接调用的eval上下文的函数声明、变量声明绑定的值
```

#### DeleteBinding(N)

```javascript
/*
 * 删除绑定
 *     绑定不存在，返回true
 *     绑定存在
 *         绑定不可删除，返回false
 *         绑定可删除，删除绑定并返回true
 */
let envRec = declarative environment record
if (!envRec.HasBinding(N)) {
  return true
} else {
  let binding = envRec.bindings.N
  if (!binding.deleted) {
    return false
  } else {
    envRec.removeBind(N)
    return true
  }
}
```

示例

```javascript
// 函数上下文的形参、arguments、函数声明、变量声明不可删除
// 严格模式直接调用的eval上下文的函数声明、变量声明可删除
```

#### ImplicitThisValue()

```javascript
/*
 * 获取绑定的函数对象的this值
 */
return undefined
```

示例

```javascript
// 直接调用函数上下文的函数声明，this值为undefined
// 直接调用严格模式直接调用的eval上下文的函数声明，this值为undefined
```

#### CreateImmutableBinding(N)

```javascript
/*
 * 创建不可变绑定
 *     绑定不可变
 *     绑定未初始化
 *     绑定不可删除
 */
let envRec = declarative environment record
assert(!envRec.HasBinding(N))
let binding = envRec.createBinding()
binding.mutable = false
binding.uninitialised = true
binding.deleted = false
```

示例

```javascript
// 严格模式函数上下文的arguments创建不可变绑定
```

#### InitializeImmutableBinding(N, V)

```javascript
/*
 * 初始化不可变绑定的值
 */
let envRec = declarative environment record
assert(!envRec.HasBinding(N))
let binding = envRec.createBinding()
assert(!binding.mutable && binding.uninitialised)
binding.value = V
binding.uninitialised = false
```

示例

```javascript
// 严格模式函数上下文的arguments绑定相应的值
```

### 对象式环境记录项

将标识符与对象属性绑定，如全局上下文、with语句

#### HasBinding(N)

```javascript
/*
 * 判断绑定对象是否有指定属性
 */
let envRec = object environment record
let bindings = envRec.bindingObject
return bindings.[[HasProperty]](N)
```

#### CreateMutableBinding(N, D)

```javascript
/*
 * 绑定对象添加属性
 *     [[Value]]为undefined
 *     [[Configurable]]由D指定
 */
let envRec = declarative environment record
let bindings = envRec.bindingObject
assert(!bindings.[[HasProperty]](N))
bindings.[[DefineOwnProperty]](N, {
  [[Value]]: undefined,
  [[Writable]]: true,
  [[Enumerable]]: true,
  [[Configurable]]: ToBoolean(D)
}, true)
```

示例

```javascript
// 全局上下文的函数声明、变量声明创建绑定，不可配置
```

#### SetMutableBinding(N,V,S)

```javascript
/*
 * 绑定对象设置属性
 */
let envRec = declarative environment record
let bindings = envRec.bindingObject
bindings.[[Put]](N, V, S)
```

示例

```javascript
// 全局上下文的函数声明、变量声明绑定相应的值
```

#### GetBindingValue(N,S)

```javascript
/*
 * 获取绑定对象的属性值
 *     属性不存在，由S确定是否抛出Reference错误或返回undefined
 *     属性存在，返回属性值
 */
let envRec = declarative environment record
let bindings = envRec.bindingObject
if (!bindings.[[HasProperty]](N)) {
  if (S) {
    throw ReferenceError
  } else {
    return undefined
  }
} else {
  return bindings.[[Get]](N)
}
```

示例

```javascript
// 获取全局上下文的函数声明、变量声明绑定的值
```

#### DeleteBinding(N)

```javascript
/*
 * 绑定对象删除属性
 */
let envRec = declarative environment record
let bindings = envRec.bindingObject
return bindings.[[Delete]](N, false)
```

示例

```javascript
// 全局上下文的函数声明、变量声明不可删除
```

#### ImplicitThisValue()

```javascript
/*
 * 获取绑定的函数对象的this值
 */
let envRec = declarative environment record
let bindings = envRec.bindingObject
return envRec.provideThis ? bindings : undefined
```

示例

```javascript
// 直接调用全局上下文的函数声明，this值为undefined
// 调用with语句里的函数表达式，this为with的对象
```

## 词法环境

关联环境记录项形成作用域链

### 操作

#### GetIdentifierReference(lex, name, strict)

获取词法环境内的标识符引用

```javascript
/*
 * 词法环境为null，返回base为undefined的引用
 * 词法环境不为null
 *     标识符在词法环境的环境记录项中，返回base为该环境记录项的引用
 *     标识符不在词法环境的环境记录项中，沿外部词法环境递归
 */
if (lex === null) {
  return new Reference({
    base: undefined,
    name,
    strict
  })
}
let envRec = lex.environmentRecord
let exists = envRec.HasBinding(N)
if (exists) {
  return new Reference({
    base: envRec,
    name,
    strict
  })
} else {
  let outer = lex.outerEnvironmentReference
  retrun GetIdentifierReference(outer, name, strict)
}
```

示例

```javascript
// 未声明的标识符
console.log(typeof a)

Reference {
  base: undefined,
  name: 'a',
  strict: false
}

// 全局上下文的标识符
var a = 1
console.log(a)

Reference {
  base: global Environment Record,
  name: 'a',
  strict: false
}

// 函数上下文的标识符
function foo (a) {
  console.log(a)
  
  Reference {
    base: function Environment Record,
    name: 'a',
    strict: false
  }
}
```

#### NewDeclarativeEnvironment(E)

创建声明式词法环境

```javascript
/*
 * 创建词法环境
 *     环境记录项为声明式环境记录项
 *     外部词法环境为E
 */
let env = new LexicalEnvironment()
env.environmentRecord = new DeclarativeEnvironmentRecord()
env.outerEnvironmentReference = E
return env
```

示例

```javascript
// 调用函数，生成函数词法环境
NewDeclarativeEnvironment(F.[[Scope]])

// 严格模式直接调用eval，生成eval词法环境
NewDeclarativeEnvironment(callingContext.LexicalEnvironment)
```

#### NewObjectEnvironment(O, E)

创建对象式词法环境

```javascript
/*
 * 创建词法环境
 *     环境记录项为对象式环境记录项，绑定对象为O
 *     外部词法环境为E
 */
let env = new LexicalEnvironment()
env.environmentRecord = new ObjectEnvironmentRecord({bindingObject: O})
env.outerEnvironmentReference = E
return env
```

示例

```javascript
// 全局词法环境
NewObjectEnvironment(global, null)

// 执行with语句，生成对象词法环境
NewObjectEnvironment(o, callingContext.LexicalEnvironment)
```
