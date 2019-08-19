# Date构造器

标签（空格分隔）： ECMAScript规范

---

## 表示

* 时间值：代表时间的一个数字
 * 任一数字表示距1970年1月1日00:00:00(UTC时间)的毫秒数，范围为前后1亿天
 * NaN表示非时间
* 年：时间值对应的年份
* 月：时间值对应的月份，[0, 11]的整数
* 日：时间值对应的几号，[1, 31]的整数
* 星期：时间值对应的星期几，[0, 6]的整数，星期天为0
* 时：时间值对应的时，[0, 23]的整数
* 分：时间值对应的分，[0, 59]的整数
* 秒：时间值对应的秒，[0, 59]的整数
* 毫秒：时间值对应的毫秒，[0, 999]的整数
* 夏时制时间：Daylight Saving Time，实行夏时制的地区会在夏季将时间拨快1小时，在秋季调整回来
* 本地时间：本地时间 = UTC时间 + 时区 + 夏时制时间调整

## 标准时间格式

`YYYY-MM-DDTHH:MM:ss.sssZ`

* `YYYY`: 0000 - 9999, 四位数的年
* `MM`: 01 - 12, 两位数的月，默认为01
* `DD`: 01 - 31, 两位数的天，默认为01
* `T`: 分隔日期和时间
* `HH`: 00 - 23, 两位数的时，默认为00
* `MM`: 00 - 59, 两位数的分，默认为00
* `ss`: 00 - 59, 两位数的秒，默认为00
* `sss`: 00 - 999, 三位数的毫秒，默认为000
* `Z`: 时区，默认为单`Z`
 * 单`Z`: UTC时间
 * `Z+HH:mm`: 东时区
 * `Z-HH:mm`: 西时区

日期格式

* `YYYY`
* `YYYY-MM`
* `YYYY-MM-DD`

日期后的时间格式

* `THH:mm`
* `THH:mm:ss`
* `THH:mm:ss.sss`

## 操作

### Day(t)

获取天数

```
msPerDay = 86400000

Day(t) = floor(t / msPerDay)
TimeWithinDay(t) = t modulo msPerDay
```

实现

```javascript
const msPerDay = 86400000

function Day (t) {
  return Math.floor(t / msPerDay)
}

function TimeWithinDay (t) {
  return t % msPerDay
}
```

### YearFromTime(t)

获取年

```
DaysInYear(y) = 365 if (y modulo 4) ≠ 0
= 366 if (y modulo 4) = 0 and (y modulo 100) ≠ 0
= 365 if (y modulo 100) = 0 and (y modulo 400) ≠ 0
= 366 if (y modulo 400) = 0

DayFromYear(y)	= 365 × (y − 1970) + floor((y − 1969) / 4) − floor((y − 1901) / 100) + floor((y − 1601) / 400)

TimeFromYear(y)	= msPerDay × DayFromYear(y)

YearFromTime(t)	= the largest integer y (closest to positive infinity) such that TimeFromYear(y) ≤ t

InLeapYear(t) = 0 if DaysInYear(YearFromTime(t)) = 365
= 1 if DaysInYear(YearFromTime(t)) = 366
```

实现

```javascript
function IsLeapYear (y) {
  return (y % 4 === 0 && y % 100 !== 0) || (y % 400 === 0)
}

function DaysInYear (y) {
  return 365 + IsLeapYear(y)
}

function DayFromYear (y) {
  return 365 * (y − 1970) + Math.floor((y − 1969) / 4) − Math.floor((y − 1901) / 100) + Math.floor((y − 1601) / 400)
}

function TimeFromYear (y) {
  return msPerDay * DayFromYear(y)
}

function YearFromTime (t) {
  let start = Math.floor(Day(t) / 365) + 1970
  do {
    let m = TimeFromYear(start)
    if (m < t) {
      start++
    } else if (m === t) {
      return start
    } else {
      return start - 1
    }
  } while (true)
}

function InLeapYear (t) {
  let y = YearFromTime(t)
  return IsLeapYear(y) ? 1 : 0
}
```

### MonthFromTime(t)

获取月

```
DayWithinYear(t) = Day(t) − DayFromYear(YearFromTime(t))

MonthFromTime(t) = 0 if 0 ≤ DayWithinYear(t) < 31
= 1	if 31 ≤ DayWithinYear (t) < 59 + InLeapYear(t)
= 2	if 59 + InLeapYear(t) ≤ DayWithinYear(t) < 90 + InLeapYear(t)
= 3	if 90 + InLeapYear(t) ≤ DayWithinYear(t) < 120 + InLeapYear(t)
= 4	if 120 + InLeapYear(t) ≤ DayWithinYear(t) < 151 + InLeapYear(t)
= 5	if 151 + InLeapYear(t) ≤ DayWithinYear(t) < 181 + InLeapYear(t)
= 6	if 181 + InLeapYear(t) ≤ DayWithinYear(t) < 212 + InLeapYear(t)
= 7	if 212 + InLeapYear(t) ≤ DayWithinYear(t) < 243 + InLeapYear(t)
= 8	if 243 + InLeapYear(t) ≤ DayWithinYear(t) < 273 + InLeapYear(t)
= 9	if 273 + InLeapYear(t) ≤ DayWithinYear(t) < 304 + InLeapYear(t)
= 10 if	304 + InLeapYear(t) ≤ DayWithinYear(t) < 334 + InLeapYear(t)
= 11 if	334 + InLeapYear(t) ≤ DayWithinYear(t) < 365 + InLeapYear(t)
```

实现

```javascript
function DayWithinYear (t) {
  let y = YearFromTime(t)
  return Day(t) - DayFromYear(y)
}

const monthDays = [0, 31, 59, 90, 120, 151, 181, 212, 243, 273, 304, 334]

function InMonth (d, start, end, inStartLeap, inEndLeap) {
  if (typeof inStartLeap === Number) {
    start += inStartLeap
  }
  if (typeof inEndLeap === Number) {
    end += inEndLeap
  }
  return d >= start && d < end
}

function MonthFromTime (t) {
  let d = DayWithinYear(t)
  let inLeap = InLeapYear(t)
  for (let i = 0; i < monthDays.length - 1; i++) {
    if (InMonth(d, monthDays[i], monthDays[i + 1], i > 1 ? inLeap : undefined, i > 0 ? inLeap : undefined)) {
      return i
    }
  }
}
```

### DateFromTime(t)

获取日

```
DateFromTime(t)	= DayWithinYear(t) + 1 if MonthFromTime(t) = 0
= DayWithinYear(t) − 30	if MonthFromTime(t) = 1
= DayWithinYear(t) − 58 − InLeapYear(t)	if MonthFromTime(t) = 2
= DayWithinYear(t) − 89 − InLeapYear(t)	if MonthFromTime(t) = 3
= DayWithinYear(t) − 119 − InLeapYear(t) if MonthFromTime(t) = 4
= DayWithinYear(t) − 150 − InLeapYear(t) if MonthFromTime(t) = 5
= DayWithinYear(t) − 180 − InLeapYear(t) if MonthFromTime(t) = 6
= DayWithinYear(t) − 211 − InLeapYear(t) if MonthFromTime(t) = 7
= DayWithinYear(t) − 242 − InLeapYear(t) if MonthFromTime(t) = 8
= DayWithinYear(t) − 272 − InLeapYear(t) if MonthFromTime(t) = 9
= DayWithinYear(t) − 303 − InLeapYear(t) if MonthFromTime(t) = 10
= DayWithinYear(t) − 333 − InLeapYear(t) if MonthFromTime(t) = 11
```

实现

