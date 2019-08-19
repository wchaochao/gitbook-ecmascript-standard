# 全局对象

标签（空格分隔）： ECMAScript规范

---

在上下文执行前创建，用于访问全局属性、全局方法

```javascript
// 内置属性的属性描述符默认为
{
  [[Writable]]: true,
  [[Enumerable]]: false,
  [[Configurable]]: true
}
```

## 属性

### 值属性

#### undefined

值undefined

```javascript
GlobalObject.[[DefineOwnProperty]]('undefined', {
  [[Value]]: undefined,
  [[Writable]]: false,
  [[Enumerable]]: false,
  [[Configurable]]: false
})
```

#### NaN

值NaN

```javascript
GlobalObject.[[DefineOwnProperty]]('NaN', {
  [[Value]]: NaN,
  [[Writable]]: false,
  [[Enumerable]]: false,
  [[Configurable]]: false
})
```

#### Infinity

值Infinity

```javascript
GlobalObject.[[DefineOwnProperty]]('Infinity', {
  [[Value]]: Infinity,
  [[Writable]]: false,
  [[Enumerable]]: false,
  [[Configurable]]: false
})
```

### 构造器属性

* `Object`: Object构造器
* `Function`: Function构造器
* `Array`: Array构造器
* `String`：String构造器
* `Boolean`：Boolean构造器
* `Number`: Number构造器
* `Date`: Date构造器
* `RegExp`: RegExp构造器
* `Error`: Error构造器
* `EvalError`: EvalError构造器
* `RangeError`: RangeError构造器
* `ReferenceError`: ReferenceError构造器
* `SyntaxError`: SyntaxError构造器
* `TypeError`: TypeError构造器
* `URIError`: URIError构造器

### 单体对象属性

* `Math`: Math对象
* `JSON`: JSON对象

## 方法

### eval方法

```javascript
eval(x)

/**
 * x不为字符串，直接返回
 * x为字符串，解析为程序
 *    解析失败，抛出SyntaxError
 *    解析成功，创建eval上下文，执行程序并退出eval上下文，结果为result
 *        result.type为normal
 *            result.value为empty, 返回undefined
 *            result.value不为empty, 返回result.vlaue
 *        result.type为throw, 抛出result.value
 */
if (Type(x) !== String) {
  return x
}

let prog = parseToProgram(x)
if (parseFail) {
  throw SyntaxError()
}

let evalCtx = createEvalContext()
let result = exec(x)
evalCtx.quit()

if (result.type === normal) {
  return result.value === empty ? undefined : result.value
} else if (result.type === throw) {
  throw result.value
}
```

示例

```javascript
eval(2) // 2
eval('let a = 1') // undefined
eval('1 + 1') // 2
eval('throw new Error('error')') // error
```

### 数字方法

#### isNaN(number)

转换为Number类型后判断是否是NaN

```javascript
/**
 * 将number转换为Number类型
 *     是NaN，返回true
 *     不是NaN，返回false
 */
return SameValue(ToNumber(number), NaN)
```

示例

```javascript
isNaN(undefined) // true
isNaN(null) // false
isNaN(true) // false
isNaN(1) // false
isNaN(NaN) // true
isNaN('123') // false
isNaN('123ab') // true
isNaN({}) // true
```

#### isFinite(number)

转换为Number类型后判断是否是有限数字

```javascript
/**
 * 将number转换为Number类型
 *     是NaN、Infinity、-Infinity，返回false
 *     其他情况，返回true
 */
let n = ToNumber(number)
if (SameValue(n, NaN) || SameValue(abs(n), Infinity)) {
  return false
} else {
  return true
}
```

示例

```javascript
isFinite(undefined) // false
isFinite(null) // true
isFinite(true) // true
isFinite(1) // true
isFinite(NaN) // false
isFinite(Infinity) // false
isFinite('123') // true
isFinite('123ab') // false
isFinite({}) // false
```

### 字符串方法

#### parseInt(string, radix)

将字符串解析为整数

