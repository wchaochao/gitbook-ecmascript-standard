# 执行上下文

标签（空格分隔）： ECMAScript规范

---

## 可执行代码

* `global code`: 全局代码，作为 ECMAScript 程序处理的源代码文本
* `function code`: 函数代码，函数体对应的源代码文本
* `eval code`: eval代码，提供给 eval 内置函数的源代码文本

### 严格模式

* 全局代码严格模式：以Use Strict Directive开头
* 函数代码严格模式
  * 函数声明、函数表达式以Use Strict Directive开头或被包含在严格模式代码中
  * new Function创建的函数以Use Strict Directive开头
* eval代码严格模式：以Use Strict Directive开头或被严格模式的代码直接调用

## 创建

当程序进入可执行代码时会创建对应的执行上下文，并被压入上下文堆栈中

* 执行全局代码，创建全局上下文
* 执行函数代码，创建函数上下文
* 执行eval代码，创建eval上下文

## 组成

| 组成 | 类型 | 描述 |
| --- | --- | --- |
| ThisBinding | any | 执行上下文的this值 |
| VariableEnvironment | Lexical Environment | 变量环境，用于声明绑定，在上下文执行过程中绑定的值不会发生变化 |
| LexicalEnvironment | Lexical Environment | 词法环境，用于解析执行上下文中的标识符引用，在上下文执行过程中绑定的值会发生变化 |

## 过程

* 初始化
* 声明绑定
* 执行

### 初始化

#### 全局上下文初始化

```javascript
/*
 * this值为全局对象
 * 变量环境和词法环境为全局环境
 */
context.ThisBinding = global
context.VariableEnvironment = Global Environment
context.LexicalEnvironment = Global Environment
```

示例

```javascript
// 全局上下文
{
  ThisBinding: global,
  LexicalEnvironment: NewObjectEnvironment(global, null)
}
```

#### 函数上下文初始化

```javascript
/*
 * this值由F.thisArg决定
 *      严格模式，this为thisArg
 *      非严格模式
 *          thisArg不能转换为Object时，this为全局对象
 *          thisArg能转换为Object时，this为ToObject(thisArg)
 * 变量环境、词法环境为F.[[Scope]]生成的声明式词法环境
 */
let thisArg = F.thisArg
if (strict) {
  context.ThisBinding = thisArg
} else {
  if (thisArg === undefined || thisArg === null) {
    context.ThisBinding = global
  } else {
    context.ThisBinding = ToObject(thisArg)
  }
}

let localEnv = NewDeclarativeEnvironment(F.[[Scope]])
context.VariableEnvironment = localEnv
context.LexicalEnvironment = localEnv
```

示例

```javascript
// 严格模式函数上下文
{
  ThisBinding: F.thisArg,
  LexicalEnvironment: NewDeclarativeEnvironment(F.[[Scope]])
}
```

#### eval上下文初始化

```javascript
/*
 * 无调用上下文或非直接调用
 *     this值为全局对象
 *     变量环境和词法环境为全局环境
 * 有调用上下文或直接调用
 *     非严格模式，this、变量环境、词法环境为调用上下文的值
 *     严格模式，this为调用上下文的值，变量环境、词法环境为调用上下文词法环境生成的声明式词法环境
 */
if (!callingContext || !directCall) {
  context.ThisBinding = global
  context.VariableEnvironment = Global Environment
  context.LexicalEnvironment = Global Environment
} else {
  if (!strict) {
    context.ThisBinding = callingContext.ThisBinding
    context.VariableEnvironment = callingContext.VariableEnvironment
    context.LexicalEnvironment = callingContext.LexicalEnvironment
  } else {
    context.ThisBinding = callingContext.ThisBinding
    let strictVarEnv = NewDeclarativeEnvironment(callingContext.LexicalEnvironment)
    context.VariableEnvironment = strictVarEnv
    context.LexicalEnvironment = strictVarEnv
  }
}
```

示例

```javascript
// 严格模式直接调用eval
{
  ThisBinding: callingContext.ThisBinding,
  LexicalEnvironment: NewDeclarativeEnvironment(callingContext.LexicalEnvironment)
}
```

### 声明绑定

变量声明、函数声明、参数列表会被绑定到VariableEnvironment里