```javascript
const monthDays = [0, 31, 59, 90, 120, 151, 181, 212, 243, 273, 304, 334]

function DateFromTime (t) {
  let d = DayWithinYear(t)
  let m = MonthFromTime(t)
  let inLeap = InLeapYear(t)
  return d - monthDays[m] + 1 - (m > 1 ? inLeap : 0)
}
```

### WeekDay(t)

获取星期

```
WeekDay(t)	= (Day(t) + 4) modulo 7
```

实现

```javascript
function WeekDay (t) {
  return (Day(t) + 4) % 7
}
```

### HourFromTime(t)

获取时

```
msPerHour = 3600000
HoursPerDay	= 24

HourFromTime(t)	= floor(t / msPerHour) modulo HoursPerDay
```

实现

```javascript
const msPerHour = 3600000
const HoursPerDay = 24

function HourFromTime (t) {
  return Math.floor(t / msPerHour) % HoursPerDay
}
```

### MinFromTime(t)

获取分

```
msPerMinute	= 60000
MinutesPerHour = 60

MinFromTime(t) = floor(t / msPerMinute) modulo MinutesPerHour
```

实现

```javascript
const msPerMinute = 60000
const MinutesPerHour = 60

function MinFromTime (t) {
  return Math.floor(t / msPerMinute) % MinutesPerHour
}
```

### SecFromTime(t)

获取秒

```
msPerSecond	= 1000
SecondsPerMinute = 60

SecFromTime(t) = floor(t / msPerSecond) modulo SecondsPerMinute
```

实现

```javascript
const msPerSecond = 1000
const SecondsPerMinute = 60

function SecFromTime (t) {
  return Math.floor(t / msPerSecond) % SecondsPerMinute
}
```

### msFromTime(t)

获取毫秒

```
msPerSecond	= 1000

msFromTime(t) = t modulo msPerSecond
```

实现

```javascript
const msPerSecond = 1000

function msFromTime (t) {
  return t % msPerSecond
}
```

### LocalTime(t)

本地时间

```
// UTC时间转本地时间
LocalTime(t) = t + LocalTZA + DaylightSavingTA(t)

// 本地时间转UTC时间
UTC(t) = t – LocalTZA – DaylightSavingTA(t – LocalTZA)
```

### MakeTime(hour, min, sec, ms)

获取时、分、秒、毫秒对应的毫秒数

```javascript
/**
 * 时、分、秒、毫秒不为有限值时，返回NaN
 * 时、分、秒、毫秒为有限值时，转换为整数
 * 计算时、分、秒、毫秒对应的毫秒数
 */
let arr = [hour, min, sec, ms]
if (!arr.every(item => isFinite(item))) {
  return NaN
}

let [h, m, s, ms] = arr.map(item => ToInteger(item))
let t = h * msPerHour + m * msPerMinute + s * msPerSecond + ms
return t
```

### MakeDay(year, month, date)

获取年、月、日对应的天数

```javascript
/**
 * 年、月、日不为有限值时，返回NaN
 * 年、月、日为有限值时，转换为整数
 * 计算年、月、日对应的天数
 */
let arr = [year, month, date]
if (!arr.every(item => isFinite(item))) {
  return NaN
}

let [y, m, d] = arr.map(item => ToInteger(item))
let ym = y + Math.floor(m / 12)
let mn = m % 12
return DayFromYearAndMonth(ym, mn) + d
```

### MakeDate(day, time)

获取天数、毫秒数对应的总毫秒数

```javascript
/**
 * 天数、毫秒数不为有限值时，返回NaN
 * 天数、毫秒数为有限值时，转换为总毫秒数
 */
let arr = [day, time]
if (!arr.every(item => isFinite(item))) {
  return NaN
}

return day × msPerDay + time
```

### TimeClip(time)

时间值限制为前后一亿天内的毫秒数

```javascript
/**
 * 时间值不为有限值或超过前后一亿天范围时，返回NaN
 * 其他，转换为整数
 */
if (!isFinite(time) || abs(time) >= 8.64e+15) {
  return NaN
} else {
  return ToInteger(time)
}
```

## 构造函数

### Date([year [, month [, date [, hours [, minutes [, seconds [, ms ] ] ] ] ] ] ] )

作为函数使用，返回当前时间的字符串表示（当前时间为计算机的时间）

```javascript
/**
 * 返回当前时间的字符串表示
 */
return new Date().toString()
```

### new Date()

无参数，作为构造器使用，创建Date对象，时间为当前时间

```javascript
/**
 * 创建Date对象
 * [[Class]]属性为'Date'
 * [[Prototype]]属性为Date.prototype
 * [[Extensible]]属性为true
 * [[PrimitiveValue]]属性为当前时间对应的UTC毫秒数
 */
let date = new Object()
date.[[Class]] = 'Date'
date.[[Prototype]] = Date.prototype
date.[[Extensible]] = true
date.[[PrimitiveValue]] = currentTimeValue
```

### new Date(value)

1个参数，作为构造器使用，创建Date对象，指定UTC毫秒数或时间字符串

```javascript
/**
 * 创建Date对象
 * [[Class]]属性为'Date'
 * [[Prototype]]属性为Date.prototype
 * [[Extensible]]属性为true
 * [[PrimitiveValue]]属性为指定的UTC毫秒数或时间字符串对应的UTC毫秒数，限制前后一亿天内
 *   value转换为Primitive类型
 *      value为字符串时，调用Date.parse获取UTC毫秒数
 *      value为其他，转换为Number类型
 */
let date = new Object()
date.[[Class]] = 'Date'
date.[[Prototype]] = Date.prototype
date.[[Extensible]] = true

let v = ToPrimitive(value)
if (Type(v) === String) {
  v = Date.parse(v)
}
let t = toNumber(v)
return TimeClip(t)
```

示例

```javascript
new Date(undefined) // Invalid Date
new Date(0) // Thu Jan 01 1970 08:00:00 GMT+0800 (中国标准时间)
new Date('2010') // Fri Jan 01 2010 08:00:00 GMT+0800 (中国标准时间)
new Date('2010-01-01') // Fri Jan 01 2010 08:00:00 GMT+0800 (中国标准时间)
new Date('2010-01-01T01:01') // Fri Jan 01 2010 01:01:00 GMT+0800 (中国标准时间)
new Date('2010-01-01T01:01Z') // Fri Jan 01 2010 09:01:00 GMT+0800 (中国标准时间)
new Date('2/3/2010') // Wed Feb 03 2010 00:00:00 GMT+0800 (中国标准时间)
new Date('Febraury 3, 2010') // Wed Feb 03 2010 00:00:00 GMT+0800 (中国标准时间)
```

### new Date(year, month [, date [, hours [, minutes [, seconds [, ms ] ] ] ] ] )

至少2个参数，作为构造器使用，创建Date对象，指定本地时间元素

```javascript
/**
 * 创建Date对象
 * [[Class]]属性为'Date'
 * [[Prototype]]属性为Date.prototype
 * [[Extensible]]属性为true
 * [[PrimitiveValue]]属性为本地时间元素对应的UTC毫秒数，限制前后一亿天内
 *   year转换为Number类型
 *      year转换为整数在[0, 99]之内时，将year置为1900 + ToInteger(year)
 *   month转换为Number类型
 *   date转换为Number类型，默认为1
 *   hours转换为Number类型，默认为0
 *   minutes转换为Number类型，默认为0
 *   seconds转换为Number类型，默认为0
 *   ms转换为Number类型，默认为0
 */
let date = new Object()
date.[[Class]] = 'Date'
date.[[Prototype]] = Date.prototype
date.[[Extensible]] = true

let y = ToNumber(year)
let yt = ToInteger(y)
if (yt >= 0 && yt <= 99) {
  y = 1900 + yt
}

let m = ToNumber(month)
let d = IsPresent(date) ? ToNumber(date) : 1
let h = IsPresent(hours) ? ToNumber(hours) : 0
let m = IsPresent(minutes) ? ToNumber(minutes) : 0
let s = IsPresent(seconds) ? ToNumber(seconds) : 0
ms = IsPresent(ms) ? ToNumber(ms) : 0
let t = MakeDate(MakeDay(y, m, d), MakeTime(h, m, s, ms))

date.[[PrimitiveValue]] = TimeClip(UTC(t))
```

