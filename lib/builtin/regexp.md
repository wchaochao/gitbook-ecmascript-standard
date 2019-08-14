# RegExp构造器

标签（空格分隔）： ECMAScript规范

---

## 正则模式

```
Pattern :: // 模式
  Disjunction // 析取
  
Disjunction :: // 析取
  Alternative // 单个选择
  Alternative | Disjunction // 多个选择

Alternative :: // 单个选择
  [empty] // 空匹配项
  Alternative Term // 多个匹配项

Term :: // 单个匹配项
  Assertion // 断言
  Atom // 原子匹配
  Atom Quantifier // 重复匹配

Assertion :: // 断言
  ^ // 开头处
  $ // 结尾处
  \ b // 单词边界处
  \ B // 非单词边界处
  ( ? = Disjunction ) // 后面紧跟着
  ( ? ! Disjunction ) // 后面不紧跟着

Quantifier :: // 数量
  QuantifierPrefix // 贪婪匹配
  QuantifierPrefix ? // 非贪婪匹配

QuantifierPrefix :: // 数量符号
  * // 任意次
  + // 至少1次
  ? // 0次或1次
  { DecimalDigits } // n次
  { DecimalDigits , } // 至少n次
  { DecimalDigits , DecimalDigits } // n到m次

Atom :: // 原子匹配
  PatternCharacter // 普通字符
  . // 行结束符外的任意字符
  \ AtomEscape // 转义
  CharacterClass // 字符类
  ( Disjunction ) // 捕获组
  ( ? : Disjunction ) // 非捕获组

PatternCharacter :: // 普通字符
  SourceCharacter but not one of
    ^ $ \ . * + ? ( ) [ ] { } |

AtomEscape :: // 转义
  DecimalEscape // 十进制转义
  CharacterEscape // 字符转义
  CharacterClassEscape // 字符类转义

DecimalEscape :: // 十进制转义
  DecimalIntegerLiteral [lookahead ∉ DecimalDigit]

CharacterEscape :: // 字符转义
  ControlEscape // 控制转义
  c ControlLetter // 控制字符
  HexEscapeSequence // 十六进制转义
  UnicodeEscapeSequence // Unicode转义
  IdentityEscape // 任意字符转义

ControlEscape :: one of // 控制转义
  f n r t v

ControlLetter :: one of // 控制字符
  a b c d e f g h i j k l m n o p q r s t u v w x y z
  A B C D E F G H I J K L M N O P Q R S T U V W X Y Z

IdentityEscape :: // 任意字符转义
  SourceCharacter but not IdentifierPart
  <ZWJ>
  <ZWNJ>

CharacterClassEscape :: one of // 字符类转义
  d D s S w W

CharacterClass :: // 字符类
  [ [lookahead ∉ {^}] ClassRanges ] // 类范围
  [ ^ ClassRanges ] // 类范围之外

ClassRanges :: // 类范围
  [empty] // 空范围
  NonemptyClassRanges // 非空类范围

NonemptyClassRanges :: // 非空类范围
  ClassAtom // 类原子
  ClassAtom NonemptyClassRangesNoDash // 多个类原子
  ClassAtom - ClassAtom ClassRanges // 类原子范围

ClassAtom :: // 类原子
  - // 连字符类原子
  ClassAtomNoDash // 非连字符类原子

ClassAtomNoDash :: // 非连字符类原子
  SourceCharacter but not one of \ or ] or - // 普通字符类原子
  \ ClassEscape // 转义类原子

ClassEscape :: // 转义类原子
  DecimalEscape // 十进制转义
  b // 退格转义
  CharacterEscape // 字符转义
  CharacterClassEscape // 字符类转义

NonemptyClassRangesNoDash :: // 非空非连字符范围
  ClassAtom // 类原子
  ClassAtomNoDash NonemptyClassRangesNoDash // 多个类原子
  ClassAtomNoDash - ClassAtom ClassRanges // 类原子范围
```

### 术语

* `Input`: 匹配的字符串
* `InputLength`: 匹配字符串的长度
* `NcapturingParens`: 捕获组数目
* `IgnoreCase`: 是否忽略大小写
* `Multiline`: 是否多行匹配
* `CharSet`: 字符集
* `State`: {endIndex: Number, captures: Array}，匹配状态
 * `endIndex`: 下次匹配开始的位置
 * `captures`: 捕获组的匹配情况
* `MatchResult `: State | failure, 匹配结果
* `Continuation`: (State) => MatchResult, 下次匹配函数
* `Matcher`: (State, Continuation) => MatchResult, 匹配函数
 * 先对初始State进行Matcher匹配，得到新State再进行Continuation匹配
* `AssertionTester`: (State) => Boolean, 判断能否匹配
* `EscapeValue`: 转义值（捕获组引用）

### Pattern

```javascript
Pattern :: Disjunction

/**
 * 执行Disjunction获取Matcher
 * 返回一个函数，参数为str、index
 *    初始状态为{endIndex: index, captures: new Array(NcapturingParens)}，进行Matcher匹配，返回匹配结果
 */
let m = evaluate(Disjunction)
return (str, index) => {
  Input = str
  InputLength = str.length
  let s = {
    endIndex: index,
    captures: new Array(NcapturingParens)
  }
  let c = (state) => state
  return m(s, c)
}
```

### Disjunction

#### 单个选择

