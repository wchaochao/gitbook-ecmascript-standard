# 程序

标签（空格分隔）： ECMAScript规范

---

## 语法

```
Program :
  SourceElements(opt) // 源代码

SourceElements : // 源代码
  SourceElement // 单个源代码元素
  SourceElements SourceElement // 多个源代码元素

SourceElement : // 单个源代码元素
  Statement // 语句
  FunctionDeclaration // 函数声明
```

### 源代码

```javascript
Program : SourceElements(opt)

/**
 * 开头有严格模式指令，则为严格模式
 * 无SourceElements，直接返回Completion(normal, empty, empty)
 * 有SourceElements
 *     将全局上下文压入上下文堆栈并初始化
 *     执行SourceElements，退出全局上下文，返回SourceElements结果
 */
if (beginWithUserStrictDirective) {
  let strict = true
}

if (!IsPresent(SourceElements)) {
  return Completion(normal, empty, empty)
}

let progCxt = GlobalEnvironment
let result = SourceElements
progCxt.exit()
return result
```

### 单个源代码元素

```javascript
SourceElements : SourceElement
```

#### 语句

```javascript
SourceElement : Statement

/**
 * 执行Statement，返回Statement结果
 */
let s = Statement
return s
```

#### 函数声明

```javascript
SourceElement : FunctionDeclaration

/**
 * 返回Completion(normal, empty, empty)
 */
return Completion(normal, empty, empty)
```

### 多个源代码元素

```javascript
SourceElements : SourceElements SourceElement

/**
 * 执行SourceElements
 * 结果为abrupt completion，直接返回SourceElements结果
 * 结果不为abrupt completion，执行SourceElement，返回Completion(tailResult.type, tailResult.value || tailResult.value, tailResult.target)
 */
let headResult = SourceElements
if (headResult.type !== normal) {
  return headResult
}
let tailResult = SourceElement
let V = tailResult.value === empty ? headResult.value : tailResult.value
return Completion(tailResult.type, V, tailResult.target)
```
