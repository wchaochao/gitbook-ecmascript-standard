# 词法

标签（空格分隔）： ECMAScript规范

---

## 源代码文本

```
SourceCharacter :: // 源代码字符
  any Unicode code unit // 任意Unicode单元
```

* 字符集：Unicode字符集
* 编码方式：USC-2

## 输入元素序列

源代码文本会被首先转换为输入序列

### InputElementDiv

除法输入元素

```
InputElementDiv :: // 除法输入元素
  WhiteSpace // 空白符
  LineTerminator // 行终结符
  Comment // 注释
  Token // Token
  DivPunctuator // 除法符号
```

### InputElementRegExp

正则输入元素

```
InputElementRegExp :: // 正则输入元素
  WhiteSpace // 空白符
  LineTerminator // 行终结符
  Comment // 注释
  Token // Token
  RegularExpressionLiteral // 正则字面量
```

## Unicode格式控制字符

用于控制文本解释或者显示，这些字符不可见或不占空间

| 字符编码值 | 名称 | 符号 | 用途 |
| --- | --- | --- | --- |
| \u200C | Zero width non-joiner 零宽不连字符 | `<ZWNJ>` | 抑制连字效果 |
| \u200D | Zero width joiner 零宽连字符 | `<ZWJ>` | 产生连字效果 |
| \uFEFF | Byte Order Mark 字节顺序标记符 | `<BOM>` | 在开头用于指示字节编码方式<br/>在中间为零宽度非换行空格 |

## 空白字符

改善源文本的可读性和分隔 tokens

| 字符编码值 | 名称 | 符号 |
| --- | --- | --- |
| \u0009 | Tab 制表符 | `<TAB>` |
| \u000B | Vertical Tab 垂直制表符 | `<VT>` |
| \u000C | Form Feed 换页符 | `<FF>` |
| \u0020 | Space 空格符 | `<SP>` |
| \u00A0 | No-break space 不换行空格符 | `<NBSP>` |
| \uFEFF | Byte Order Mark 零宽度非换行空格符 | `<BOM>` |
| Other category “Zs” | Any other Unicode “space separator” 其它空格分隔符 | `<USP>` |

```
WhiteSpace :: // 空白字符
  <TAB> // 制表符
  <VT> // 垂直制表符
  <FF> // 换页符
  <SP> // 空格符
  <NBSP> // 不换行空格符
  <BOM> // 零宽度非换行空格符
  <USP> // 其它空格分隔符
```

## 行终结符

改善源文本的可读性和分隔 tokens，会影响语法文法和自动分号插入

| 字符编码值 | 名称 | 符号 |
| --- | --- | --- |
| \u000A | Line Feed 换行符 | `<LF>` |
| \u000D | Carriage Return 回车符 | `<CR>` |
| \u2028 | Line separator 行分隔符 | `<LS>` |
| \u2029 | Paragraph separator 段分隔符 | `<PS>` |

```
LineTerminator :: // 行终结符 
  <LF> // 换行符
  <CR> // 回车符
  <LS> // 行分隔符
  <PS> // 段分隔符

LineTerminatorSequence :: // 行终结符系列
  <LF> // 换行符
  <CR> [lookahead ∉ <LF> ] // 回车不换行
  <CR> <LF> // 回车换行
  <LS> // 行分隔符
  <PS> // 段分隔符
```

## 注释

包括单行注释和多行注释

```
Comment :: // 注释
  SingleLineComment // 单行注释
  MultiLineComment // 多行注释
```

### 单行注释

// 非行终结符

```
SingleLineComment :: // 单行注释
  // SingleLineCommentChars(opt) // + 单行注释字符

SingleLineCommentChars :: // 单行注释字符
  SingleLineCommentChar SingleLineCommentChars(opt) // 一个或多个注释字符

SingleLineCommentChar :: // 单个注释字符
  SourceCharacter but not LineTerminator // 非行终结符的源代码字符
```

示例

```javascript
// 单行注释
```

### 多行注释

/* 非*/字符 */