```javascript
Disjunction :: Alternative

/**
 * 执行Alternative获取Matcher并返回这个Matcher
 */
let m = evaluate(Alternative)
return m
```

#### 多个选择

```javascript
Disjunction :: Alternative | Disjunction

/**
 * 执行Alternative获取Matcher1
 * 执行Disjunction获取Matcher2
 * 返回一个Matcher, 该Matcher先匹配Matcher1，failure后再匹配Matcher2
 */
let m1 = evaluate(Alternative)
let m2 = evaluate(Disjunction)
return (s, c) => {
  let r = m1(s, c)
  if (r !== failure) {
    return r
  } else {
    return m2(s, c)
  }
}
```

示例

```javascript
/a|ab/.exec('abc') // ['a', index: 0, input: 'abc']
/((a)|(ab))((c)|(bc))/.exec('abc') // ['abc', 'a', 'a', undefined, 'bc', undefined, 'bc', index: 0, input: 'abc']
```

### Alternative

#### 空匹配项

```javascript
Alternative :: [empty]

/**
 * 返回一个Matcher，该Matcher执行下一个匹配函数
 */
return (s, c) => {
  return c(s)
}
```

#### 多个匹配项

```javascript
Alternative :: Alternative Term

/**
 * 执行Alternative获取Matcher1
 * 执行Term获取Matcher2
 * 返回一个Matcher，该Matcher先匹配Matcher1，再匹配Matcher2
 */
let m1 = evaluate(Alternative)
let m2 = evaluate(Term)
return (s, c) => {
  let d = (x) => {
    return m2(x, c)
  }
  return m1(s, d)
}
```

### Term

#### 断言

```javascript
Term :: Assertion

/**
 * 执行Assertion获取AssertionTester 
 * 返回一个Matcher
 *    AssertionTester能匹配时，执行下一个匹配函数
 *    AssertionTester不能匹配时，返回failure
 */
let t = evaluate(Assertion)
return (s, c) => {
  if (t(s)) {
    return c(x)
  } else {
    return failure
  }
}
```

#### 原子匹配

```javascript
Term :: Atom

/**
 * 执行Atom获取Matcher并返回这个Matcher
 */
let m = evaluate(Atom)
return m
```

#### 数量个原子匹配

```javascript
Term :: Atom Quantifier

/**
 * 执行Atom获取Matcher
 * 执行Quantifier获取数量范围[min, max]和是否为贪婪匹配greedy
 * 获取Atom内的捕获组索引和数量，进行重复匹配
 *    每次匹配将Atom内的捕获组匹配置为undefined，数量范围减一
 *    连续匹配直至min减为0
 *    非贪婪匹配，将匹配结果传给下一次匹配
 *       整体有匹配，返回该匹配结果
 *       整体无匹配，进行下一次重复匹配
 *    贪婪匹配，进行下一次重复匹配
 *       整体有匹配，继续下一次重复匹配，直至max为0
 *       整体无匹配，返回上一次匹配结果
 */
let m = evaluate(Atom)
let {min, max, greedy} = evaluate(Quantifier)
if (isFinite(max) && max < min) {
  throw TypeError
}
let parenIndex = getCaptureParenIndex(Atom)
let parenCount = getInnerCaptureParenCount(Atom)

function RepeatMatcher (m, min, max, greedy, s, c, parenIndex, parenCount) {
  // 重复匹配至max为0
  if (max === 0) { 
    return c(s)
  }
  
  // 每次匹配前将Atom的捕获组匹配置为undefined
  let cap = copy(s.captures)
  for (let k = parenIndex; k < parenIndex + parenCount; k++) {
    cap[k] = undefined
  }
  let r = {
    endIndex: s.endIndex,
    captures: cap
  }
  
  // 下一次重复匹配
  let d = (x) => {
    if (min === 0 && x.endIndex === s.endIndex) { // 避免无限循环
      return failure
    }
    let min2 = min === 0 ? 0 : min - 1
    let max2 = max === Infinity ? Infinity ? max - 1
    return RepeatMatcher(m, min2, max2, greedy, x, c, parenIndex, parenCount)
  }
  
  // 重复匹配至min为0
  if (min !== 0) {
    return m(r, d)
  }
  
  if (greedy === false) { // 非贪婪匹配
    let z = c(s)
    if (z !== failure) {
      return z
    } else {
      return m(r, d)
    }
  } else { // 贪婪匹配
    let z = m(r, d)
    if (z !== failure) {
      return z
    } else {
      return c(s)
    }
  }
}

return (s, c) => {
  return RepeatMatcher(m, min, max, greedy, s, c, parenIndex, parenCount)
}
```

示例

```javascript
/a[a-z]{2,4}/.exec('abcdefghi') // ['abcde', index: 0, input: 'abcdefghi']
/a[a-z]{2,4}?/.exec('abcdefghi') // ['abc', index: 0, input: 'abcdefghi']
/(aa|aabaac|ba|b|c)*/.exec('aabaac') // ['aaba', 'ba', index: 0, input: 'aabaac']
/(z)((a+)?(b+)?(c))*/.exec('zaacbbbcac') // ['zaacbbbcac', 'z', 'ac', 'a', undefined, 'c', index: 0, input: 'zaacbbbcac']
```

### Assertion

#### 开头