示例

```javascript
new Date(2101, undefined) // Invalid Date
new Date(2010, 1) // Mon Feb 01 2010 00:00:00 GMT+0800 (中国标准时间)
new Date(10, 3) // Fri Apr 01 1910 00:00:00 GMT+0800 (中国标准时间)
new Date(2000, 4, 5, 6, 7, 8, 9) // Fri May 05 2000 06:07:08 GMT+0800 (中国标准时间)
new Date(2002, 16, 32, 25, 26, 80) // Mon Jun 02 2003 01:27:20 GMT+0800 (中国标准时间)
```

### 静态属性

```javascript
Date.[[Class]] = 'Function'
Date.[[Prototype]] = Function.Prototype
Date.[[Extensible]] = true

Date.[[DefineOwnProperty]]('length', {
  [[Value]]: 7,
  [[Writable]]: false,
  [[Enumerable]]: false,
  [[Configurable]]: false
}, false)
Date.[[DefineOwnProperty]]('prototype', {
  [[Value]]: {constructor: Date,...},
  [[Writable]]: false,
  [[Enumerable]]: false,
  [[Configurable]]: false
}, false)
```

### 静态方法

#### Date.now()

获取当前时间对应的UTC毫秒数

```javascript
/**
 * 返回当前时间对应的UTC毫秒数
 */
return currentTimeValue
```

#### Date.parse(string)

将时间字符串解析为UTC毫秒数

```javascript
/**
 * string转换为String类型，再解析为UTC毫秒数
 *   YYYY - UTC时间
 *   YYYY-MM - UTC时间
 *   YYYY-MM-DD - UTC时间
 *   YYYY-MM-DDTHH:mm - 本地时间
 *   YYYY-MM-DDTHH:mm:ss - 本地时间
 *   YYYY-MM-DDTHH:mm:ss.sss - 本地时间
 *   YYYY-MM-DDTHH:mm:ss.sssZ - 时区时间
 *   M/D/YYYY - 本地时间
 *   MMMM D, YYYY - 本地时间
 *   ddd MMMM D YYYY HH:mm:ss - 本地时间
 *   ddd MMMM D YYYY HH:mm:ss Z - UTC时间
 */
let s = ToString(s)
return parse(s)
```

#### Date.UTC(year, month [, date [, hours [, minutes [, seconds [, ms ] ] ] ] ] )

将UTC时间元素转换为UTC毫秒数

```javascript
/**
 * 将UTC时间元素转换为对应的UTC毫秒数，限制前后一亿天内
 *   year转换为Number类型
 *      year转换为整数在[0, 99]之内时，将year置为1900 + ToInteger(year)
 *   month转换为Number类型
 *   date转换为Number类型，默认为1
 *   hours转换为Number类型，默认为0
 *   minutes转换为Number类型，默认为0
 *   seconds转换为Number类型，默认为0
 *   ms转换为Number类型，默认为0
 */
let y = ToNumber(year)
let yt = ToInteger(y)
if (yt >= 0 && yt <= 99) {
  y = 1900 + yt
}

let m = ToNumber(month)
let d = IsPresent(date) ? ToNumber(date) : 1
let h = IsPresent(hours) ? ToNumber(hours) : 0
let m = IsPresent(minutes) ? ToNumber(minutes) : 0
let s = IsPresent(seconds) ? ToNumber(seconds) : 0
ms = IsPresent(ms) ? ToNumber(ms) : 0
let t = MakeDate(MakeDay(y, m, d), MakeTime(h, m, s, ms))

return TimeClip(UTC(t))
```

## 原型对象

Date对象的原型

```javascript
Date.prototype
```

### 原型属性

```javascript
Date.prototype.[[Class]] = 'Date'
Date.prototype.[[Prototype]] = Object.prototype
Date.prototype.[[Extensible]] = true
Date.prototype.[[PrimitiveValue]] = NaN

Date.prototype.constructor = Date
```

### 原型方法

**转换方法**

#### Date.prototype.toString( )

获取Date对象的字符串表示

```javascript
/**
 * this不为Date对象，抛出TypeError
 * this为Date对象，返回Date对象的字符串表示
 *   通常为ddd MMM D YYYY HH:mm:ss GMT+时区格式
 *   非法Date返回'Invalid Date'
 */
if (Object.prototype.toString.call(this) !== '[object Date]') {
  throw TypeError
}
return toString(this)
```

示例

```javascript
Date.prototype.toString.call(undefined) // 抛出TypeError

(new Date(undefined)).toString() // 'Invalid Date'
(new Date('2010-01-01')).toString() // 'Fri Jan 01 2010 08:00:00 GMT+0800 (中国标准时间)'
```

#### Date.prototype.toDateString( )

获取Date对象的日期字符串表示

```javascript
/**
 * this不为Date对象，抛出TypeError
 * this为Date对象，返回Date对象的日期字符串表示
 *   通常为ddd MMM D YYYY格式
 *   非法Date返回'Invalid Date'
 */
if (Object.prototype.toString.call(this) !== '[object Date]') {
  throw TypeError
}
return toDateString(this)
```

示例

```javascript
Date.prototype.toDateString.call(undefined) // 抛出TypeError

(new Date(undefined)).toDateString() // 'Invalid Date'
(new Date('2010-01-01')).toDateString() // 'Fri Jan 01 2010'
```

#### Date.prototype.toTimeString( )

获取Date对象的时间字符串表示

```javascript
/**
 * this不为Date对象，抛出TypeError
 * this为Date对象，返回Date对象的时间字符串表示
 *   通常为HH:mm:ss GMT+时区格式
 *   非法Date返回'Invalid Date'
 */
if (Object.prototype.toString.call(this) !== '[object Date]') {
  throw TypeError
}
return toTimeString(this)
```

示例

```javascript
Date.prototype.toTimeString.call(undefined) // 抛出TypeError

(new Date(undefined)).toTimeString() // 'Invalid Date'
(new Date('2010-01-01')).toTimeString() // '08:00:00 GMT+0800 (中国标准时间)'
```

#### Date.prototype.toLocaleString( )

获取Date对象的本地字符串表示

```javascript
/**
 * this不为Date对象，抛出TypeError
 * this为Date对象，返回Date对象的本地字符串表示
 *   非法Date返回'Invalid Date'
 */
if (Object.prototype.toString.call(this) !== '[object Date]') {
  throw TypeError
}
return toLocaleString(this)
```

示例

```javascript
Date.prototype.toLocaleString.call(undefined) // 抛出TypeError

(new Date(undefined)).toLocaleString() // 'Invalid Date'
(new Date('2010-01-01')).toLocaleString() // '2010/1/1 上午8:00:00'
```

#### Date.prototype.toLocaleDateString( )

获取Date对象的本地日期字符串表示

```javascript
/**
 * this不为Date对象，抛出TypeError
 * this为Date对象，返回Date对象的本地日期字符串表示
 *   非法Date返回'Invalid Date'
 */
if (Object.prototype.toString.call(this) !== '[object Date]') {
  throw TypeError
}
return toLocaleDateString(this)
```

示例

```javascript
Date.prototype.toLocaleDateString.call(undefined) // 抛出TypeError

(new Date(undefined)).toLocaleDateString() // 'Invalid Date'
(new Date('2010-01-01')).toLocaleDateString() // '2010/1/1'
```

