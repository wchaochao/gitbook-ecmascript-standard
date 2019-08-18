# JSON对象

标签（空格分隔）： ECMAScript规范

---

## 词法

### JSONWhiteSpace

```
JSONWhiteSpace :: // JSON空白字符
  <SP> // 空格
  <TAB> // Tab
  <CR> // 回车
  <LF> // 换行
```

### JSONNullLiteral

```
JSONNullLiteral :: // JSON Null字面量
  NullLiteral
```

示例

```javascript
null
```

### JSONBooleanLiteral

```javascript
JSONBooleanLiteral :: // JSON Boolean字面量
  BooleanLiteral
```

示例

```javascript
true
false
```

### JSONNumber

```
JSONNumber :: // JSON数字
  -(opt) DecimalIntegerLiteral JSONFraction(opt) ExponentPart(opt)

JSONFraction :: // JSON小数部分
  . DecimalDigits
```

示例

```javascript
1
1.2
1e+2
1.2e+2
-1.2e+2
```

### JSONString 

```
JSONString :: // JSON字符串
  "JSONStringCharacters(opt)"

JSONStringCharacters :: // 多个JSON字符
  JSONStringCharacter JSONStringCharacters(opt)

JSONStringCharacter :: // 单个JSON字符
  SourceCharacter but not one of " or \ or U+0000 through U+001F // 普通JSON字符
  \ JSONEscapeSequence // JSON转义系列

JSONEscapeSequence :: // JSON转义系列
  JSONEscapeCharacter // JSON转义字符
  UnicodeEscapeSequence // Unicode转义系列

JSONEscapeCharacter :: one of // JSON转义字符
" / \ b f n r t
```

示例

```javascript
""
"'"
"\""
"ab"
```

## 句法

```
JSONText : // JSON文本
  JSONValue // JSON值

JSONValue : // JSON值
  JSONNullLiteral // JSON Null字面量
  JSONBooleanLiteral // JSON Boolean字面量
  JSONNumber // JSON数字
  JSONString // JSON字符串
  JSONObject // JSON对象
  JSONArray // JSON数组

JSONObject : // JSON对象
  { } // 空对象
  { JSONMemberList } // 非空对象

JSONMemberList : // JSON属性列表
  JSONMember // JSON单个属性
  JSONMemberList , JSONMember // JSON多个属性

JSONMember : // JSON单个属性
  JSONString : JSONValue

JSONArray : // JSON数组
  [ ] // 空数组
  [ JSONElementList ] // 非空数组

JSONElementList : // JSON元素列表
  JSONValue // JSON单个元素
  JSONElementList , JSONValue // 
```

* null：`null`
* 布尔值：`true, false`
* 数值: 十进制数值
  * 小数点后要有数字
  * 禁止前导`0`
  * 不能使用八进制和十六进制
  * 不能使用`NaN, Infinity, -Infinity`
* 字符串：必须使用双引号
* 数组
  * 元素必须是JSON格式值
  * 最后一个元素后不能有逗号
* 对象
  * 属性名必须放在双引号中
  * 属性值必须是JSON格式值
  * 最后一个属性后不能有逗号

示例

```javascript
// JSONNullLiteral
null

// JSONBooleanLiteral
true
false

// JSONNumber
1
1.2
1e+2
1.2e+2
-1.2e+2

// JSONString
""
"'"
"\""
"ab"

// JSONObject
{}
{
  "a": 1
}
{
  "a": 1,
  "b": {}
}

// JSONArray
[]
[1]
[1, []]
```

## 属性

```javascript
JSON.[[Class]] = 'JSON'
JSON.[[Prototype]] = 'Object.prototype'
JSON.[[Extensible]] = true
```

## 方法

### JSON.parse(text [ , reviver ])

将JSON格式的字符串转换为JavaScript值