```javascript
Assertion :: ^

/**
 * 返回一个AssertionTester函数
 *    非多行模式，判断endIndex是否为开头
 *    多行模式，判断endIndex是否为行首
 */
return (s) => {
  let e = s.endIndex
  if (e === 0) {
    return true
  } else {
    if (!Multiline) {
      return false
    } else {
      return IsLineTerminator(Input[e - 1])
    }
  }
}
```

示例

```javascript
/^hello/.test('hello world') // true
/^hello/m.test('\nhello world') // true
```

#### 结尾

```javascript
Assertion :: $

/**
 * 返回一个AssertionTester函数
 *    非多行模式，判断endIndex是否为结尾
 *    多行模式，判断endIndex是否为行尾
 */
return (s) => {
  let e = s.endIndex
  if (e === InputLength) {
    return true
  } else {
    if (!Multiline) {
      return false
    } else {
      return IsLineTerminator(Input[e])
    }
  }
}
```

示例

```javascript
/world$/.test('hello world') // true
/world$/m.test('hello world\n') // true
```

#### 单词边界处

```javascript
Assertion :: \ b

/**
 * 返回一个AssertionTester函数
 *    判断endIndex是否为单词边界处，即endIndex - 1与endIndex处的字符一个为单词字符、另一个不为单词字符（[a-zA-Z0-9_]）
 */
function IsWordChar (e) {
  if (e === 0 || e === InputLength) {
    return false
  }
  let c = Input(e)
  return /[a-zA-Z0-9_]/.test(c)
}
 
return (s) => {
  let e = s.endIndex
  return IsWordChar(e - 1) !== IsWordChar(e)
}
```

示例

```javascript
/\bworld/.test('hello world') // true
/\bworld/m.test('hello_world\n') // false
```

#### 非单词边界处

```javascript
Assertion :: \ B

/**
 * 返回一个AssertionTester函数
 *    判断endIndex是否不为单词边界处，即endIndex - 1与endIndex处的字符都为单词字符或都不为单词字符（[a-zA-Z0-9_]）
 */
function IsWordChar (e) {
  if (e === 0 || e === InputLength) {
    return false
  }
  let c = Input(e)
  return /[a-zA-Z0-9_]/.test(c)
}
 
return (s) => {
  let e = s.endIndex
  return IsWordChar(e - 1) === IsWordChar(e)
}
```

示例

```javascript
/\Bworld/.test('hello world') // false
/\Bworld/m.test('hello_world\n') // true
```

#### 后面紧跟着

```javascript
Assertion :: ( ? = Disjunction )

/**
 * 执行Disjunction获取Matcher函数m
 * 返回一个Matcher函数，该函数进行m匹配
 *    匹配，将上一个匹配结果的endIndex和这个匹配结果的captures传给下一个匹配函数
 *    不匹配，返回failure
 */
let m = evaluate(Disjunction)
return (s, c) => {
  let d = (x) => x
  let r = m(s, d)
  if (r === failure) {
    return failure
  } else {
    let z = {
      endIndex: s.endIndex,
      captures: r.captures
    }
    return c(z)
  }
}
```

示例

```javascript
/a(?=(b))/.exec('ab') // ['a', 'b', index: 0, input: 'ab']
```

#### 后面不紧跟着

```javascript
Assertion :: ( ? ! Disjunction )

/**
 * 执行Disjunction获取Matcher函数m
 * 返回一个Matcher函数，该函数进行m匹配
 *    匹配，返回failure
 *    不匹配，将上一个的匹配结果传给下一个匹配函数
 */
let m = evaluate(Disjunction)
return (s, c) => {
  let d = (x) => x
  let r = m(s, d)
  if (r !== failure) {
    return failure
  } else {
    return c(s)
  }
}
```

示例

```javascript
/a(?!(c))/.exec('ab') // ['a', undefined, index: 0, input: 'ab']
```

### Quantifier

#### 贪婪匹配

```javascript
Quantifier :: QuantifierPrefix

/**
 * 执行QuantifierPrefix获取数量范围[min, max]
 * 返回{min, max, greedy: true}
 */
let {min, max} = evaluate(QuantifierPrefix)
return {
  min,
  max,
  greedy: true
}
```

#### 非贪婪匹配

```javascript
Quantifier :: QuantifierPrefix

/**
 * 执行QuantifierPrefix获取数量范围[min, max]
 * 返回{min, max, greedy: false}
 */
let {min, max} = evaluate(QuantifierPrefix)
return {
  min,
  max,
  greedy: false
}
```

#### 任意次

```javascript
QuantifierPrefix :: *

/**
 * 返回数量范围{min: 0, max: Infinity}
 */
return {
  min: 0,
  max: Infinity
}
```

示例

```javascript
/a(b*)b/.exec('abb') // ['abb', 'b', index: 0, input: 'abb']
/a(b*?)b/.exec('abb') // ['ab', '', index: 0, input: 'abb']
```

#### 至少一次

```javascript
QuantifierPrefix :: +

/**
 * 返回数量范围{min: 1, max: Infinity}
 */
return {
  min: 1,
  max: Infinity
}
```

示例

```javascript
/a(b+)b/.exec('abbb') // ['abbb', 'bb', index: 0, input: 'abbb']
/a(b+?)b/.exec('abbb') // ['abb', 'b', index: 0, input: 'abbb']
```

#### 0次或1次