```javascript
/*
 * 形参绑定
 *     未绑定时先创建可变绑定，设置绑定值为对应的实参或undefined
 *     已绑定时修改绑定值为对应的实参或undefined
 * arguments绑定
 *     未绑定时创建绑定
 *         严格模式创建不可变绑定，初始化绑定值为argsObj
 *         非严格模式创建可变绑定，设置绑定值为argsObj
 *     已绑定时不做处理
 * 函数声明绑定
 *     未绑定时创建可变绑定，设置绑定值为声明的函数值
 *     已绑定时需先判断能否修改
 *         全局环境属性不可配置且为访问器属性或[[Writable] !== true && existingProp.[[Enumerable] !== true时不能修改，抛出TypeError
 *         其他情况修改绑定值为声明的函数值
 * 变量声明绑定
 *     未绑定时创建可变绑定，设置绑定值为undefined
 *     已绑定时不做处理
 * eval代码的声明绑定可配置
 */
let envRec = context.VariableEnvironment.environmentRecord
let configurableBindings = code === eval code
let strict = code === strict mode code

if (code === function code) {
  let names = func.[[FormalParameters]]
  for (var i = 0; i < names.length; i++) {
    let argName = names[i]
    if (!envRec.HasBinding(argName)) {
      envRec.CreateMutableBinding(argName)
    }
    let v = i > args.length - 1 ? undefined : args[i]
    envRec.SetMutableBinding(argName, v, strict)
  }
  
  if (!envRec.HasBinding('arguments')) {
    let argsObj = CreateArgumentsObject(func, names, args, envRec, strict)
    if (strict) {
      envRec.CreateImmutableBinding('arguments')
      envRec.InitializeImmutableBinding('arguments', argsObj)
    } else {
      envRec.CreateMutableBinding('arguments')
      envRec.SetMutableBinding('arguments', argsObj, false)
    }
  }
}

forEach(FunctionDeclaration f in code) {
  let fn = f.Identifier
  let fo = f.value
  if (!envRec.HasBinding(fn)) {
    envRec.CreateMutableBinding(argName, configurableBindings)
  } else if (envRec === GlobalEnvironment.environmentRecord) {
    let go = global
    let existingProp = go.[[GetProperty]](fn)
    if (existingProp.[[Configurable]]) {
      go.[[DefineOwnProperty]](fn, {
        [[Value]]: undefined,
        [[Writable]]: true,
        [[Enumerable]]: true,
        [[Configurable]]: configurableBindings 
      }, true)
    } else if (IsAccessorDescriptor(existingProp) || (existingProp.[[Writable] !== true && existingProp.[[Enumerable] !== true)) {
      throw TypeError
    }
  }
  envRec.SetMutableBinding(fn, fo, strict)
}

forEach(VariableDeclaration d in code) {
  let dn = d.Identifier
  if (!envRec.HasBinding(dn)) {
    envRec.CreateMutableBinding(dn, configurableBindings)
    envRec.SetMutableBinding(dn, undefined, strict)
  }
}
```

示例

```javascript
let n = 1
function foo (a, b, c) {
  let s = 's'
  function bar (m, n) {}
}
foo(1, 2)

// 初始化
// 全局上下文环境记录项
{
  foo: foo,
  n: undefined
}

// 函数上下文环境记录项
{
  a: 1,
  b: 2,
  c: undefined,
  arguments: {
    0: 1,
    1: 2,
    length: 2
  },
  bar: bar,
  s: undefined
}
```

#### CreateArgumentsObject(func, names, args, envRec, strict)

创建Arguments对象