```
MultiLineComment :: // 多行注释
  /* MultiLineCommentChars(opt) */ // /* 多行注释字符 */

MultiLineCommentChars :: // 多行注释字符
  MultiLineNotAsteriskChar MultiLineCommentChars(opt) // 非*字符 + 多行注释字符
  * PostAsteriskCommentChars(opt) // * + *后注释字符

MultiLineNotAsteriskChar :: // 非*字符
  SourceCharacter but not *

PostAsteriskCommentChars :: // *后注释字符
  MultiLineNotForwardSlashOrAsteriskChar MultiLineCommentChars(opt) // 非*/字符 + 多行注释字符
  * PostAsteriskCommentChars(opt) // * + *后注释字符

MultiLineNotForwardSlashOrAsteriskChar :: // 非*/字符
  SourceCharacter but not one of / or *
```

示例

```javascript
/**
 * 多行注释
 */
```

## Tokens

```
Token ::
  IdentifierName // 标识符名
  Punctuator // 符号
  NumericLiteral // 数字字面量
  StringLiteral // 字符串字面量
```

## 标识符

非保留字的标识符名

```
Identifier :: // 标识符
  IdentifierName but not ReservedWord // 非保留字的标识符名

IdentifierName :: // 标识符名
  IdentifierStart // 标识符开头字符
  IdentifierName IdentifierPart // 标识符开头字符 + 标识符字符

IdentifierStart :: // 标识符开头字符
  UnicodeLetter // Unicode字母
  $ // $字符
  _ // _字符
  \ UnicodeEscapeSequence // Unicode转义系列

UnicodeLetter :: // Unicode字母
   any character in the Unicode categories “Uppercase letter (Lu)”, “Lowercase letter (Ll)”, “Titlecase letter (Lt)”, “Modifier letter (Lm)”, “Other letter (Lo)”, or “Letter number (Nl)”.

UnicodeEscapeSequence :: // Unicode转义系列
  u HexDigit HexDigit HexDigit HexDigit

IdentifierPart :: // 标识符字符
  IdentifierStart // 标识符开头字符
  UnicodeDigit // Unicode十进制数字
  UnicodeCombiningMark // Unicode结合字符
  UnicodeConnectorPunctuation // Unicode连接字符
  <ZWNJ> // 零宽非连接符
  <ZWJ> // 零宽连接符

UnicodeDigit :: // Unicode十进制数字
  any character in the Unicode category “Decimal number (Nd)”
  
UnicodeCombiningMark :: // Unicode结合符
  any character in the Unicode categories “Non-spacing mark (Mn)” or “Combining spacing mark (Mc)”

UnicodeConnectorPunctuation :: // Unicode连接符
  any character in the Unicode category “Connector punctuation (Pc)”
```

## 保留字

不能作为标识符的标识符名

```
ReservedWord :: // 保留字
  Keyword // 关键字
  FutureReservedWord // 未来保留字
  NullLiteral // 空值字面量
  BooleanLiteral // 布尔字面量
```

### 关键字

有特定意义的词

```
Keyword :: one of
  break	    do	      instanceof  typeof
  case	    else	  new	      var
  catch	    finally	  return	  void
  continue	for	      switch	  while
  debugger	function  this	      with
  default	if	      throw	
  delete	in	      try
```

### 未来保留字

可能成为关键字的词

```
FutureReservedWord :: one of
  class	      enum	   extends	  super
  const	      export   import
  implements  let	   private	  public	yield
  interface	  package  protected  static	
```

## 符号

```
Punctuator :: one of
  {	  }	   (	)	[	]
  .	  ;	   ,	<	>	<=
  >=  ==   !=	===	!==	
  +	  -	   *    %	++	--
  <<  >>   >>>	&	|	^
  !	  ~	   &&	||	?	:
  =	  +=   -=	*=	%=	<<=
  >>= >>>= &=	|=	^=	

DivPunctuator :: one of
  /	 /=				
```

## 字面量

```
Literal :: // 字面量
  NullLiteral // 空值字面量
  BooleanLiteral // 布尔值字面量
  NumericLiteral // 数字字面量
  StringLiteral // 字符串字面量
  RegularExpressionLiteral // 正则字面量
```

### 空值字面量

```
NullLiteral ::
  null
```

### 布尔值字面量

```
BooleanLiteral ::
  true
  false
```

### 数字字面量

十进制数字或十六进制整数