```javascript
QuantifierPrefix :: ?

/**
 * 返回数量范围{min: 0, max: 1}
 */
return {
  min: 0,
  max: 1
}
```

示例

```javascript
/a(b?)b/.exec('abb') // ['abb', 'b', index: 0, input: 'abb']
/a(b??)b/.exec('abb') // ['ab', '', index: 0, input: 'abb']
```

#### n次

```javascript
QuantifierPrefix :: { DecimalDigits }

/**
 * 执行DecimalDigits获取次数n
 * 返回数量范围{min: n, max: n}
 */
let n = evaluate(DecimalDigits)
return {
  min: n,
  max: n
}
```

示例

```javascript
/a(b{1})b/.exec('abb') // ['abb', 'b', index: 0, input: 'abb']
/a(b{1}?)b/.exec('abb') // ['abb', 'b', index: 0, input: 'abb']
```

#### 至少n次

```javascript
QuantifierPrefix :: { DecimalDigits , }

/**
 * 执行DecimalDigits获取次数n
 * 返回数量范围{min: n, max: Infinity}
 */
let n = evaluate(DecimalDigits)
return {
  min: n,
  max: Infinity
}
```

示例

```javascript
/a(b{1,})b/.exec('abbb') // ['abbb', 'bb', index: 0, input: 'abbb']
/a(b{1,}?)b/.exec('abbb') // ['abb', 'b', index: 0, input: 'abbb']
```

#### n到m次

```javascript
QuantifierPrefix :: { DecimalDigits , DecimalDigits }

/**
 * 执行第一个DecimalDigits获取次数n
 * 执行第二个DecimalDigits获取次数m
 * 返回数量范围{min: n, max: m}
 */
let n = evaluate(FirstDecimalDigits)
let m = evaluate(SecondDecimalDigits)
return {
  min: n,
  max: m
}
```

示例

```javascript
/a(b{1,3})b/.exec('abbb') // ['abbb', 'bb', index: 0, input: 'abbb']
/a(b{1,3}?)b/.exec('abbb') // ['abb', 'b', index: 0, input: 'abbb']
```

### Atom

#### 普通字符

```javascript
Atom :: PatternCharacter

/**
 * 执行PatternCharacter获取字符，组成只含这个字符的字符集
 * 返回一个Matcher函数，该Matcher进行字符集匹配
 *   endIndex处的字符在字符集内，endIndex加1，进行下一次匹配
 *   endIndex处的字符不在字符集内，返回failure
 *   IgnoreCase为true时忽略大小写
 */
let ch = evaluate(PatternCharacter)
let A = new CharSet(ch)

function CharacterSetMatcher (A, invert) {
  return (s, c) => {
    let e = s.endIndex
    if (e === InputLength) {
      return failure
    }
    
    let ch = Input[e]
    if (invert === A.has(ch, Canonicalize)) {
      return failure
    }
    
    let x = {
      endIndex: e + 1,
      captures: s.captures
    }
    return c(x)
  }
}

function Canonicalize (ch) {
  if (!IgnoreCase) {
    return ch
  }
  
  let u = ch.toUpperCase(ch)
  if (u.length > 1) {
    return ch
  }
  if (ch.charCodeAt(0) >= 128 && u.charCodeAt(0) < 128) {
    return ch
  } else {
    return u
  }
}

return CharacterSetMatcher(A, false)
```

示例

```javascript
/at/.exec('cat') // ['at', index: 1, input: 'cat']
```

#### 行结束符外的任意字符

```javascript
Atom :: .

/**
 * 设置字符集为行结束符外的任意字符
 * 返回一个Matcher函数，该Matcher进行字符集匹配
 */
let ch = evaluate(PatternCharacter)
let A = new CharSet(LineTerminator, except)

return CharacterSetMatcher(A, false)
```

示例

```javascript
/.at/.exec('cat') // ['cat', index: 0, input: 'cat']
```

#### 转义字符

```javascript
Atom :: \ AtomEscape

/**
 * 执行AtomEscape获取Matcher函数，返回该Matcher
 */
let m = evaluate(AtomEscape)
return m
```

#### 字符类

```javascript
Atom :: CharacterClass

/**
 * 执行CharacterClass获取字符集A和是否在字符集内invert
 * 返回一个Matcher函数，该Matcher进行字符集匹配
 */
let {A, invert} = evaluate(CharacterClass)

return CharacterSetMatcher(A, invert)
```

示例

```javascript
// 类范围
/[-]/.test('-') // true
/[a]/.test('a') // true
/[\0]/.test('\u0000') // true
/(a)[\1]/.test('aa') // false
/[\b]/.test('\b') // true
/[\a]/.test('a') // true
/[\d]/.test('1') // true
/[abc]/.test('a') // true
/[a-c]/.test('a') // true

// 类范围之外
/[^]/.test('a') // true
/[^abc]/.test('a') // false
```

#### 捕获组

```javascript
Atom :: ( Disjunction )

/**
 * 执行Disjunction获取Macher1
 * 返回一个Matcher函数，该Matcher进行Macher1匹配
 *    有匹配时，将对应的捕获组置为该匹配，进行下一次匹配
 *    无匹配时，返回failure
 */
let m = evaluate(Disjunction)
let parenIndex = getCaptureParenIndex(Atom)

return (s, c) => {
  let d = (x) => {
    let e = x.endIndex
    let cap = copy(s.captures)
    cap[parenIndex] = Input.slice(s.endIndex, e)
    let r = {
      endIndex: e,
      captures: cap
    }
    return c(r)
  }
  return m(s, d)
}
```