#### Date.prototype.toLocaleTimeString( )

获取Date对象的本地时间字符串表示

```javascript
/**
 * this不为Date对象，抛出TypeError
 * this为Date对象，返回Date对象的本地时间字符串表示
 *   非法Date返回'Invalid Date'
 */
if (Object.prototype.toString.call(this) !== '[object Date]') {
  throw TypeError
}
return toLocaleTimeString(this)
```

示例

```javascript
Date.prototype.toLocaleTimeString.call(undefined) // 抛出TypeError

(new Date(undefined)).toLocaleTimeString() // 'Invalid Date'
(new Date('2010-01-01')).toLocaleTimeString() // '上午8:00:00'
```

#### Date.prototype.toUTCString( )

获取Date对象的UTC时间字符串表示

```javascript
/**
 * this不为Date对象，抛出TypeError
 * this为Date对象，返回Date对象的UTC时间字符串表示
 *   通常为ddd, DD MMM YYYY HH:mm:ss GMT格式
 *   非法Date返回'Invalid Date'
 */
if (Object.prototype.toString.call(this) !== '[object Date]') {
  throw TypeError
}
return toUTCString(this)
```

示例

```javascript
Date.prototype.toUTCString.call(undefined) // 抛出TypeError

(new Date(undefined)).toUTCString() // 'Invalid Date'
(new Date('2010-01-01')).toUTCString() // 'Fri, 01 Jan 2010 00:00:00 GMT'
```

#### Date.prototype.toISOString( )

获取Date对象的ISO时间字符串表示

```javascript
/**
 * this不为Date对象，抛出TypeError
 * this为Date对象，返回Date对象的ISO时间字符串表示
 *   通常为YYYY-MM-DDTHH:mm:ss.sssZ格式
 *   非法Date抛出RangeError
 */
if (Object.prototype.toString.call(this) !== '[object Date]') {
  throw TypeError
}
if (SameValue(this.[[PrimitiveValue]], NaN)) {
  throw RangeError
}
return toISOString(this)
```

示例

```javascript
Date.prototype.toISOString.call(undefined) // 抛出TypeError

(new Date(undefined)).toISOString() // 抛出RangeError
(new Date('2010-01-01')).toISOString() // '2010-01-01T00:00:00.000Z'
```

#### Date.prototype.toJSON()

获取Date对象的JSON字符串表示

```javascript
/**
 * this转换为Object对象O，再转换为原始值类型t，偏向Number
 * t为Number类型且不为有限数字，返回null
 * t为其他，调用O的toISOString方法，返回返回值
 *   toISOString方法不存在时，抛出TypeError
 */
let O = ToObject(this)
let t = ToPrimitive(O, 'Number')
if (Type(t) === Number && !IsFinite(t)) {
  return null
}

let toISO = O.[[Get]]('toISOString')
if (!IsCallable(toISO)) {
  throw TypeError
} else {
  return toISO.[[Call]](this)
}
```

示例

```javascript
Date.prototype.toJSON.call(undefined) // 抛出TypeError
Date.prototype.toJSON.call(NaN) // null
Date.prototype.toJSON.call(1) // 抛出TypeError

(new Date(undefined)).toJSON() // null
(new Date('2010-01-01')).toJSON() // '2010-01-01T00:00:00.000Z'
```

#### Date.prototype.valueOf( )

获取Date对象的值表示

```javascript
/**
 * this不为Date对象，抛出TypeError
 * this为Date对象，返回[[PrimitiveValue]]值
 */
if (Object.prototype.toString.call(this) !== '[object Date]') {
  throw TypeError
}
return this.[[PrimitiveValue]]
```

示例

```javascript
Date.prototype.valueOf.call(undefined) // 抛出TypeError

(new Date(undefined)).valueOf() // NaN
(new Date('2010-01-01')).valueOf() // 1262304000000
```

**获取方法**

#### Date.prototype.getTime( )

获取Date对象的时间值

```javascript
/**
 * this不为Date对象，抛出TypeError
 * this为Date对象，返回[[PrimitiveValue]]值
 */
if (Object.prototype.toString.call(this) !== '[object Date]') {
  throw TypeError
}
return this.[[PrimitiveValue]]
```

示例

```javascript
Date.prototype.getTime.call(undefined) // 抛出TypeError

(new Date(undefined)).getTime() // NaN
(new Date('2010-01-01')).getTime() // 1262304000000
```

#### Date.prototype.getFullYear( )

获取Date对象的本地年

```javascript
/**
 * this不为Date对象，抛出TypeError
 * this为Date对象，获取时间值
 *    时间值为NaN，返回NaN
 *    时间值不为NaN，转换为本地时间值，返回对应的年
 */
if (Object.prototype.toString.call(this) !== '[object Date]') {
  throw TypeError
}
let t = this.[[PrimitiveValue]]
if (SameValue(t, NaN)) {
  return NaN
} else {
  return YearFromTime(LocalTime(t))
}
```

示例

```javascript
Date.prototype.getFullYear.call(undefined) // 抛出TypeError

(new Date(undefined)).getFullYear() // NaN
(new Date('2010-01-01T00:00:00')).getFullYear() // 2010
```

#### Date.prototype.getUTCFullYear( )

获取Date对象的UTC年

```javascript
/**
 * this不为Date对象，抛出TypeError
 * this为Date对象，获取时间值
 *    时间值为NaN，返回NaN
 *    时间值不为NaN，返回对应的年
 */
if (Object.prototype.toString.call(this) !== '[object Date]') {
  throw TypeError
}
let t = this.[[PrimitiveValue]]
if (SameValue(t, NaN)) {
  return NaN
} else {
  return YearFromTime(t)
}
```

示例

```javascript
Date.prototype.getUTCFullYear.call(undefined) // 抛出TypeError

(new Date(undefined)).getUTCFullYear() // NaN
(new Date('2010-01-01T00:00')).getUTCFullYear() // 2009
```

#### Date.prototype.getMonth( )

获取Date对象的本地月，[0, 11]的整数

```javascript
/**
 * this不为Date对象，抛出TypeError
 * this为Date对象，获取时间值
 *    时间值为NaN，返回NaN
 *    时间值不为NaN，转换为本地时间值，返回对应的月
 */
if (Object.prototype.toString.call(this) !== '[object Date]') {
  throw TypeError
}
let t = this.[[PrimitiveValue]]
if (SameValue(t, NaN)) {
  return NaN
} else {
  return MonthFromTime(LocalTime(t))
}
```

示例

```javascript
Date.prototype.getMonth.call(undefined) // 抛出TypeError

(new Date(undefined)).getMonth() // NaN
(new Date('2010-01-01T00:00:00')).getMonth() // 0
```

#### Date.prototype.getUTCMonth( )

获取Date对象的UTC月，[0, 11]的整数

```javascript
/**
 * this不为Date对象，抛出TypeError
 * this为Date对象，获取时间值
 *    时间值为NaN，返回NaN
 *    时间值不为NaN，返回对应的月
 */
if (Object.prototype.toString.call(this) !== '[object Date]') {
  throw TypeError
}
let t = this.[[PrimitiveValue]]
if (SameValue(t, NaN)) {
  return NaN
} else {
  return MonthFromTime(t)
}
```

示例

```javascript
Date.prototype.getUTCMonth.call(undefined) // 抛出TypeError

(new Date(undefined)).getUTCMonth() // NaN
(new Date('2010-01-01T00:00:00')).getUTCMonth() // 11
```

#### Date.prototype.getDate( )

获取Date对象的本地日，[1, 31]的整数