```javascript
/**
 * 将string转换为字符串并去除前置空格，结果为S
 * 确定符号位
 *     S以-开头，符号为负
 *     S以+或其他开头，符号为正
 * 去掉符号位继续解析S
 * 将radix转换为Int32，结果为R
 *    R为0
 *       S以0x或0X开头，去掉0x或0X，按照16进制继续解析S
 *       其他情况，按照10进制解析
 *    R不为0
 *       R为16且S以0x或0X开头，去掉0x或0X，按照16进制解析 
 *       R在[2, 36]之内，按照R进制解析
 *       R在[2, 36]之外，直接返回NaN
 * 获取S中匹配R进制的最长前缀
 *    Z为空字符串，返回NaN
 *    Z不为字符串，将Z按R进制解析为数字 * 符号
 */
let S = removeLeadingWhite(ToString(string))

let sign = 1
if (S[0] === '-') {
  sign = -1
}
if (S[0] === '-' || S[0] === +) {
  S = S.slice(1)
}

let R = ToInt32(radix)
let stripPrefix = true
if (R === 0) {
  R = 10
} else (R !== 0) {
  if (R < 2|| R > 36) {
    return NaN
  }
  if (R !== 16) {
    stripPrefix = false
  }
}
if (stripPrefix) {
  if (S.length >= 2 && (S.indexOf('0x') === 0 || S.indexOf('0X') === 0)) {
    S = S.slice(2)
    R = 16
  }
}

let Z = getLongestPrefixMatchRadix(S)
if (Z === '') {
  return NaN
} else {
  let mathInt = parseRadixDigit(Z, R)
  return sign * mathInt
}
```

示例

```javascript
parseInt(undefined) // NaN
parseInt('   123') // 123
parseInt('\t\v\r123\n ') // 123
parseInt('-123') // -123
parseInt('0x123') // 291
parseInt('0x123', 16) // 291
parseInt('123', 16) // 291
parseInt('0x123', 10) // 0
parseInt('123', 1) // NaN
parseInt('') // NaN
parseInt('123.45') // 123
parseInt('123e-2') // 123

// 自动转换为科学计数法的数字
parseInt(1000000000000000000000.5) // 1
// 等同于
parseInt('1e+21') // 1

parseInt(0.0000008) // 8
// 等同于
parseInt('8e-7') // 8
```

#### parseFloat(string)

将字符串解析为十进制数字

```javascript
/**
 * 将string转换为字符串并去除前置空格，结果为S
 * 获取S中匹配StrDecimalLiteral的最长前缀
 *    Z为空字符串，返回NaN
 *    Z不为空字符串，将Z转换为Number类型并返回
 */
let S = removeLeadingWhite(ToString(string))
let Z = getLongestPrefixMatchStrDecimalLiteral(S)
if (Z === '') {
  return NaN
} else {
  return ToNumber(Z)
}
```

示例

```javascript
parseFloat(undefined) // NaN
parseFloat('   123') // 123
parseFloat('') // NaN
parseFloat('123.45abc') // 123.45
parseFloat('0x12abc') // 18
parseFloat('Infinity') // Infinity
parseFloat('1e2') // 100
```

### URI方法

#### URI组成

```
Scheme : First / Second ; Third ? Fourth
```

#### URI词法

```javascript
uri :::
  uriCharacters(opt) // URI字符

uriCharacters ::: // URI字符
  uriCharacter uriCharacters(opt)

uriCharacter ::: // 单个URI字符
  uriReserved // URI保留字符
  uriUnescaped // URI非转义字符
  uriEscaped // URI转义字符

uriReserved ::: one of // URI保留字符
  ; / ? : @ & = + $ ,

uriUnescaped :::  // URI非转义字符
  uriAlpha // URI字母
  DecimalDigit // 十进制数字
  uriMark // URI标记

uriAlpha ::: one of // URI字母
  a b c d e f g h i j k l m n o p q r s t u v w x y z
  A B C D E F G H I J K L M N O P Q R S T U V W X Y Z

uriMark ::: one of // URI标记
  - _ . ! ~ * ' ( )
  
uriEscaped ::: // URI转义字符
  % HexDigit HexDigit
```

非URI字符都需转义：使用UTF-8编码，再用%xx%xx表示

```
'字'转码：%E5%AD%97
```

#### URI编码

```javascript
encode(string, unescapedSet)

/**
 * 遍历string
 * 字符在unescapedSet内，不做处理
 * 字符不在unescapedSet内
 *     字符为不正确的多字节字符，抛出URIError
 *        只有低位[0xDC00, 0xDFFF]
 *        只有高位[0xD800, 0xDBFF]
 *        高位为[0xD800, 0xDBFF]，低位不为[0xDC00, 0xDFFF]
 *     字符为正确的字节字符，获取Unicode码并按UTF-8编码，每个字节转换为%xy, xy为大写的十六进制
 */
let strLen = string.length
let R = ''
let k = 0

while (true) {
  if (k === strLen) {
    return R
  }
  
  let C = string[k]
  if (unescapedSet.includes(C)) {
    R += C
  } else {
    let V = C.charCodeAt(0)
    
    // 多字节处理
    if (V >= 0xDC00 && V <= 0xDFFF) {
      throw URIError
    } else if (V >= 0xD800 && V <= 0xDBFF) {
      if (k + 1 >= strLen) {
        throw URIError
      }
      k++
      let kChar = string[k].charCodeAt(0)
      if (kChar < 0xDC00 || kChar > 0xDFFF) {
        throw URIError
      } else {
        V = (V – 0xD800) × 0x400 + (kChar – 0xDC00) + 0x10000
      }
    }
    
    let Octets = TransformUnicodeToOctetArrayByUTF8(V)
    for (let i = 0; i < Octets.length; i++) {
      let S = '%' + TransformOctetToHexUppercaseString(Octets[i])
      R += S
    }
    k++
  }
}
```