示例

```javascript
/<(\w+)([^>]*)>(.*?)<\/\1>/.exec('<b class="hello">Hello</b>') //  ['<b class="hello">Hello</b>', 'b', ' class="hello"', 'Hello', index: 0, input: '<b class="hello">Hello</b>']
```

#### 非捕获组

```javascript
Atom :: ( ? : Disjunction )

/**
 * 执行Disjunction获取Matcher函数，返回该Matcher
 */
let m = evaluate(Disjunction)
return m
```

示例

```javascript
/(?:http|ftp):\/\/([^/\r\n]+)(\/[^\r\n]*)?/.exec('http://google.com/') // ['http://google.com/', 'google.com', '/', index: 0, input: 'http://google.com/']
```

### AtomEscape

#### 十进制转义

捕获组引用

```javascript
AtomEscape :: DecimalEscape

/**
 * 执行DecimalEscape获取结果E
 * E为<NUL>字符，返回一个Matcher函数，该Matcher进行只含该字符的字符集匹配
 * E为整数n, 表示第n个捕获组，返回一个Matcher函数
 *    n = 0 或 n > NCapturingParens, 抛出SyntaxError
 *    第n个捕获组为undefined, 直接进行下一次匹配
 *    第n个捕获组为字符串, 进行字符串匹配，匹配成功后再进行下一次匹配
 */
let E = evaluate(DecimalEscape)
if (E === '\u0000') {
  let A = new CharSet(E)
  return CharacterSetMatcher(A, false)
}

let n = E
if (n = 0 || n > NCapturingParens) {
  throw SyntaxError
}

return (s, c) => {
  let cap = s.captures
  let str = cap[n]
  if (str === undefined) {
    return c(s)
  }
  
  let e = s.endIndex
  let len = str.length
  let f = e + len
  if (f > InputLength) {
    return failure
  }
  for (let i = 0; i < len; i++) {
    if (Canonicalize(s[i]) !==  Canonicalize(Input[e+i])) {
      return failure
    }
  }
  
  let y = {
    endIndex: f,
    captrues: cap
  }
  return c(y)
}
```

示例

```javascript
// NUL匹配
/a\0/.test('a\u0000') // true

// 捕获组引用
/(a)\2/.exec('ab') // null
/(a)\1/.exec('aa') // ['aa', 'a', index: 0, input: 'aa']
/(a)(a)*\2/.exec('ab') // ['a', 'a', undefined, index: 0, input: 'ab']
```

#### 字符转义

```javascript
AtomEscape :: CharacterEscape

/**
 * 执行CharacterEscape获取字符ch
 * 返回一个Matcher函数，该Matcher进行只含该字符的字符集匹配
 */
let ch = evaluate(CharacterEscape)
let A = new CharSet(ch)
return CharacterSetMatcher(A, false)
```

示例

```javascript
// 控制转义
/\t/.test('\t') // true

// 控制字符
let ctrl = String.fromCharCode(1)
/\ca/.test(ctrl) // true

// 十六进制转义
/\x61/.test('a') // true

// Unicode转义
/\u0061/.test('a') // true

// 任意字符转义
/\+/.test('+') // true
```

#### 字符类转义

```javascript
AtomEscape :: CharacterClassEscape

/**
 * 执行CharacterClassEscape获取字符集A
 * 返回一个Matcher函数，该Matcher进行字符集匹配
 */
let A = evaluate(CharacterClassEscape)
return CharacterSetMatcher(A, false)
```

示例

```javascript
// 十进制数字
/\d/.test('0') // true
/\D/.test('0') // false

// 空白符
/\s/.test(' ') // true
/\S/.test(' ') // false

// 单词
/\w/.test('_') // true
/\W/.test('_') // false
```

### DecimalEscape

```javascript
DecimalEscape :: DecimalIntegerLiteral [lookahead ∉ DecimalDigit]

/**
 * 执行DecimalIntegerLiteral获取整数v
 *    v为0，返回<NUL>字符
 *    v不为0，返回v
 */
let v = evaluate(DecimalIntegerLiteral)
if (v === 0) {
  return '\u0000'
} else {
  return v
}
```

### CharacterEscape

#### 控制转义

```javascript
CharacterEscape :: ControlEscape

/**
 * 按下表进行转义
 */
```

| ControlEscape | Code Unit | Name | Symbol |
| --- | --- | --- | --- |
| t | \u0009 | horizontal tab | <HT> |
| v	| \u000B | vertical tab	| <VT> |
| r | \u000D | carriage return | <CR> |
| n | \u000A | line feed (new line) | <LF> |
| f	| \u000C | form feed | <FF> |

#### 控制字符

```javascript
CharacterEscape :: c ControlLetter

/**
 * 执行ControlLetter获取字符ch
 * 返回ch的Unicode码 % 32后对应的字符
 */
let ch = evaluate(ControlLetter)
let j = ch.charCodeAt(0) % 32
return String.fromCharCode(j)
```

#### 十六进制转义

```javascript
CharacterEscape :: HexEscapeSequence

/**
 * 返回两位十六进制对应的Unicode字符
 */
```

#### Unicode转义