```javascript
/**
 * this不为Date对象，抛出TypeError
 * this为Date对象，获取时间值
 *    时间值为NaN，返回NaN
 *    时间值不为NaN，转换为本地时间值，返回对应的日
 */
if (Object.prototype.toString.call(this) !== '[object Date]') {
  throw TypeError
}
let t = this.[[PrimitiveValue]]
if (SameValue(t, NaN)) {
  return NaN
} else {
  return DateFromTime(LocalTime(t))
}
```

示例

```javascript
Date.prototype.getDate.call(undefined) // 抛出TypeError

(new Date(undefined)).getDate() // NaN
(new Date('2010-01-01T00:00:00')).getDate() // 1
```

#### Date.prototype.getUTCDate( )

获取Date对象的UTC日，[1, 31]的整数

```javascript
/**
 * this不为Date对象，抛出TypeError
 * this为Date对象，获取时间值
 *    时间值为NaN，返回NaN
 *    时间值不为NaN，返回对应的日
 */
if (Object.prototype.toString.call(this) !== '[object Date]') {
  throw TypeError
}
let t = this.[[PrimitiveValue]]
if (SameValue(t, NaN)) {
  return NaN
} else {
  return DateFromTime(t)
}
```

示例

```javascript
Date.prototype.getUTCDate.call(undefined) // 抛出TypeError

(new Date(undefined)).getUTCDate() // NaN
(new Date('2010-01-01T00:00:00')).getUTCDate() // 31
```

#### Date.prototype.getDay( )

获取Date对象的本地星期，[0, 6]的整数

```javascript
/**
 * this不为Date对象，抛出TypeError
 * this为Date对象，获取时间值
 *    时间值为NaN，返回NaN
 *    时间值不为NaN，转换为本地时间值，返回对应的星期
 */
if (Object.prototype.toString.call(this) !== '[object Date]') {
  throw TypeError
}
let t = this.[[PrimitiveValue]]
if (SameValue(t, NaN)) {
  return NaN
} else {
  return WeekDay(LocalTime(t))
}
```

示例

```javascript
Date.prototype.getDay.call(undefined) // 抛出TypeError

(new Date(undefined)).getDay() // NaN
(new Date('2010-01-01T00:00:00')).getDay() // 5
```

#### Date.prototype.getUTCDay( )

获取Date对象的UTC星期，[0, 6]的整数

```javascript
/**
 * this不为Date对象，抛出TypeError
 * this为Date对象，获取时间值
 *    时间值为NaN，返回NaN
 *    时间值不为NaN，返回对应的星期
 */
if (Object.prototype.toString.call(this) !== '[object Date]') {
  throw TypeError
}
let t = this.[[PrimitiveValue]]
if (SameValue(t, NaN)) {
  return NaN
} else {
  return WeekDay(t)
}
```

示例

```javascript
Date.prototype.getUTCDay.call(undefined) // 抛出TypeError

(new Date(undefined)).getUTCDay() // NaN
(new Date('2010-01-01T00:00:00')).getUTCDay() // 4
```

#### Date.prototype.getHours( )

获取Date对象的本地时，[0, 23]的整数

```javascript
/**
 * this不为Date对象，抛出TypeError
 * this为Date对象，获取时间值
 *    时间值为NaN，返回NaN
 *    时间值不为NaN，转换为本地时间值，返回对应的时
 */
if (Object.prototype.toString.call(this) !== '[object Date]') {
  throw TypeError
}
let t = this.[[PrimitiveValue]]
if (SameValue(t, NaN)) {
  return NaN
} else {
  return HourFromTime(LocalTime(t))
}
```

示例

```javascript
Date.prototype.getHours.call(undefined) // 抛出TypeError

(new Date(undefined)).getHours() // NaN
(new Date('2010-01-01T00:00:00')).getHours() // 0
```

#### Date.prototype.getUTCHours( )

获取Date对象的UTC时，[0, 23]的整数

```javascript
/**
 * this不为Date对象，抛出TypeError
 * this为Date对象，获取时间值
 *    时间值为NaN，返回NaN
 *    时间值不为NaN，返回对应的时
 */
if (Object.prototype.toString.call(this) !== '[object Date]') {
  throw TypeError
}
let t = this.[[PrimitiveValue]]
if (SameValue(t, NaN)) {
  return NaN
} else {
  return HourFromTime(t)
}
```

示例

```javascript
Date.prototype.getUTCHours.call(undefined) // 抛出TypeError

(new Date(undefined)).getUTCHours() // NaN
(new Date('2010-01-01T00:00:00')).getUTCHours() // 16
```

#### Date.prototype.getMinutes( )

获取Date对象的本地分，[0, 59]的整数

```javascript
/**
 * this不为Date对象，抛出TypeError
 * this为Date对象，获取时间值
 *    时间值为NaN，返回NaN
 *    时间值不为NaN，转换为本地时间值，返回对应的分
 */
if (Object.prototype.toString.call(this) !== '[object Date]') {
  throw TypeError
}
let t = this.[[PrimitiveValue]]
if (SameValue(t, NaN)) {
  return NaN
} else {
  return MinFromTime(LocalTime(t))
}
```

示例

```javascript
Date.prototype.getMinutes.call(undefined) // 抛出TypeError

(new Date(undefined)).getMinutes() // NaN
(new Date('2010-01-01T00:00:00')).getMinutes() // 0
```

#### Date.prototype.getUTCMinutes( )

获取Date对象的UTC分，[0, 59]的整数

```javascript
/**
 * this不为Date对象，抛出TypeError
 * this为Date对象，获取时间值
 *    时间值为NaN，返回NaN
 *    时间值不为NaN，返回对应的分
 */
if (Object.prototype.toString.call(this) !== '[object Date]') {
  throw TypeError
}
let t = this.[[PrimitiveValue]]
if (SameValue(t, NaN)) {
  return NaN
} else {
  return MinFromTime(t)
}
```

示例

```javascript
Date.prototype.getUTCMinutes.call(undefined) // 抛出TypeError

(new Date(undefined)).getUTCMinutes() // NaN
(new Date('2010-01-01T00:00:00')).getUTCMinutes() // 0
```

#### Date.prototype.getSeconds( )

获取Date对象的本地秒，[0, 59]的整数

```javascript
/**
 * this不为Date对象，抛出TypeError
 * this为Date对象，获取时间值
 *    时间值为NaN，返回NaN
 *    时间值不为NaN，转换为本地时间值，返回对应的秒
 */
if (Object.prototype.toString.call(this) !== '[object Date]') {
  throw TypeError
}
let t = this.[[PrimitiveValue]]
if (SameValue(t, NaN)) {
  return NaN
} else {
  return SecFromTime(LocalTime(t))
}
```

示例

```javascript
Date.prototype.getSeconds.call(undefined) // 抛出TypeError

(new Date(undefined)).getSeconds() // NaN
(new Date('2010-01-01T00:00:00')).getSeconds() // 0
```

#### Date.prototype.getUTCSeconds( )

获取Date对象的UTC秒，[0, 59]的整数

```javascript
/**
 * this不为Date对象，抛出TypeError
 * this为Date对象，获取时间值
 *    时间值为NaN，返回NaN
 *    时间值不为NaN，返回对应的秒
 */
if (Object.prototype.toString.call(this) !== '[object Date]') {
  throw TypeError
}
let t = this.[[PrimitiveValue]]
if (SameValue(t, NaN)) {
  return NaN
} else {
  return SecFromTime(t)
}
```

示例

```javascript
Date.prototype.getUTCSeconds.call(undefined) // 抛出TypeError

(new Date(undefined)).getUTCSeconds() // NaN
(new Date('2010-01-01T00:00:00')).getUTCSeconds() // 0
```

#### Date.prototype.getMilliseconds( )