```
NumericLiteral :: // 数字字面量
  DecimalLiteral // 十进制字面量
  HexIntegerLiteral // 十六进制整数字面量

DecimalLiteral :: // 十进制字面量
  DecimalIntegerLiteral . DecimalDigits(opt) ExponentPart(opt) // 十进制整数字面量.十进制数字 指数部分
  . DecimalDigits ExponentPart(opt) // .十进制数字 指数部分
  DecimalIntegerLiteral ExponentPart(opt) // 十进制整数字面量 指数部分

DecimalIntegerLiteral :: // 十进制整数字面量
  0 // 0
  NonZeroDigit DecimalDigits(opt) // 非0数字 + 十进制数字

NonZeroDigit :: one of // 非0数字
  1 2 3 4 5 6 7 8 9

DecimalDigits :: // 十进制数字
  DecimalDigit // 单个十进制数字
  DecimalDigits DecimalDigit // 多个十进制数字

DecimalDigit :: one of // 单个十进制数字
  0 1 2 3 4 5 6 7 8 9

ExponentPart :: // 指数部分
  ExponentIndicator SignedInteger // 指数字符 + 带符号整数

ExponentIndicator :: one of // 指数字符
  e E

SignedInteger :: // 带符号整数
  DecimalDigits // 十进制数字
  + DecimalDigits // 正十进制数字
  - DecimalDigits // 负十进制数字

HexIntegerLiteral :: // 十六进制整数字面量
  0x HexDigit // 0x + 单个十六进制
  0X HexDigit // 0X + 单个十六进制
  HexIntegerLiteral HexDigit // 0x|0X + 多个十六进制

HexDigit :: one of // 单个十六进制
  0 1 2 3 4 5 6 7 8 9 a b c d e f A B C D E F
```

示例

```javascript
// 十进制
100
100.23
100.23e3
.23
.23e+3
100e-3

// 十六进制
0x1a
0X1a
```

八进制数字，兼容以前的版本（禁止在严格模式下使用）

```
NumericLiteral ::
  DecimalLiteral
  HexIntegerLiteral
  OctalIntegerLiteral

OctalIntegerLiteral ::
  0 OctalDigit
  OctalIntegerLiteral OctalDigit

OctalDigit :: one of
  0 1 2 3 4 5 6 7
```

示例

```javascript
012 // 10
```

### 字符串字面量

"非双引号字符"或'非单引号字符'

```
StringLiteral :: // 字符串字面量
  " DoubleStringCharacters(opt) " // "双引号字符串字符"
  ' SingleStringCharacters(opt) ' // '单引号字符串字符'

DoubleStringCharacters :: // 双引号字符串字符
  DoubleStringCharacter DoubleStringCharacters(opt) // 单个或多个双引号字符串字符

DoubleStringCharacter :: // 单个双引号字符串字符
  SourceCharacter but not one of " or \ or LineTerminator // 非"\行终结符
  \ EscapeSequence // \ + 转义系列
  LineContinuation // 行延续符

EscapeSequence :: // 转义系列
  CharacterEscapeSequence // 字符转义系列
  0 [lookahead ∉ DecimalDigit] // 0转义
  HexEscapeSequence // 十六进制转义系列
  UnicodeEscapeSequence // Unicode转义系列

CharacterEscapeSequence :: // 字符转义系列
  SingleEscapeCharacter // 单个转义字符
  NonEscapeCharacter // 非转义字符

SingleEscapeCharacter :: one of // 单个转义字符
  ' " \ b f n r t v

NonEscapeCharacter :: // 非转义字符
  SourceCharacter but not one of EscapeCharacter or LineTerminator

EscapeCharacter :: // 转义字符
  SingleEscapeCharacter // 单个转义字符
  DecimalDigit // 十进制数字
  x // 十六进制转义系列的x
  u // Unicode转义系列的u

HexEscapeSequence :: // 十六进制转义系列
  x HexDigit HexDigit // x + 两位十六进制

UnicodeEscapeSequence :: // Unicode转义系列
  u HexDigit HexDigit HexDigit HexDigit // u + 四位十六进制

LineContinuation :: // 行延续符
  \ LineTerminatorSequence // \ + 行终结符系列

SingleStringCharacters :: // 单引号字符串字符
  SingleStringCharacter SingleStringCharacters(opt) // 单个或多个单引号字符串字符

SingleStringCharacter :: // 单个单引号字符串字符
  SourceCharacter but not one of ' or \ or LineTerminator // 非'\行终结符
  \ EscapeSequence // \ + 转义系列
  LineContinuation // 行延续符
```

转义字符

 * `\0`: `NUL`字符
 * `\r`: 回车符
 * `\n`: 换行符
 * `\t`: 水平制表符
 * `\v`: 垂直制表符
 * `\b`: 退格符
 * `\f`: 换页符
 * `\'`: 单引号
 * `\"`: 双引号
 * `\\`: 反斜杠
 * `\x两位十六进制`: 对应的ASCII码
 * `\u四位十六进制`: 对应的Unicode字符
 * `\其他`: 忽略\