```javascript
CharacterEscape :: UnicodeEscapeSequence

/**
 * 返回Unicode码对应的Unicode字符
 */
```

#### 任意字符转义

```javascript
CharacterEscape :: IdentityEscape

/**
 * 返回字符本身
 */
```

### CharacterClassEscape

#### 十进制数字

```javascript
CharacterClassEscape :: d

/**
 * 返回一个字符集，该字符集为0-9
 */
return new CharSet('[0-9]')
```

#### 非十进制数字

```javascript
CharacterClassEscape :: D

/**
 * 返回一个字符集，该字符集为0-9之外的所有字符
 */
return new CharSet('[^0-9]')
```

#### 空白符

```javascript
CharacterClassEscape :: s

/**
 * 返回一个字符集，该字符集为空白字符或行结束符
 */
return new CharSet(WhiteSpace, LineTerminator)
```

#### 非空白符

```javascript
CharacterClassEscape :: S

/**
 * 返回一个字符集，该字符集为空白字符、行结束符之外的所有字符
 */
return new CharSet(WhiteSpace, LineTerminator, except)
```

#### 单词

```javascript
CharacterClassEscape :: w

/**
 * 返回一个字符集，该字符集为a-zA-Z0-9_
 */
return new CharSet('[a-zA-Z0-9_]')
```

#### 非单词

```javascript
CharacterClassEscape :: W

/**
 * 返回一个字符集，该字符集为a-zA-z0-9_之外的所有字符
 */
return new CharSet('[^a-zA-z0-9_]')
```

### CharacterClass

#### 类范围

```javascript
CharacterClass :: [ [lookahead ∉ {^}] ClassRanges ]

/**
 * 执行ClassRanges获取字符集A
 * 返回{A, invert: false}
 */
let A = evaluate(ClassRanges)
let invert = false
return {A, invert}
```

#### 类范围之外

```javascript
CharacterClass :: [ ^ ClassRanges ]

/**
 * 执行ClassRanges获取字符集A
 * 返回{A, invert: true}
 */
let A = evaluate(ClassRanges)
let invert = true
return {A, invert}
```

### ClassRanges

#### 空范围

```javascript
ClassRanges :: [empty]

/**
 * 返回空的字符集
 */
return new CharSet()
```

#### 非空类范围

```javascript
ClassRanges :: NonemptyClassRanges

/**
 * 执行NonemptyClassRanges获取字符集，返回该字符集
 */
let A = evaluate(NonemptyClassRanges)
return A
```

### NonemptyClassRanges

#### 类原子

```javascript
NonemptyClassRanges :: ClassAtom

/**
 * 执行ClassAtom获取字符集，返回该字符集
 */
let A = evaluate(ClassAtom)
return A
```

#### 多个类原子

```javascript
NonemptyClassRanges :: ClassAtom NonemptyClassRangesNoDash

/**
 * 执行ClassAtom获取字符集A
 * 执行NonemptyClassRangesNoDash获取字符集B
 * 返回A与B联合成的字符集
 */
let A = evaluate(ClassAtom)
let B = evaluate(NonemptyClassRangesNoDash)
return A.union(B)
```

#### 类原子范围

```javascript
NonemptyClassRanges :: ClassAtom - ClassAtom ClassRanges

/**
 * 执行第一个ClassAtom获取字符集A
 * 执行第二个ClassAtom获取字符集B
 * 执行ClassRanges获取字符集C
 * 只含单个字符的字符集A、B组成字符集范围D（Unicode码范围）
 * 返回D与C联合成的字符集
 */
let A = evaluate(FirstClassAtom)
let B = evaluate(SecondClassAtom)
let C = evaluate(ClassRanges)
let D = CharacterRange(A, B)

function CharacterRange (A, B) {
  if (A.length !== 1 || B.length !== 1) {
    throw SyntaxError 
  }
  
  let a = A[0]
  let b = B[0]
  let i = a.charCodeAt(0)
  let j = b.charCodeAt(0)
  
  if (i > j) {
    throw SyntaxError
  } else {
    return new CharSet(i, j)
  }
}

return D.union(C)
```

### ClassAtom

#### 连字符类原子

```javascript
ClassAtom :: -

/**
 * 返回只含连字符的字符集
 */
return new CharSet('-')
```

#### 非连字符类原子

```javascript
ClassAtom :: ClassAtomNoDash

/**
 * 执行ClassAtomNoDash获取字符集，返回该字符集
 */
let A = evaluate(ClassAtomNoDash)
return A
```

### ClassAtomNoDash

#### 普通字符类原子

```javascript
ClassAtomNoDash :: SourceCharacter but not one of \ or ] or -

/**
 * 执行SourceCharacter获取普通字符，返回只含该字符的字符集
 */
let ch = evaluate(SourceCharacter)
let A = new CharSet(ch)
return A
```

#### 转义类原子

```javascript
ClassAtomNoDash :: \ ClassEscape

/**
 * 执行ClassEscape获取字符集，返回该字符集
 */
let A = evaluate(ClassEscape)
return A
```

### ClassEscape

#### 十进制转义

```javascript
ClassEscape :: DecimalEscape

/**
 * 执行DecimalEscape获取结果E
 * E为<NUL>字符，返回只含该字符的字符集匹配
 * E不为<NUL>字符，抛出SyntaxError
 */
let E = evaluate(DecimalEscape)
if (E !== '\u0000') {
  throw SyntaxError 
} else {
  return new CharSet('\u0000')
}
```