UTF-16字符转UTF-8字符

| Code Unit Value | Representation | 1st Octet | 2nd Octet | 3rd Octet | 4th Octet |
| --- | --- | --- | --- | --- | --- |
| 0x0000 - 0x007F | 00000000 0zzzzzzz | 0zzzzzzz |			
| 0x0080 - 0x07FF | 00000yyy yyzzzzzz | 110yyyyy | 10zzzzzz	|	
| 0x0800 - 0xD7FF | xxxxyyyy yyzzzzzz | 1110xxxx | 10yyyyyy | 10zzzzzz |	
| 0xD800 - 0xDBFF<br/> followed by<br/> 0xDC00 – 0xDFFF | 110110vv vvwwwwxx<br/> followed by<br/> 110111yy yyzzzzzz | 11110uuu | 10uuwwww | 10xxyyyy | 10zzzzzz |
| 0xD800 - 0xDBFF<br/> not followed by<br/> 0xDC00 – 0xDFFF | causes URIError |		
| 0xDC00 – 0xDFFF | causes URIError |
| 0xE000 - 0xFFFF | xxxxyyyy yyzzzzzz | 1110xxxx | 10yyyyyy | 10zzzzzz |

#### URI解码

```javascript
decode(string, reversedSet)

/**
 * 遍历string
 * 字符不是%，不做处理
 * 字符是%
 *     字符后的字符无法组成正确意义的UTF-8值，抛出URIError
 *        %后不为两位大写十六进制字母
 *        %xy(一个或多个)没有对应的Unicode码
 *     字符后的字符能组成正确意义的UTF-8值，转换为对应的Unicode字符
 *        字符不在reversedSet内，为该字符
 *        字符在reversedSet内，为该字符对应的%xy(一个或多个)原字符
 */
let strLen = string.length
let R = ''
let k = 0

while (true) {
  if (k === strLen) {
    return R
  }
  
  let S = ''
  let C = string[k]
  if (C !== '%') {
    S = C
  } else {
    let start = k
    if (k + 2 >= strLen) {
      throw URIError
    }

    let B = ToNumber('0x' + string[k + 1] + string[k + 2])
    if (SameValue(B, NaN)) {
      throw URIError
    }

    K += 2
    let bin = B.toString(2)
    if (bin.startWith('0')) {
      let C = String.fromCharCode(B)
      if (!reversedSet.includes(C)) {
        S = C
      } else {
        S = string.slice(start, k + 1)
      }
    } else {
      let n = 1
      while (((B << n) & 0x80)) !== 0) {
        n++
      }
     
      if (n === 1 || n > 4) {
        throw URIError
      }
      
      let Octets = new Array(n)
      Octets[0] = B
      
      if ( k + (3 × (n – 1)) >= strLen) {
        throw URIError
      }
      
      for (let j = 1; j < n; j++) {
        k++
        if (string[k] !== '%') {
          throw URIError
        }
        
        let B = ToNumber('0x' + string[k + 1] + string[k + 2])
        if (SameValue(B, NaN)) {
          throw URIError
        }
        
        let bin = B.toString(2)
        if (!bin.startWith('10')) {
          throw URIError
        }
        
        Octets[j] = B
        K += 2
      }
      
      let V = transformOctetArrayToUnicodeByUTF8(Octets)
      if (V === undefined) {
        throw URIError
      } else if (V < 0x10000) {
        let C = String.fromCharCode(V)
        if (!reversedSet.includes(C)) {
          S = C
        } else {
          S = string.slice(start, k + 1)
        }
      } else {
        let L = ((V – 0x10000) & 0x3FF) + 0xDC00
        let H = (((V – 0x10000) >> 10) & 0x3FF) + 0xD800
        S = String.fromCharCode(H, L)
      }
    }
  }
  R += S
  k++
}
```

#### encodeURI(uri)

编码URI