```javascript
/**
 * text转换为String类型
 *   不符合JSON格式时，抛出SyntaxError
 *   符合JSON格式时，解析成JavaScript值unfiltered
 * reviver不为函数时，直接返回unfiltered
 * reviver为函数时，使用递归函数Walk对unfiltered进行过滤转换，Walk接收对象和属性两个参数，初始参数为{'': unfiltered}和''
 *   使用对象和属性获取属性值
 *   该属性值为数组时，先对每个元素进行Walk处理，参数为数组和索引
 *      返回值为undefined时，删除该元素
 *      返回值为其他时，修改该元素为返回值
 *   该属性值为对象时，对每个可枚举属性进行Walk处理，参数为对象和属性
 *      返回值为undefined时，删除该元素
 *      返回值为其他时，修改该元素为返回值
 *   最后调用reviver函数，this为对象，参数为属性和属性值，返回返回值
 */
let JText = ToString(text)
let unfiltered
if (!IsJSONFormat(JText)) {
  throw SyntaxError
} else {
  unfiltered = evaluate(JText)
}

if (!IsCallable(reviver)) {
  return unfiltered
}

let root = new Object()
root.[[DefineOwnProperty]]('', {
  [[Value]]: unfiltered,
  [[Writable]]: true,
  [[Enumerable]]: true,
  [[Configurable]]: true
}, false)

function Walk (holder, name) {
  let val = holder.[[Get]](name)
  if (Type(val) === Object) {
    if (val.[[Class]] === 'Array') {
      for (let i = 0; i < val.length; i++) {
        let P = ToString(i)
        let newElement = Walk(val, P)
        if (newElement === undefined) {
          val.[[Delete]](P, false)
        } else {
          val.[[DefineOwnProperty]](P, {
            [[Value]]: newElement,
            [[Writable]]: true,
            [[Enumerable]]: true,
            [[Configurable]]: true
          })
        }
      }
    } else {
      let keys = Object.keys(val)
      for (let i = 0; i < keys.length; i++) {
        let P = keys[i]
        let newElement = Walk(val, P)
        if (newElement === undefined) {
          val.[[Delete]](P, false)
        } else {
          val.[[DefineOwnProperty]](P, {
            [[Value]]: newElement,
            [[Writable]]: true,
            [[Enumerable]]: true,
            [[Configurable]]: true
          })
        }
      } 
    }
  }
  return revivier.[[Call]](holder, name, val)
}

return Walk(root, '')
```

示例

```javascript
// 无reviver
JSON.parse('undefined') // 抛出SyntaxError
JSON.parse('null') // null
JSON.parse('true') // true
JSON.parse('12.3') // 12.3
JSON.parse('"abc"') // 'abc'
JSON.parse('{"a": 1}') // {a: 1}
JSON.parse('["a"]') // ['a']

// 有reviver
function reviver (key, val) {
 console.log('[' + key + ']: ' + val)
  return val
}

JSON.parse('123', reviver)
// []: 123

JSON.parse('{"a":1,"b":[1,2,3],"c":{"d":[4,5,6]}}', reviver)
// [a]: 1
// [0]: 1
// [1]: 2
// [2]: 3
// [b]: [1, 2, 3]
// [0]: 4
// [1]: 5
// [2]: 6
// [d]: 4,5,6
// [c]: {d: [4, 5, 6]}
// []: {a: 1, b: [1, 2, 3], c: {d:[4, 5, 6]}}

JSON.parse('{"a": 1, "b": [1, 2, 3]}', (key, value) => {
  if (key === 'a' || key === '1') {
    return undefined
  } else {
    return value
  }
}) // {b: [1, empty, 3]}
```

### stringify(value [ , replacer [ , space ] ])

将JavaScript值转换为JSON格式的字符串