获取Date对象的本地毫秒，[0, 999]的整数

```javascript
/**
 * this不为Date对象，抛出TypeError
 * this为Date对象，获取时间值
 *    时间值为NaN，返回NaN
 *    时间值不为NaN，转换为本地时间值，返回对应的毫秒
 */
if (Object.prototype.toString.call(this) !== '[object Date]') {
  throw TypeError
}
let t = this.[[PrimitiveValue]]
if (SameValue(t, NaN)) {
  return NaN
} else {
  return msFromTime(LocalTime(t))
}
```

示例

```javascript
Date.prototype.getMilliseconds.call(undefined) // 抛出TypeError

(new Date(undefined)).getMilliseconds() // NaN
(new Date('2010-01-01T00:00:00')).getMilliseconds() // 0
```

#### Date.prototype.getUTCMilliseconds( )

获取Date对象的UTC毫秒，[0, 59]的整数

```javascript
/**
 * this不为Date对象，抛出TypeError
 * this为Date对象，获取时间值
 *    时间值为NaN，返回NaN
 *    时间值不为NaN，返回对应的毫秒
 */
if (Object.prototype.toString.call(this) !== '[object Date]') {
  throw TypeError
}
let t = this.[[PrimitiveValue]]
if (SameValue(t, NaN)) {
  return NaN
} else {
  return msFromTime(t)
}
```

示例

```javascript
Date.prototype.getUTCMilliseconds.call(undefined) // 抛出TypeError

(new Date(undefined)).getUTCMilliseconds() // NaN
(new Date('2010-01-01T00:00:00')).getUTCMilliseconds() // 0
```

#### Date.prototype.getTimezoneOffset( )

获取UTC时间和本地时间的分钟差

```javascript
/**
 * this不为Date对象，抛出TypeError
 * this为Date对象，获取时间值
 *    时间值为NaN，返回NaN
 *    时间值不为NaN，返回UTC时间和本地时间的分钟差
 */
if (Object.prototype.toString.call(this) !== '[object Date]') {
  throw TypeError
}
let t = this.[[PrimitiveValue]]
if (SameValue(t, NaN)) {
  return NaN
} else {
  return (t - LocalTime(t)) / msPerMinute
}
```

示例

```javascript
Date.prototype.getTimezoneOffset.call(undefined) // 抛出TypeError

(new Date(undefined)).getTimezoneOffset() // NaN
(new Date('2010-01-01T00:00:00')).getTimezoneOffset() // -480
```

**设置方法**

#### Date.prototype.setTime(time)

设置Date对象的时间值

```javascript
/**
 * this不为Date对象，抛出TypeError
 * time转换为Number类型，限制为前后一亿天内的毫秒数
 * 设置[[PrimitiveValue]]属性为time，并返回
 */
if (Object.prototype.toString.call(this) !== '[object Date]') {
  throw TypeError
}
let v = TimeClip(ToNumber(time))
this.[[PrimitiveValue]] = v
return v
```

示例

```javascript
Date.prototype.setTime.call(undefined) // 抛出TypeError

(new Date()).setTime(NaN) // NaN
(new Date(undefined)).setTime(0) // 0
(new Date('2010-01-01T00:00:00')).setTime(0) // 0
```

#### Date.prototype.setMilliseconds(ms)

设置Date对象的本地毫秒

```javascript
/**
 * this不为Date对象，抛出TypeError
 * ms转换为Number类型，设置本地毫秒为ms
 * 重新计算时间值赋给[[PrimitiveValue]]属性并返回
 */
if (Object.prototype.toString.call(this) !== '[object Date]') {
  throw TypeError
}
let t = LocalTime(this.[[PrimitiveValue]])
let time = MakeTime(HourFromTime(t), MinFromTime(t), SecFromTime(t), ToNumber(ms))
let v = TimeClip(UTC(MakeDate(Day(t), time)))
this.[[PrimitiveValue]] = v
return v
```

示例

```javascript
Date.prototype.setMilliseconds.call(undefined) // 抛出TypeError

(new Date()).setMilliseconds(NaN) // NaN
(new Date(undefined)).setMilliseconds(0) // NaN
(new Date('2010-01-01T00:00:00')).setMilliseconds(100) // 1262275200100
(new Date('2010-01-01T00:00:00')).setMilliseconds(2000) // 1262275202000
```

#### Date.prototype.setUTCMilliseconds(ms)

设置Date对象的UTC毫秒

```javascript
/**
 * this不为Date对象，抛出TypeError
 * ms转换为Number类型，设置UTC毫秒为ms
 * 重新计算时间值赋给[[PrimitiveValue]]属性并返回
 */
if (Object.prototype.toString.call(this) !== '[object Date]') {
  throw TypeError
}
let t = this.[[PrimitiveValue]]
let time = MakeTime(HourFromTime(t), MinFromTime(t), SecFromTime(t), ToNumber(ms))
let v = TimeClip(MakeDate(Day(t), time))
this.[[PrimitiveValue]] = v
return v
```

示例

```javascript
Date.prototype.setUTCMilliseconds.call(undefined) // 抛出TypeError

(new Date()).setUTCMilliseconds(NaN) // NaN
(new Date(undefined)).setUTCMilliseconds(0) // NaN
(new Date('2010-01-01T00:00:00')).setUTCMilliseconds(100) // 1262275200100
(new Date('2010-01-01T00:00:00')).setUTCMilliseconds(2000) // 1262275202000
```

#### Date.prototype.setSeconds(sec [, ms ])

设置Date对象的本地秒

```javascript
/**
 * this不为Date对象，抛出TypeError
 * sec转换为Number类型，设置本地秒为sec
 * ms提供时转换为Number类型，设置本地毫秒为ms
 * 重新计算时间值赋给[[PrimitiveValue]]属性并返回
 */
if (Object.prototype.toString.call(this) !== '[object Date]') {
  throw TypeError
}
let t = LocalTime(this.[[PrimitiveValue]])
let time = MakeTime(HourFromTime(t), MinFromTime(t), ToNumber(sec), IsPresent(ms) ? ToNumber(ms) : msFromTime(t))
let v = TimeClip(UTC(MakeDate(Day(t), time)))
this.[[PrimitiveValue]] = v
return v
```

示例

```javascript
Date.prototype.setSeconds.length // 2 

Date.prototype.setSeconds.call(undefined) // 抛出TypeError

(new Date()).setSeconds(NaN) // NaN
(new Date(undefined)).setSeconds(0) // NaN
(new Date('2010-01-01T00:00:00')).setSeconds(1) // 1262275201000
(new Date('2010-01-01T00:00:00')).setSeconds(1, 100) // 1262275201100
(new Date('2010-01-01T00:00:00')).setSeconds(100, 2000) // 1262275302000
```

#### Date.prototype.setUTCSeconds(sec [, ms ])

设置Date对象的UTC秒

```javascript
/**
 * this不为Date对象，抛出TypeError
 * sec转换为Number类型，设置UTC秒为sec
 * ms提供时转换为Number类型，设置UTC毫秒为ms
 * 重新计算时间值赋给[[PrimitiveValue]]属性并返回
 */
if (Object.prototype.toString.call(this) !== '[object Date]') {
  throw TypeError
}
let t = this.[[PrimitiveValue]]
let time = MakeTime(HourFromTime(t), MinFromTime(t), ToNumber(sec), IsPresent(ms) ? ToNumber(ms) : msFromTime(t))
let v = TimeClip(MakeDate(Day(t), time))
this.[[PrimitiveValue]] = v
return v
```

示例