```javascript
/*
 * 创建Arguments对象
 * [[Class]]属性为'Arguments'
 * [[Prototype]]属性为Object.prototype
 * length属性为实参个数
 * <index>属性为对应实参，非严格模式下与对应的形参共享值
 * [[ParameterMap]]属性为共享的形参
 * 非严格模式下，callee属性为func
 * 严格模式下，访问callee、caller属性抛出TypeError
 */
let obj = new Object()
obj.[[Class]] = 'Arguments'
obj.[[Prototype]] = Object.prototype

let len = args.length
obj.[[DefineOwnProperty]]('length', {
  [[Value]]: len,
  [[Writable]]: true,
  [[Enumerable]]: false,
  [[Configurable]]: true
}, false)

let map = new Object() // 共享的形参
let mappedNames = new List() // 共享的形参名
let index = len - 1
while (index >= 0) {
  let val = args[index]
  obj.[[DefineOwnProperty]](ToString(index), {
    [[Value]]: val,
    [[Writable]]: true,
    [[Enumerable]]: true,
    [[Configurable]]: true
  }, false)
 
  if (index <= names.length - 1) { // 填充map
    let name = names[index]
    if (!strict && !mappedNames.has(name)) {
      mappedNames.add(name)
      let g = createFunction(new List(), 'return name', env, true)
      let p = createFunction(new List(val), 'name = val', env, true)
      map.[[DefineOwnProperty]](ToString(index), {
        [[Get]]: g,
        [[Set]]: p,
        [[Configurable]]: true
      }, false)
    }
  }
}

// 有共享形参，重写arguments对象的[[Get]]、[[GetOwnProperty]]、[[DefineOwnProperty]]、[[Delete]]方法
if (mappedNames.length) {
  obj.[[ParameterMap]] = map

  obj.defaultGet = obj.[[Get]]
  obj.[[Get]] = function (P) {
    let map = obj.[[ParameterMap]]
    let isMapped = map.[[GetOwnProperty]](P)
    if (isMapped === undefined) {
      return obj.defaultGet(P)
    } else {
      return map.[[Get]](P)
    }
  }
  
  obj.defaultGetOwnProperty = obj.[[GetOwnProperty]]
  obj.[[GetOwnProperty]] = function (P) {
    let desc = obj.defaultGetOwnProperty(P)
    if (desc === undefined) {
      return undefined
    }
    
    let map = obj.[[ParameterMap]]
    let isMapped = map.[[GetOwnProperty]](P)
    if (isMapped !== undefined) {
      desc.[[Value]] = map.[[Get]](P)
    }
    
    return desc
  }
  
  obj.defaultDefineOwnProperty = obj.[[DefineOwnProperty]]
  obj.[[DefineOwnProperty]] = function (P, Desc, Throw) {
    let allowed = obj.defaultDefineOwnProperty(P, Desc, false)
    if (allowed === false) {
      if (Throw) {
        throw TypeError
      } else {
        return false
      }
    }
    
    let map = obj.[[ParameterMap]]
    let isMapped = map.[[GetOwnProperty]](P)
    if (isMapped !== undefined) {
      if (IsAccessorDescriptor(Desc)) {
        map.[[Delete]](P, false)
      } else {
        if (Desc.[[Value]] !== empty) {
          map.[[Put]](P, Desc.[[Value]], false)
        }
        if (Desc.[[Writable]] === false) {
          map.[[Delete]](P, false)
        }
      }
    }
    
    return true
  }
 
  obj.defaultDelete = obj.[[Delete]]
  obj.[[Delete]] = function (P, Throw) {
    let result = obj.defaultDelete(P, Throw)
    let map = obj.[[ParameterMap]]
    let isMapped = map.[[GetOwnProperty]](P)
    if (result === true && isMapped !== undefined) {
      map.[[Delete]](P, false)
    }
    return result
  }
}

// callee、caller属性
if (!strict) {
  obj.[[DefineOwnProperty]]('callee', {
    [[Value]]: func,
    [[Writable]]: true,
    [[Enumerable]]: false,
    [[Configurable]]: true
  }, false)
} else {
  let thrower = [[ThrowTypeError]]
  obj.[[DefineOwnProperty]]('caller', {
    [[Get]]: thrower,
    [[Set]]: thrower,
    [[Enumerable]]: false,
    [[Configurable]]: false
  }, false)
  obj.[[DefineOwnProperty]]('callee', {
    [[Get]]: thrower,
    [[Set]]: thrower,
    [[Enumerable]]: false,
    [[Configurable]]: false
  }, false)
}
```

示例

```javascript
// 严格模式
function foo (a, b) {
  'use strict'
}
foo(1, 2, 3)

// arguments
{
  [[Class]]: 'arguments',
  [[Prototype]]: Object.prototype,
  length: 3,
  0: 1,
  1: 2,
  2: 3,
  [[ParameterMap]]: {},
  caller: [[ThrowTypeError]],
  callee: [[ThrowTypeError]]
}

// 非严格模式
function foo (a, b) {
}
foo(1, 2, 3)

// arguments
{
  [[Class]]: 'arguments',
  [[Prototype]]: Object.prototype,
  length: 3,
  0: 1,
  1: 2,
  2: 3,
  [[ParameterMap]]: {
    0: map(a),
    1: map(b)
  },
  callee: foo
}
```

## 退出

* return退出一个上下文
* 抛出错误退出一个或多个上下文

示例

```javascript
// return 退出上下文
function foo () {
  return 1
}

// throw 退出上下文
function foo () {
  throw new Error('error')
}
```