```javascript
/**
 * value包装成{'': value}对象
 * replacer为函数时，replacerFunction为replacer
 * replacer为数组时，挑选为Number类型、Number对象、String类型、String对象的元素转换为String类型，push进PropertyList
 * space为Number类型或Number对象时，转换为Number类型，限制在[0, 10]内，gap为space个空格
 * space为String类型或String对象时，gap为space前10个字符
 * space为其他时，gap为空字符串
 * 调用Str函数，接收对象和属性两个参数，初始参数为{'': value}和''，返回返回值
 *   使用对象和属性获取属性值
 *   属性值为对象且有toJSON方法时，调用toJSON方法，属性值置为返回值
 *   replacerFunction函数存在时，调用replacerFunction方法，this为对象，参数为属性名和属性值，属性值置为返回值
 *   属性值为Number对象时，转换为Number类型
 *   属性值为String对象时，转换为String类型
 *   属性值为Boolean对象时，属性值置为[[PrimitiveValue]]值
 *   属性值为Null或Boolean类型时，转换为String类型并返回
 *   属性值为Number类型时
 *      属性值为有限值，转换为String类型并返回
 *      属性值不为有限值，返回'null'
 *   属性值为String类型时，进行Quate处理
 *      使用双引号包裹
 *      "、\、\b、\f、\n、\r、\r加\
 *      \u0020前的字符改为\\u
 *   属性值为数组时，进行JA处理
 *      将元素处理为Str(holder, index)形式
 *         Str函数返回值为undefined时置为'null'
 *      无gap时返回'[' + properties + ']'，属性分隔符为','
 *      有gap时返回'[' + sparator + properties + '\n' + lastIndent + ']'，属性分隔符为',\n' + indent
 *   属性值为对象且不为函数时，进行JO处理
 *      将白名单属性或可枚举属性处理为"key":Str(holder, key)形式
 *         Str函数返回值为undefined时忽略
 *         有gap时:后有空格
 *      无gap时返回'{' + properties + '}'，属性分隔符为','
 *      有gap时返回'{' + sparator + properties + '\n' + lastIndent + '}'，属性分隔符为',\n' + indent
 *   属性值为其他值，返回undefined
 */
// 用于对对象进行递归处理
let stack = new List()
let indent = ''

// value处理
let wrapper = new Object()
wrapper.[[DefineOwnProperty]]('', {
  [[Value]]: value,
  [[Writable]]: true,
  [[Enumerable]]: true,
  [[Configurable]]: true
}, false)

// replace处理
let replacerFunction = undefined
let PropertyList = undefined
if (Type(replacer) === Object) {
  if (IsCallable(replacer)) { // replace函数
    replacerFunction = replacer
  } else if (replacer.[[Class]] === 'Array') { // 属性白名单
    PropertyList = []
    for (let i = 0; i < replacer.length; i++) {
      let item
      let v = replacer[i]
      let classStr = Object.prototype.toString.call(v)
      if (classStr === '[object Number]' || classStr === '[object String]') {
        item = ToString(v)
      }
      if (item !== undefined) {
        PropertyList.push(item)
      }
    }
  }
}

// space处理
let gap = ''
let classStr = Object.prototype.toString.call(space)
if (classStr === '[object Number]') { // 数字
  space = Math.max(0, Math.min(10, ToNumber(space)))
  for (let i = 0; i < space; i++) {
    gap += ' '
  }
} else if (classStr === '[object String]') { // 字符串
  gap = space.slice(0, 10)
}

function Str (holder, key) {
  let value = holder.[[Get]](key)
  
  // toJSON处理
  if (Type(value) === Object) {
    let toJSON = value.[[Get]]('toJSON')
    if (IsCallable(toJSON)) {
      value = value.toJSON(key)
    }
  }
  
  // replacerFunction处理
  if (replacerFunction !== undefined) {
    value = replacerFunction.[[Call]](holder, key, value)
  }
  
  // 包装对象处理
  if (Type(value) === Object) {
    if (value.[[Class]] === 'Number') {
      value = ToNumber(value)
    } else if (value.[[Class]] === 'String') {
      value = ToString(value)
    } else if (value.[[Class]] === 'Boolean') {
      value = value.[[PrimitiveValue]]
    }
  }
  
  // Null或Boolean类型
  if (Type(value) === Null || Type(value) === Boolean) {
    return ToString(value)
  }
  
  // Number类型
  if (Type(value) === Number) {
    if (IsFinite(value)) { // 有限值
      return ToString(value)
    } else {
      return 'null'
    }
  }
  
  // String类型
  if (Type(value) === String) {
    return Quate(value)
  }
  
  // 非函数对象
  if (Type(value) === Object && !IsCallable(value)) {
    if (value.[[Class]] === 'Array') { // 数组
      return JA(value)
    } else { // 对象
      return JO(value)
    }
  }
  
  // 其他
  return undefined
}

// 字符串处理
function Quate (value) {
  let product = '"'
  for (let i = 0; i < value.length; i++) {
    let C = value[i]
    let escMap = {
      '"': '\\"',
      '\\': '\\\\',
      '\b': '\\b',
      '\f': '\\f',
      '\n': '\\n',
      '\r': '\\r',
      '\t': '\\t'
    };
    if (escMap(C)) { // "、\、\b、\f、\n、\r、\r处理
      product += escMap(C)
    } else if (C < '\u0020') { // 控制字符处理
      product += '\\u' + (C.charCodeAt(0) + 0x10000).toString(16).substr(1)
    } else {
      product += C
    }
  }
  product += '"'
  return product
}

// 对象处理
function JO (value) {
  if (stack.has(value)) { // 循环引用
    throw TypeError
  }
  
  stack.push(value)
  let lastIndent = indent // 备份上次的缩进
  indent += gap
  
  // 可枚举属性处理
  let K = PropertyList || Object.keys(value)
  let partical = []
  for (let i = 0; i < K.length; i++) {
    let P = K[i]
    let StrP = Str(value, P)
    if (strP !== undefined) {
      let member = Quate(P) + ':' + (gap ? ' ' : '') + StrP
      partial.push(member)
    }
  }
  
  let final
  if (partial.length === 0) {
    final = '{}'
  } else {
    if (gap === '') {
      let separator = ','
      let properties = partial.join(separator)
      final = '{' + properties + '}'
    } else {
      let sparator = ',\n' + indent
      let properties = partial.join(separator)
      final = '{' + sparator + properties + '\n' + lastIndent + '}'
    }
  }
  
  stack.pop()
  indent = lastIndent
  
  return final
}

// 数组处理
function JO (value) {
  if (stack.has(value)) { // 循环引用
    throw TypeError
  }
  
  stack.push(value)
  let lastIndent = indent // 备份上次的缩进
  indent += gap
  
  // 元素处理
  for (let i = 0; i < value.length; i++) {
    let P = ToString[i]
    let StrP = Str(value, P)
    if (strP === undefined) {
      strP = 'null'
    }
    partial.push(StrP)
  }
  
  let final
  if (partial.length === 0) {
    final = '[]'
  } else {
    if (gap === '') {
      let separator = ','
      let properties = partial.join(separator)
      final = '[' + properties + ']'
    } else {
      let sparator = ',\n' + indent
      let properties = partial.join(separator)
      final = '[' + sparator + properties + '\n' + lastIndent + ']'
    }
  }
  
  stack.pop()
  indent = lastIndent
  
  return final
}

return Str(wrapper, '')
```