示例

```javascript
// 普通字符串
'a'

// 转义序列
'\''
'\a'
'\0a'
'\x1a'
'\u001a'

// 行延续符
'ab\
c'
```

八进制转义，兼容以前版本（禁止在严格模式下使用）

```
EscapeSequence ::
  CharacterEscapeSequence
  OctalEscapeSequence
  HexEscapeSequence
  UnicodeEscapeSequence

OctalEscapeSequence ::
  OctalDigit [lookahead ∉ DecimalDigit]
  ZeroToThree OctalDigit [lookahead ∉ DecimalDigit]
  FourToSeven OctalDigit
  ZeroToThree OctalDigit OctalDigit

ZeroToThree :: one of
  0 1 2 3

FourToSeven :: one of
  4 5 6 7
```

示例

```javascript
'\1'
'\11'
'\111'
'\41'
```
 
### 正则字面量

/pattern/flag, 会自动转换为 RegExp 对象

```
RegularExpressionLiteral :: // 正则字面量
  / RegularExpressionBody / RegularExpressionFlags // /正则主体/正则标志 

RegularExpressionBody :: // 正则主体
  RegularExpressionFirstChar RegularExpressionChars // 第一个正则字符 + 正则字符

RegularExpressionFirstChar :: // 第一个正则字符
  RegularExpressionNonTerminator but not one of * or / or \  or [ // 非*/\[行终结符
  RegularExpressionBackslashSequence // 正则转义系列
  RegularExpressionClass // 正则类

RegularExpressionNonTerminator :: // 非行终结符
  SourceCharacter but not LineTerminator

RegularExpressionBackslashSequence :: // 正则转义系列
  \ RegularExpressionNonTerminator // \ + 非行终结符

RegularExpressionClass :: // 正则类
  [ RegularExpressionClassChars ] // [正则类字符]

RegularExpressionClassChars :: // 正则类字符
  [empty] // 空
  RegularExpressionClassChars RegularExpressionClassChar // 单个或多个正则类字符

RegularExpressionClassChar :: // 单个正则类字符
  RegularExpressionNonTerminator but not one of ] or \ // 非]\行终结符
  RegularExpressionBackslashSequence // 正则转义系列

RegularExpressionChars :: // 正则字符
  [empty] // 空
  RegularExpressionChars RegularExpressionChar // 单个或多个正则字符

RegularExpressionChar :: // 单个正则字符
  RegularExpressionNonTerminator but not one of / or \ or [ // 非/\[行终结符
  RegularExpressionBackslashSequence // 正则转义系列
  RegularExpressionClass // 正则类

RegularExpressionFlags :: // 正则标志
  [empty] // 空
  RegularExpressionFlags IdentifierPart // 单个或多个正则标志
```

示例

```javascript
// 空正则
/(?:)/i

// 正则转义
/\(/g

// 正则类
/[0-9]/m
```

## 自动分号插入

省略分号时自动插入分号用于终止语句

* 非法Token为`{`时，在非法Token前插入分号
* 非法Token前为行终结符时，在非法Token前插入分号
* 输入流结束且无法解析成完整程序时，在结束位置插入分号
* 受限产生式被行结束符分隔时，在受限符前插入分号
* 自动插入分号后为空语句或for语句的头两个分号时不插入

受限产生式

```
PostfixExpression : // 后缀表达式
  LeftHandSideExpression [no LineTerminator here] ++
  LeftHandSideExpression [no LineTerminator here] --

ContinueStatement : // continue语句
  continue [no LineTerminator here] Identifier ;

BreakStatement : // break语句
  break [no LineTerminator here] Identifier ;

ReturnStatement : // return语句
  return [no LineTerminator here] Expression ;

ThrowStatement : // throw语句
  throw [no LineTerminator here] Expression ;
```

示例

```javascript
// 源文本
{ 1
2 } 3

// 自动插入分号
{ 1
;2 ;} 3;

// 源文本
return
a + b

// 自动插入分号
return;
a + b;

// 源文本
a = b
++c

// 自动插入分号
a = b;
++c;

// 源文本
a = b + c
(d + e).print()

// 自动插入分号
a = b + c(d + e).print()
```