```javascript
Date.prototype.setUTCSeconds.length // 2 

Date.prototype.setUTCSeconds.call(undefined) // 抛出TypeError

(new Date()).setUTCSeconds(NaN) // NaN
(new Date(undefined)).setUTCSeconds(0) // NaN
(new Date('2010-01-01T00:00:00')).setUTCSeconds(1) // 1262275201000
(new Date('2010-01-01T00:00:00')).setUTCSeconds(1, 100) // 1262275201100
(new Date('2010-01-01T00:00:00')).setUTCSeconds(100, 2000) // 1262275302000
```

#### Date.prototype.setMinutes(min [, sec [, ms ] ] )

设置Date对象的本地分

```javascript
/**
 * this不为Date对象，抛出TypeError
 * min转换为Number类型，设置本地分为min
 * sec提供时转换为Number类型，设置本地秒为sec
 * ms提供时转换为Number类型，设置本地毫秒为ms
 * 重新计算时间值赋给[[PrimitiveValue]]属性并返回
 */
if (Object.prototype.toString.call(this) !== '[object Date]') {
  throw TypeError
}
let t = LocalTime(this.[[PrimitiveValue]])
let time = MakeTime(HourFromTime(t), ToNumber(min), IsPresent(sec) ? ToNumber(sec) : SecFromTime(t), IsPresent(ms) ? ToNumber(ms) : msFromTime(t))
let v = TimeClip(UTC(MakeDate(Day(t), time)))
this.[[PrimitiveValue]] = v
return v
```

示例

```javascript
Date.prototype.setMinutes.length // 3

Date.prototype.setMinutes.call(undefined) // 抛出TypeError

(new Date()).setMinutes(NaN) // NaN
(new Date(undefined)).setMinutes(0) // NaN
(new Date('2010-01-01T00:00:00')).setMinutes(1) // 1262275260000
(new Date('2010-01-01T00:00:00')).setMinutes(1, 1) // 1262275261000
(new Date('2010-01-01T00:00:00')).setMinutes(1, 1, 100) // 126227561100
(new Date('2010-01-01T00:00:00')).setMinutes(100, 100, 2000) // 1262281302000
```

#### Date.prototype.setUTCMinutes(min [, sec [, ms ] ] )

设置Date对象的UTC分

```javascript
/**
 * this不为Date对象，抛出TypeError
 * min转换为Number类型，设置UTC分为min
 * sec提供时转换为Number类型，设置UTC秒为sec
 * ms提供时转换为Number类型，设置UTC毫秒为ms
 * 重新计算时间值赋给[[PrimitiveValue]]属性并返回
 */
if (Object.prototype.toString.call(this) !== '[object Date]') {
  throw TypeError
}
let t = this.[[PrimitiveValue]]
let time = MakeTime(HourFromTime(t), ToNumber(min), IsPresent(sec) ? ToNumber(sec) : SecFromTime(t), IsPresent(ms) ? ToNumber(ms) : msFromTime(t))
let v = TimeClip(MakeDate(Day(t), time))
this.[[PrimitiveValue]] = v
return v
```

示例

```javascript
Date.prototype.setUTCMinutes.length // 3

Date.prototype.setUTCMinutes.call(undefined) // 抛出TypeError

(new Date()).setUTCMinutes(NaN) // NaN
(new Date(undefined)).setUTCMinutes(0) // NaN
(new Date('2010-01-01T00:00:00')).setUTCMinutes(1) // 1262275260000
(new Date('2010-01-01T00:00:00')).setUTCMinutes(1, 1) // 1262275261000
(new Date('2010-01-01T00:00:00')).setUTCMinutes(1, 1, 100) // 126227561100
(new Date('2010-01-01T00:00:00')).setUTCMinutes(100, 100, 2000) // 1262281302000
```

#### Date.prototype.setHours(hour [, min [, sec [, ms ] ] ])

设置Date对象的本地时

```javascript
/**
 * this不为Date对象，抛出TypeError
 * hour转换为Number类型，设置本地时为hour
 * min提供时转换为Number类型，设置本地秒为min
 * sec提供时转换为Number类型，设置本地秒为sec
 * ms提供时转换为Number类型，设置本地毫秒为ms
 * 重新计算时间值赋给[[PrimitiveValue]]属性并返回
 */
if (Object.prototype.toString.call(this) !== '[object Date]') {
  throw TypeError
}
let t = LocalTime(this.[[PrimitiveValue]])
let time = MakeTime(ToNumber(hour), IsPresent(min) ? ToNumber(min) : MinFromTime(t), IsPresent(sec) ? ToNumber(sec) : SecFromTime(t), IsPresent(ms) ? ToNumber(ms) : msFromTime(t))
let v = TimeClip(UTC(MakeDate(Day(t), time)))
this.[[PrimitiveValue]] = v
return v
```

示例

```javascript
Date.prototype.setHours.length // 4

Date.prototype.setHours.call(undefined) // 抛出TypeError

(new Date()).setHours(NaN) // NaN
(new Date(undefined)).setHours(0) // NaN
(new Date('2010-01-01T00:00:00')).setHours(1) // 1262278800000
(new Date('2010-01-01T00:00:00')).setHours(1, 1) // 1262278860000
(new Date('2010-01-01T00:00:00')).setHours(1, 1, 1) // 1262278861000
(new Date('2010-01-01T00:00:00')).setHours(1, 1, 1, 100) // 1262278861100
(new Date('2010-01-01T00:00:00')).setHours(100, 100, 100, 2000) // 1262641302000
```

#### Date.prototype.setUTCHours(hour [, min [, sec [, ms ] ] ])

设置Date对象的UTC时

```javascript
/**
 * this不为Date对象，抛出TypeError
 * hour转换为Number类型，设置UTC时为hour
 * min提供时转换为Number类型，设置UTC秒为min
 * sec提供时转换为Number类型，设置UTC秒为sec
 * ms提供时转换为Number类型，设置UTC毫秒为ms
 * 重新计算时间值赋给[[PrimitiveValue]]属性并返回
 */
if (Object.prototype.toString.call(this) !== '[object Date]') {
  throw TypeError
}
let t = this.[[PrimitiveValue]]
let time = MakeTime(ToNumber(hour), IsPresent(min) ? ToNumber(min) : MinFromTime(t), IsPresent(sec) ? ToNumber(sec) : SecFromTime(t), IsPresent(ms) ? ToNumber(ms) : msFromTime(t))
let v = TimeClip(MakeDate(Day(t), time))
this.[[PrimitiveValue]] = v
return v
```

示例

```javascript
Date.prototype.setUTCHours.length // 4

Date.prototype.setUTCHours.call(undefined) // 抛出TypeError

(new Date()).setUTCHours(NaN) // NaN
(new Date(undefined)).setUTCHours(0) // NaN
(new Date('2010-01-01T00:00:00')).setUTCHours(1) // 1262221200000
(new Date('2010-01-01T00:00:00')).setUTCHours(1, 1) // 1262221260000
(new Date('2010-01-01T00:00:00')).setUTCHours(1, 1, 1) // 1262221261000
(new Date('2010-01-01T00:00:00')).setUTCHours(1, 1, 1, 100) // 1262221261100
(new Date('2010-01-01T00:00:00')).setUTCHours(100, 100, 100, 2000) // 1262583702000
```

#### Date.prototype.setDate(date)

设置Date对象的本地日

```javascript
/**
 * this不为Date对象，抛出TypeError
 * date转换为Number类型，设置本地日为date
 * 重新计算时间值赋给[[PrimitiveValue]]属性并返回
 */
if (Object.prototype.toString.call(this) !== '[object Date]') {
  throw TypeError
}
let t = LocalTime(this.[[PrimitiveValue]])
let day = MakeDay(YearFromTime(t), MonthFromTime(t), ToNumber(date))
let v = TimeClip(UTC(MakeDate(day, TimeWithinDay(t))))
this.[[PrimitiveValue]] = v
return v
```

