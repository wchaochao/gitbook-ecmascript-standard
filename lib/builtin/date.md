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

#### 标准时间格式

`YYYY-MM-DDTHH:MM:ss.sssZ`

* `YYYY`: 0000 - 9999, 四位数的年
* `MM`: 01 - 12, 两位数的月，默认为01
* `DD`: 01 - 31, 两位数的天，默认为01
* `T`: 分隔日期和时间
* `HH`: 00 - 23, 两位数的时，默认为00
* `MM`: 00 - 59, 两位数的分，默认为00
* `ss`: 00 - 59, 两位数的秒，默认为00
* `sss`: 00 - 999, 三位数的毫秒，默认为00
* `Z`: 时区，默认为单`Z`
 * 单`Z`: UTC时间
 * `Z+HH:mm`: 东时区
 * `Z-HH:mm`: 西时区

日期格式

* `YYYY`
* `YYYY-MM`
* `YYYY-MM-DD`

时间格式

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
  } while(true)
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

时间值限制在前后一亿天内

```javascript
/**
 * 时间值不为有限值或超过前后一亿天范围时，返回NaN
 * 其他，转换为整数
 */
if (!isInfinite(time) || abs(time) >= 8.64e+15) {
  return NaN
} else {
  return ToInteger(time)
}
```

## 构造函数

### Date([year [, month [, date [, hours [, minutes [, seconds [, ms ] ] ] ] ] ] ] )

作为函数使用，返回当前时间的字符串表示

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
 * [[Primitive]]属性为当前时间对应的UTC毫秒数
 */
let date = new Object()
date.[[Class]] = 'Date'
date.[[Prototype]] = Date.prototype
date.[[Extensible]] = true
date.[[Primitive]] = currentTimeValue
```

### new Date(value)

1个参数，作为构造器使用，创建Date对象，指定UTC毫秒数或时间字符串

```javascript
/**
 * 创建Date对象
 * [[Class]]属性为'Date'
 * [[Prototype]]属性为Date.prototype
 * [[Extensible]]属性为true
 * [[Primitive]]属性为指定的UTC毫秒数或时间字符串对应的UTC毫秒数，限制在前后一亿天内
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
 * [[Primitive]]属性为本地时间元素对应的UTC毫秒数，限制在前后一亿天内
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

date.[[Primitive]] = TimeClip(UTC(t))
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

### Date.now()

获取当前时间对于的UTC毫秒数

```javascript
/**
 * 返回当前时间对于的UTC毫秒数
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

### Date.UTC(year, month [, date [, hours [, minutes [, seconds [, ms ] ] ] ] ] )

将UTC时间元素转换为UTC毫秒数

```javascript
/**
 * 将UTC时间元素转换为对应的UTC毫秒数，限制在前后一亿天内
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