```javascript
/**
 * uri转换为String类型
 * unescapedURISet为uriUnescaped + uriReserved + '#'
 * 执行encode(uriString, unescapedURISet )
 */
let uriString = ToString(uri)
let unescapedURISet = uriReserved + uriUnescaped + '#'
return encode(uriString, unescapedURISet)
```

示例

```javascript
encodeURI('http://www.w3school.com.cn/My first/') // http://www.w3school.com.cn/My%20first/
```

#### encodeURIComponent(uriComponent)

编码URI组件

```javascript
/**
 * uriComponent转换为String类型
 * unescapedURIComponentSet为uriUnescaped
 * 执行encode(componentString, unescapedURIComponentSet)
 */
let componentString = ToString(uriComponent)
let unescapedURIComponentSet = uriUnescaped
return encode(componentString, unescapedURIComponentSet)
```

示例

```javascript
encodeURIComponent('http://www.w3school.com.cn/p 1/') // http%3A%2F%2Fwww.w3school.com.cn%2Fp%201%2F
```

#### decodeURI(encodeURI)

解码URI

```javascript
/**
 * encodeURI转换为String类型
 * reservedURISet为uriReserved加'#'
 * 执行decode(uriString, reservedURISet)
 */
let uriString = ToString(encodeURI)
let reservedURISet = uriReserved + '#'
return decode(uriString, reservedURISet)
```

示例

```javascript
decodeURI('http://www.w3school.com.cn/My%20first/') // http://www.w3school.com.cn/My first/

decodeURI('http%3A%2F%2Fwww.w3school.com.cn%2Fp%201%2F') // http%3A%2F%2Fwww.w3school.com.cn%2Fp 1%2F
```

#### decodeURIComponent(encodedURIComponent)

解码URI组件

```javascript
/**
 * encodedURIComponent转换为String类型
 * reservedURIComponentSet为空字符串
 * 执行decode(componentString, reservedURIComponentSet)
 */
let componentString = ToString(encodedURIComponent)
let reservedURIComponentSet = ''
return decode(componentString, reservedURIComponentSet)
```

示例

```javascript
decodeURIComponent('http://www.w3school.com.cn/My%20first/') // http://www.w3school.com.cn/My first/

decodeURIComponent('http%3A%2F%2Fwww.w3school.com.cn%2Fp%201%2F') // http://www.w3school.com.cn/p 1/ 
```

### 扩展方法

#### escape(string)

对字符串进行转义

```javascript
/**
 * string转换为String类型，对string的每个字符进行转义
 *   字符为a-zA-Z0-9@*_+-./，仍为该字符
 *   字符Unicode码小于256，改为%xx形式
 *   字符Unicode码不小于256，改为%uxxxx形式
 */
let str = ToString(string)
let len = str.length
let R = ''

function getHexString (code, num) {
  let n = Math.pow(16, num) + code
  return n.toString(16).slice(1)
}

for (let i = 0; i < len; i++) {
  let char = str[i]
  let code = char.charCodeAt(0)
  let S = ''
  if (/[a-zA-Z0-9@*_+-./]/.test(char)) {
    S = char
  } else if (code < 256) {
    S = getHexString(code, 2)
  } else {
    S = getHexString(code, 4)
  }
  R += S
}

return R
```

示例

```javascript
escape('a') // 'a'
escape('?') // '%3F'
escape('中') // '%u4E2D'
```

### unescape(string)

对字符串解转义

```javascript
/**
 * string转换为String类型，对%xx、%uxxxx进行解转义
 */
let str = ToString(string)
let len = str.length
let R = ''

function getStringByHex () {
  let code = 0
  for (let i = 0; i < arguments.length; i++) {
    let hex = ToNumber('0x' + arguments[i])
    if (SameValue(hex, NaN)) {
      return null
    } else {
      code = code * 16 + hex
    }
  }
  return String.fromCharCode(code)
}

let k = 0
while (k < len) {
  let c = str[k]
  let S = c
  let escaped
  if (c === '%') {
    if (str[k + 1] === 'u' && k + 6 >= len) {
      let char = getStringByHex(str[k + 2], str[k + 3], str[k + 4], str[k + 5])
      if (Type(char) === String) {
        S = char
        k += 5
      }
    } else if (k + 3 >= len) {
      let char = getStringByHex(str[k + 1], str[k + 2])
      if (Type(char) === String) {
        S = char
        k += 2
      }
    }
  }
  
  R += S
  k++
}
```

示例

```javascript
escape('a') // 'a'
escape('%3F') // '?'
escape('%u4E2D') // '中'
```