示例

```javascript
Date.prototype.setDate.call(undefined) // 抛出TypeError

(new Date()).setDate(NaN) // NaN
(new Date(undefined)).setDate(0)  // NaN
(new Date('2010-01-01T00:00:00')).setDate(10) // 1263052800000
(new Date('2010-01-01T00:00:00')).setDate(100) // 1270828800000
```

#### Date.prototype.setUTCDate(date)

设置Date对象的UTC日

```javascript
/**
 * this不为Date对象，抛出TypeError
 * date转换为Number类型，设置UTC日为date
 * 重新计算时间值赋给[[PrimitiveValue]]属性并返回
 */
if (Object.prototype.toString.call(this) !== '[object Date]') {
  throw TypeError
}
let t = this.[[PrimitiveValue]]
let day = MakeDay(YearFromTime(t), MonthFromTime(t), ToNumber(date))
let v = TimeClip(MakeDate(day, TimeWithinDay(t)))
this.[[PrimitiveValue]] = v
return v
```

示例

```javascript
Date.prototype.setUTCDate.call(undefined) // 抛出TypeError

(new Date()).setUTCDate(NaN) // NaN
(new Date(undefined)).setUTCDate(0)  // NaN
(new Date('2010-01-01T00:00:00')).setUTCDate(10) // 1260460800000
(new Date('2010-01-01T00:00:00')).setUTCDate(100) // 1268236800000
```

#### Date.prototype.setMonth(month [, date ] )

设置Date对象的本地月

```javascript
/**
 * this不为Date对象，抛出TypeError
 * month转换为Number类型，设置本地月为month
 * date提供时转换为Number类型，设置本地日为date
 * 重新计算时间值赋给[[PrimitiveValue]]属性并返回
 */
if (Object.prototype.toString.call(this) !== '[object Date]') {
  throw TypeError
}
let t = LocalTime(this.[[PrimitiveValue]])
let day = MakeDay(YearFromTime(t), ToNumber(month), IsPresent(date) ? ToNumber(date) : DateFromTime(t))
let v = TimeClip(UTC(MakeDate(day, TimeWithinDay(t))))
this.[[PrimitiveValue]] = v
return v
```

示例

```javascript
Date.prototype.setMonth.length // 2 

Date.prototype.setMonth.call(undefined) // 抛出TypeError

(new Date()).setMonth(NaN) // NaN
(new Date(undefined)).setMonth(0) // NaN
(new Date('2010-01-01T00:00:00')).setMonth(10) // 1288540800000
(new Date('2010-01-01T00:00:00')).setMonth(10, 10) // 1289318400000
(new Date('2010-01-01T00:00:00')).setMonth(100, 100) // 1533657600000
```

#### Date.prototype.setUTCMonth(month [, date ] )

设置Date对象的UTC月

```javascript
/**
 * this不为Date对象，抛出TypeError
 * month转换为Number类型，设置UTC月为month
 * date提供时转换为Number类型，设置UTC日为date
 * 重新计算时间值赋给[[PrimitiveValue]]属性并返回
 */
if (Object.prototype.toString.call(this) !== '[object Date]') {
  throw TypeError
}
let t = this.[[PrimitiveValue]]
let day = MakeDay(YearFromTime(t), ToNumber(month), IsPresent(date) ? ToNumber(date) : DateFromTime(t))
let v = TimeClip(MakeDate(day, TimeWithinDay(t)))
this.[[PrimitiveValue]] = v
return v
```

示例

```javascript
Date.prototype.setUTCMonth.length // 2 

Date.prototype.setUTCMonth.call(undefined) // 抛出TypeError

(new Date()).setUTCMonth(NaN) // NaN
(new Date(undefined)).setUTCMonth(0) // NaN
(new Date('2010-01-01T00:00:00')).setUTCMonth(10) // 1259683200000
(new Date('2010-01-01T00:00:00')).setUTCMonth(10, 10) // 1257868800000
(new Date('2010-01-01T00:00:00')).setUTCMonth(100, 100) // 1502208000000
```

#### Date.prototype.setFullYear(year [, month [, date ] ] )

设置Date对象的本地年

```javascript
/**
 * this不为Date对象，抛出TypeError
 * 时间值为NaN时，本地时间值置为+0
 * year转换为Number类型，设置本地年为year
 * month提供时转换为Number类型，设置本地月为month
 * date提供时转换为Number类型，设置本地日为date
 * 重新计算时间值赋给[[PrimitiveValue]]属性并返回
 */
if (Object.prototype.toString.call(this) !== '[object Date]') {
  throw TypeError
}
let t = SameValue(this.[[PrimitiveValue]], NaN) ? +0 : LocalTime(this.[[PrimitiveValue]])
let day = MakeDay(ToNumber(year), IsPresent(month) ? ToNumber(month) : MonthFromTime(t), IsPresent(date) ? ToNumber(date) : DateFromTime(t))
let v = TimeClip(UTC(MakeDate(day, TimeWithinDay(t))))
this.[[PrimitiveValue]] = v
return v
```

示例

```javascript
Date.prototype.setFullYear.length // 3

Date.prototype.setFullYear.call(undefined) // 抛出TypeError

(new Date()).setFullYear(NaN) // NaN
(new Date(undefined)).setFullYear(2010) // 1262275200000
(new Date('2010-01-01T00:00:00')).setFullYear(2012) // 1325347200000
(new Date('2010-01-01T00:00:00')).setFullYear(2012, 10, 10) // 1352476800000
(new Date('2010-01-01T00:00:00')).setFullYear(2010, 100, 100) // 1533657600000
```

#### Date.prototype.setUTCFullYear(year [, month [, date ] ] )

设置Date对象的UTC年

```javascript
/**
 * this不为Date对象，抛出TypeError
 * 时间值为NaN时，UTC时间值置为+0
 * year转换为Number类型，设置UTC年为year
 * month提供时转换为Number类型，设置UTC月为month
 * date提供时转换为Number类型，设置UTC日为date
 * 重新计算时间值赋给[[PrimitiveValue]]属性并返回
 */
if (Object.prototype.toString.call(this) !== '[object Date]') {
  throw TypeError
}
let t = SameValue(this.[[PrimitiveValue]], NaN) ? +0 : this.[[PrimitiveValue]]
let day = MakeDay(ToNumber(year), IsPresent(month) ? ToNumber(month) : MonthFromTime(t), IsPresent(date) ? ToNumber(date) : DateFromTime(t))
let v = TimeClip(MakeDate(day, TimeWithinDay(t)))
this.[[PrimitiveValue]] = v
return v
```

示例

```javascript
Date.prototype.setUTCFullYear.length // 3

Date.prototype.setUTCFullYear.call(undefined) // 抛出TypeError

(new Date()).setUTCFullYear(NaN) // NaN
(new Date(undefined)).setUTCFullYear(2010) // 1262304000000
(new Date('2010-01-01T00:00:00')).setUTCFullYear(2012) // 1356969600000
(new Date('2010-01-01T00:00:00')).setUTCFullYear(2012, 10, 10) // 1352563200000
(new Date('2010-01-01T00:00:00')).setUTCFullYear(2010, 100, 100) // 1533744000000
```

### 扩展方法

#### Date.prototype.toGMTString( )

Date.prototype.toUTCString()的旧版本方法

## 实例对象

[[Class]]为Date的对象

```javascript
// 通过构造器Date创建
new Date()
new Date(0)
new Date('2010-01-01')
new Date(2010, 0)
```

### 实例属性

```javascript
date.[[Class]] = 'Date'
date.[[Prototype]] = Date.prototype
date.[[Extensible]] = true
date.[[PrimitiveValue]] = timeValue
```