#### 退格转义

```javascript
ClassEscape :: b

/**
 * 返回只含退格符的字符集
 */
let A = evaluate('\b')
return A
```

#### 字符转义

```javascript
ClassEscape :: CharacterEscape

/**
 * 执行CharacterEscape获取字符集，返回该字符集
 */
let A = evaluate(CharacterEscape)
return A
```

#### 字符类转义

```javascript
ClassEscape :: CharacterClassEscape

/**
 * 执行CharacterClassEscape获取字符集，返回该字符集
 */
let A = evaluate(CharacterClassEscape)
return A
```

## 构造函数

### RegExp(pattern, flags)

作为函数使用，返回RegExp对象

```javascript
/**
 * pattern为RegExp对象且flags为undefined，直接返回pattern
 * 其他情况，调用new RegExp生成RegExp对象
 */
if (Object.prototype.toString.call(pattern) === '[object RegExp]' && flags === undefined) {
  return pattern
} else {
  return new RegExp(pattern, flags)
}
```

示例

```javascript
RegExp(/a/) // /a/
RegExp(/a/, undefined) // /a/
RegExp(/a/, 'i') // /a/i
```

### new RegExp(pattern, flags)

作为构造器使用，创建RegExp对象

```javascript
/**
 * 参数处理
 * pattern为RegExp对象时，置为该对象的Pattern
 * pattern为undefined时，置为空字符串
 * pattern为其他时，转换为字符串
 * flags为undefined时，置为空字符串
 * flags为其他时，转换为字符串
 * flags含'g', 'i', 'm'之外的字符或含多个'g', 'i', 'm'时，抛出SyntaxError
 * 
 * 创建RegExp对象
 * [[Class]]属性为'RegExp'
 * [[Prototype]]属性为RegExp.prototype
 * [[Extensible]]属性为true
 * [[Match]]方法为执行pattern获取的字符串匹配函数
 * source属性为pattern转义后的字符串（对/, 空字符串转义）
 * global属性为flags是否含'g'
 * ignorecase属性为flags是否含'i'
 * multiline属性为flags是否含'm'
 * lastIndex属性为0
 */
let P
if (Object.prototype.toString.call(pattern) === '[object RegExp]') {
  P = pattern.pattern
} else {
  P = pattern === undefined ? '' : ToString(pattern)
}

let F = flags === undefined ? '' : ToString(flags)
if (!IsConsistOf(F, 'g', 'i', 'm')) {
  throw SyntaxError
}

let reg = new Object()
reg.[[Class]] = 'RegExp'
reg.[[Prototype]] = RegExp.prototype
reg.[[Extensible]] = true
reg.[[Match]] = evaluate(P)

reg.source = Escape(P)
reg.global = F.indexOf('g') !== -1
reg.ignorecase = F.indexOf('i') !== -1
reg.multiline  = F.indexOf('m') !== -1
reg.lastIndex = 0
```

示例

```javascript
new RegExp(/a/) // /a/
new RegExp() // /(?:)/
new RegExp('a') // /a/
new RegExp(/a/, 'i') // /a/i
new RegExp('a', 'a') // 抛出SyntaxError
new RegExp('a', 'gg') // 抛出SyntaxError

new RegExp('/', 'gim')
// {
//   source: '\/',
//   global: true,
//   ignorecase: true,
//   multiline: true,
//   lastIndex: 0
// }
```

### 静态属性

```javascript
RegExp.[[Class]] = 'Function'
RegExp.[[Prototype]] = Function.Prototype
RegExp.[[Extensible]] = true

RegExp.[[DefineOwnProperty]]('length', {
  [[Value]]: 2,
  [[Writable]]: false,
  [[Enumerable]]: false,
  [[Configurable]]: false
}, false)
RegExp.[[DefineOwnProperty]]('prototype', {
  [[Value]]: {constructor: RegExp,...},
  [[Writable]]: false,
  [[Enumerable]]: false,
  [[Configurable]]: false
}, false)
```

## 原型对象

RegExp对象的原型

```javascript
RegExp.prototype
```

### 原型属性

```javascript
RegExp.prototype.[[Class]] = 'RegExp'
RegExp.prototype.[[Prototype]] = Object.prototype
RegExp.prototype.[[Extensible]] = true
RegExp.prototype.[[Match]] = evaluate('')

RegExp.prototype.constructor = RegExp
RegExp.prototype.source = '(?:)'
RegExp.prototype.global = false
RegExp.prototype.ignorecase = false
RegExp.prototype.multiline = false
RegExp.prototype.lastIndex = 0
```

### 原型方法

**转换方法**

#### RegExp.prototype.toString()

获取RegExp对象的字符串表示

```javascript
/**
 * this不为RegExp对象时，抛出TypeError
 * this为RegExp对象时，返回/source/flags形式的字符串
 *    flags的顺序为'gim'
 */
if (Object.prototype.toString.call(pattern) !== '[object RegExp]') {
  throw TypeError
}
let source = R.[[Get]]('source')
let global = R.[[Get]]('global')
let ignorecase = R.[[Get]]('ignorecase')
let multiline = R.[[Get]]('multiline')

let str = '/' + source + '/'
if (global) {
  str += 'g'
}
if (ignorecase) {
  str += 'i'
}
if (multiline) {
  str += 'm'
}

return str
```