示例

```javascript
// 原始值
JSON.stringify(undefined) // undefined
JSON.stringify(null) // 'null'
JSON.stringify(true) // 'true'
JSON.stringify(12) // '12'
JSON.stringify('a') // '"a"'
JSON.stringify('"') // '"\\""'
JSON.stringify('\u0000') // '"\\u0000"'

// 特殊对象
JSON.stringify(new Number(12)) // '12'
JSON.stringify(function () {}) // undefined
JSON.stringify(new Date()) // '"2019-08-15T08:32:28.452Z"'

// 对象
let o = {a: 1, b: {c: 2}}
JSON.stringify(o) // '{"a":1,"b":{"c":2}}'

JSON.stringify(o, null, 2)
// '{
//   "a": 1,
//   "b": {
//     "c": 2
//   }
// }'
JSON.stringify(o, null, '|-')
// '{
// |-"a": 1,
// |-"b": {
// |-|-"c": 2
// |-}
// }'

function replacer (key, val) {
 console.log('[' + key + ']: ' + val)
  return val
}
JSON.stringify(o, replacer)
// []: [object Object]
// [a]: 1
// [b]: [object Object]
// [c]: 2
// '{"a":1,"b":{"c":2}}'

let whiteList = ['a', 'c']
JSON.stringify(o, whiteList) // '{"a":1}'

// 数组
let arr = [undefined, 1, {a: 2}]
JSON.stringify(arr) // '[null,1,{"a":2}]'
JSON.stringify(arr, [1, 2]) // '[null,1,{}]'
JSON.stringify(arr, null, 2)
// '[
//   null,
//   1,
//   {
//     "a": 2
//   }
//  ]'

// 循环引用
let a = []
a[0] = a
JSON.stringify(a) // 抛出TypeError
```