示例

```javascript
RegExp.prototype.toString.call(undefined) // 抛出TypeError

new RegExp('', 'img') // /(?:)/gim
new RegExp('a', 'ig') // /a/gi
```

**匹配方法**

#### RegExp.prototype.exec(string)

获取匹配信息

```javascript
/**
 * this不为RegExp对象时，抛出TypeError
 * string转换为String类型
 * 非全局匹配，从0开始匹配，lastIndex属性不变
 * 全局匹配，从lastIndex开始匹配（lastIndex转换为整数）
 *    匹配成功时，将lastIndex属性置为endIndex
 *    匹配不成功时，将lastIndex属性置为0
 * 能匹配到，返回一个包含匹配信息的数组
 *    第一个元素为匹配字符串，其他元素为捕获组匹配
 *    Input属性为string
 *    Index属性为匹配成功的位置
 * 不能匹配到，返回null
 */
if (Object.prototype.toString.call(pattern) !== '[object RegExp]') {
  throw TypeError
}

let R = this
let S = ToString(string)
let length = S.length

let lastIndex = R.[[Get]]('lastIndex')
let global = R.[[Get]]('global')

let i = ToInteger(lastIndex)
if (global === false) {
  i = 0
}
let r

while (true) {
  if (i < 0 || i > length) {
    R.[[Put]]('lastIndex', 0, true)
    return null
  }
  r = R.[[Match]](S, i)
  if (r === failure) {
    i++
  } else {
    break
  }
}

let e = r.endIndex
let cap = r.captures
let n = cap.length

if (global === true) {
  R.[[Put]]('lastIndex', e, true)
}

let A = new Array(n + 1)
A.[[DefineOwnProperty]('0', {
  [[Value]]: S.slice(i, e),
  [[Writable]]: true,
  [[Enumerable]]: true,
  [[Configurable]]: true 
})
for (let j = 0; j < n; j++) {
  A.[[DefineOwnProperty](ToString(j + 1), {
    [[Value]]: cap[j],
    [[Writable]]: true,
    [[Enumerable]]: true,
    [[Configurable]]: true
  })
}
A.[[DefineOwnProperty]('Input', {
  [[Value]]: S,
  [[Writable]]: true,
  [[Enumerable]]: true,
  [[Configurable]]: true 
})
A.[[DefineOwnProperty]('Index', {
  [[Value]]: i,
  [[Writable]]: true,
  [[Enumerable]]: true,
  [[Configurable]]: true 
})
```

示例

```javascript
RegExp.prototype.exec.call(undefined) // 抛出TypeError

// 非全局匹配
/(.)at/.exec('at') // null
/(.)at/.exec('cat') // ['cat', 'c', index: 0, input: 'cat']

// 全局匹配
let reg = /(.)at/g

reg.exec('cat') // ['cat', 'c', index: 0, input: 'cat']
reg.lastIndex // 3

reg.exec('cat') // null
reg.lastIndex // 0
```

#### RegExp.prototype.test(string)

```javascript
/**
 * 调用RegExp.prototype.exec方法
 *    结果为null, 返回false
 *    结果不为null, 返回true
 */
let match = RegExp.prototype.exec.[[Call]](this, string)
return match !== null
```

示例

```javascript
RegExp.prototype.test.call(undefined) // 抛出TypeError

// 非全局匹配
/(.)at/.test('at') // false
/(.)at/.test('cat') // true

// 全局匹配
let reg = /(.)at/g

reg.test('cat') // true
reg.lastIndex // 3

reg.test('cat') // false
reg.lastIndex // 0
```

## 实例对象

[[Class]]为RegExp的对象

```javascript
// 通过正则直接量创建
/a/

// 通过构造器RegExp创建
new RegExp('a', 'i')
```

### 实例属性

```javascript
regexp.[[Class]] = 'RegExp'
regexp.[[Prototype]] = RegExp.prototype
regexp.[[Extensible]] = true
regexp.[[Match]] = evaluate(pattern)
```

#### source

pattern转义字符串

```javascript
regexp.[[DefineOwnProperty]]('source', {
  [[Value]]: source,
  [[Writable]]: false,
  [[Enumerable]]: false,
  [[Configurable]]: false
})
```

#### global

是否是全局匹配

```javascript
regexp.[[DefineOwnProperty]]('global', {
  [[Value]]: global,
  [[Writable]]: false,
  [[Enumerable]]: false,
  [[Configurable]]: false
})
```

#### ignorecase

是否忽略大小写

```javascript
regexp.[[DefineOwnProperty]]('ignorecase', {
  [[Value]]: ignorecase,
  [[Writable]]: false,
  [[Enumerable]]: false,
  [[Configurable]]: false
})
```

#### multiline

是否为多行匹配

```javascript
regexp.[[DefineOwnProperty]]('multiline', {
  [[Value]]: multiline,
  [[Writable]]: false,
  [[Enumerable]]: false,
  [[Configurable]]: false
})
```

#### lastIndex

下次匹配开始的位置

```javascript
regexp.[[DefineOwnProperty]]('lastIndex', {
  [[Value]]: lastIndex,
  [[Writable]]: true,
  [[Enumerable]]: false,
  [[Configurable]]: false
})
```
