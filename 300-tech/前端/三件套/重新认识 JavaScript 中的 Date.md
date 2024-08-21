---
title: 重新认识 JavaScript 中的 Date
date: 2023-05-29T09:27:00+08:00
updated: 2024-08-21T10:32:36+08:00
dg-publish: false
aliases:
  - 重新认识 JavaScript 中的 Date
author: Luwang
category: web
comments: true
cover: //cdn.wallleap.cn/img/post/1.jpg
description: 文章描述
image-auto-upload: true
keywords:
  - 关键词1
  - 关键词2
  - 关键词3
rating: 1
source: //keqingrong.cn/blog/2021-01-29-about-js-date/
tags:
  - 前端
  - JavaScript
url: //myblog.wallleap.cn/post/1
---

年底公司统计下班时间，个人又翻出了 19 年写的 [clock-helper-cli](https://www.npmjs.com/package/clock-helper-cli) 用来统计电脑关机时间。在使用时发现 Windows 上的 [wevtutil](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/wevtutil) 返回的日期格式有变化，从 `2019-05-25T21:15:53.500` 格式变成 `2021-01-24T22:10:15.2980000Z`。遂开始了解这两者有何区别。

## 科普时间

### 常见名词

- 格林尼治标准时间（Greenwich Mean Time, GMT），也译作“格林威治标准时间”，GMT 根据地球的自转和公转计算时间，已被 UTC 取代。
- 协调世界时（Coordinated Universal Time, UTC），也被称为“世界标准时间”，UTC 根据原子钟计算时间。
- 本地时间（Local Time），UTC + Timezone Offset = Local Time
- 北京时间（Beijing Time），即中国标准时间 (China Standard Time, CST），UTC + 8 = CST
- 夏时制（Daylight Saving Time, DST），又称“日光节约时制”、“夏令时间”、“夏令时”，是一种为节约能源而人为规定地方时间的制度，在这一制度实行期间所采用的统一时间称为“夏令时间”。夏天时钟拨快一小时，冬天再拨回标准时间（冬令时间）。
- Unix 时间戳（Unix timestamp），也被称为 Unix Epoch、Unix time、POSIX time，是一种时间表示方式，定义为从格林威治时间 1970 年 01 月 01 日 00 时 00 分 00 秒起至现在的总秒数。Unix 时间戳的 `0` 相当于 ISO 8601 的 `1970-01-01T00:00:00Z`。如果使用 32 位整型变量存储 Unix 时间戳，会引发 2038 年问题（Y2038 bug）。

### 中国时区

1949 年以前，中国一共分了 5 个时区，以哈尔滨、上海、重庆、乌鲁木齐和喀什为代表——分别是：长白时区 GMT+8:30、中原标准时区 GMT+8、陇蜀时区 GMT+7、新藏时区 GMT+6 和昆仑时区 GMT+5:30。

- `Asia/Harbin` 长白时区 GMT+8:30
- `Asia/Shanghai` 中原标准时区 GMT+8
- `Asia/Chongqing` 陇蜀时区 GMT+7
- `Asia/Urumqi` 新藏时区 GMT+6
- `Asia/Kashgar` 昆仑时区 GMT+5:30

1949 年以后，中国大陆统一使用 UTC+8 时区时间，即北京时间，以上的 5 个时区都调整为 UTC+8。由于存在历史问题，其中以和旧时区兼容的 `Asia/Shanghai` 最为常用。另外还可以使用以下城市作为 UTC+8 时区标识：

- `Asia/Hong_Kong` UTC+8
- `Asia/Macao` UTC+8
- `Asia/Taipei` UTC+8

注意，没有 `Asia/Beijing` 时区。

### 美国时区

美国目前一共存在 6 个时区：

- 东部标准时间（Eastern Standard Time, EST），代表城市华盛顿特区（Washington DC）、纽约（New York）
	- 东部夏令时间（Eastern Daylight Time, EDT）
- 中央标准时间（Central Standard Time, CST）美国中部各州所使用的标准时间，比格林威治标准时间晚 6 小时，代表城市芝加哥（Chicago）、新奥尔良（New Orleans）
- 山地标准时间（Mountain Standard Time, MST），代表城市盐湖城（Salt Lake City）、丹佛（Denver）
	- 山地夏令时间（Mountain Daylight Time, MDT）
- 太平洋标准时间（Pacific Standard Time, PST），比格林威治时间晚 8 小时，代表城市旧金山（San Francisco）、洛杉矶（Los Angeles）、西雅图（Seattle）
- 阿拉斯加标准时间（Alaska Standard Time, AKST），仅限阿拉斯加，代表城市安克雷奇（Anchorage）
- 夏威夷标准时间（Hawaii Standard Time, HST），仅限夏威夷，代表城市火奴鲁鲁（Honolulu）

### 日期和时间的相关标准

- ISO
	- [ISO 8601 Date and time -- Representations for information interchange](https://www.iso.org/iso-8601-date-and-time-format.html): 目前最新标准为 ISO 8601-1:2019 和 ISO 8601-2:2019
- IETF
	- Date and Time on the Internet: Timestamps [[RFC 3339]](<https://datatracker.ietf.org/doc/html/rfc3339>): 对标 ISO 8601，日期示例：`1985-04-12T23:20:50.52Z`、`1996-12-19T16:39:57-08:00`
	- Internet Message Format [[RFC 822]](<https://datatracker.ietf.org/doc/html/rfc822#section-5>), [[RFC 2822]](<https://datatracker.ietf.org/doc/html/rfc2822#section-3.3>), [[RFC 5322]](<https://datatracker.ietf.org/doc/html/rfc5322#section-3.3>): 虽然是 Email 标准，但包含了 Date and Time Specification，日期示例：`Fri, 21 Nov 1997 09:55:06 -0600`、`21 Nov 97 09:55:06 GMT`（已废弃，2 位年份表示）
	- Hypertext Transfer Protocol [[RFC 7231]](<https://datatracker.ietf.org/doc/html/rfc7231#section-7.1.1>): HTTP 标准中也涉及到对 Date/Time Formats 的说明，日期示例：`Sun, 06 Nov 1994 08:49:37 GMT`
- 中华人民共和国国家标准: 对标 ISO 8601
	- [GB/T 7408-2005《数据元和交换格式 信息交换 日期和时间表示法》](http://openstd.samr.gov.cn/bzgk/gb/newGbInfo?hcno=3CFD9AE8FEADB062B3BC53651930DED1)
	- [GB/T 15835-2011《出版物上数字用法的规定》](http://openstd.samr.gov.cn/bzgk/gb/newGbInfo?hcno=F5DAC3377DA99C8D78AE66735B6359C7)

为了减少日期解析的开销，像 [Moment.js](https://momentjs.com/docs/#/parsing/string/) 这样的库，默认只支持 ISO 8601 和 RFC 2822 格式，其他格式需要自行解析。而 HTTP 标准中所谓的 **HTTP-date**，首选 RFC 5322 格式。

## Date 对象自带的那些序列化方法

```js
const date = new Date(1500287944127);

date.toISOString(); // "2017-07-17T10:39:04.127Z"
date.toJSON();      // "2017-07-17T10:39:04.127Z"

date.toUTCString(); // "Mon, 17 Jul 2017 10:39:04 GMT"
date.toGMTString(); // "Mon, 17 Jul 2017 10:39:04 GMT"

date.toDateString(); // "Mon Jul 17 2017"
date.toTimeString(); // "18:39:04 GMT+0800 (中国标准时间)" 或 "18:39:04 GMT+0800 (CST)"
date.toString();     // "Mon Jul 17 2017 18:39:04 GMT+0800 (中国标准时间)"

date.toLocaleDateString(); // "2017/7/17"
date.toLocaleTimeString(); // "下午6:39:04"
date.toLocaleString();     // "2017/7/17 下午6:39:04"

date.getTime(); // 1500287944127
date.valueOf(); // 1500287944127
```

- `Date.prototype.toISOString()` 返回 ISO 8601 `YYYY-MM-DDTHH:mm:ss.sssZ` 格式时间。
- `Date.prototype.toJSON()` 基于 `toISOString()`，返回值和其一致。
- `Date.prototype.toGMTString()` 已废弃，应使用 `toUTCString()` 代替，返回值和其一致。
- `Date.prototype.toUTCString()` 返回 `Www, dd Mmm yyyy hh:mm:ss GMT` 格式时间。
- `Date.prototype.toDateString()` 返回年月日周。
- `Date.prototype.toTimeString()` 返回时分秒时区。
- `Date.prototype.toString()` 返回年月日周时分秒时区。
- `Date.prototype.toLocaleDateString()` 返回指定时区的年月日周。
- `Date.prototype.toLocaleTimeString()` 返回指定时区的时分秒时区。
- `Date.prototype.toLocaleString()` 返回指定时区的年月日周时分秒时区。
- `Date.prototype.getTime()` 返回 13 位 Unix 时间戳，毫秒单位（标准 Unix 时间戳 10 位，以秒为单位）。
- `Date.prototype.valueOf()` 等价于 `getTime()`，多为 JavaScript 内部调用。

### 本地化的 `Date.prototype.toLocaleString()`

```js
new Date().toLocaleString('en-US', {timeZone: 'America/New_York'})
new Date().toLocaleString('zh-CN', {timeZone: 'Asia/Shanghai'})
new Date().toLocaleString('zh-CN', {timeZone: 'Asia/Shanghai', hour12: false})
// IE11 传如 'Asia/Shanghai' 会报错：选项值“ASIA/SHANGHAI”(对于“timeZone”)超出了有效范围。应为: ['UTC']
new Date().toLocaleString('zh-CN', {timeZone: 'UTC', hour12: false})
new Date().toLocaleString('zh-CN', {
  timeZone: 'Asia/Shanghai',
  hour12: false,
  year: 'numeric',
  month: '2-digit',
  day: '2-digit',
  hour: '2-digit',
  minute: '2-digit',
  second: '2-digit'
})
```

## 常见日期格式解析

- `2019-05-25T13:15:53.500Z` 即 `toISOString()` 返回的日期格式 `YYYY-MM-DDTHH:mm:ss.sssZ`。
- `2019-05-25T21:15:53.500+08:00` 另一种 UTC 时间表示方式。
- `2019-05-25T21:15:53.500` Windows 10 1904 上 wevtutil 返回的日期格式。
- `2019-05-25T13:15:53.500000000Z` Windows 10 1904 事件查看器日期格式，即 `YYYY-MM-DDTHH:MM:SS.nnnnnnnnnZ`。
- `2021-01-24T22:10:15.2980000Z` Windows 10 20H2 上 wevtutil 返回的日期格式。

### `2019-05-25T13:15:53.500000000Z` 和 `2021-01-24T22:10:15.2980000Z` 格式

其中字母 `T` 用于分隔日期和时间，`Z` 表示使用 UTC 时区。`2019-05-25T13:15:53.500Z` 相当于 `2019-05-25T13:15:53.500+00:00`。对于时间格式中“秒”的部分，我们知道：

```js
1s = 1,000ms 毫秒（10的-3次方）
1s = 1,000,000μs 微秒（10的-6次方）
1s = 1,000,000,000ns 纳秒（10的-9次方）
```

在 JavaScript 中 `Date` 只精确到毫秒，所以即使原数据精确到 0.1 微秒或者 1 纳秒，也会被舍弃。

```js
new Date('2019-05-25T13:15:53.500+00:00').toISOString(); // "2019-05-25T13:15:53.500Z"
new Date('2019-05-25T13:15:53.500Z').toISOString(); // "2019-05-25T13:15:53.500Z"
new Date('2021-01-24T22:10:15.2980000Z').toISOString(); // "2021-01-24T22:10:15.298Z"
new Date('2019-05-25T13:15:53.500000000Z').toISOString(); // "2019-05-25T13:15:53.500Z"
```

### `2019-05-25T21:15:53.500` 格式

在解析不包含时区信息的日期时，默认为本地时区，对我们来说就是北京时间，即 UTC+8 中国标准时间。

```js
new Date('2019-05-25T21:15:53.500').toISOString(); // "2019-05-25T13:15:53.500Z"
```

效果等同于：

```js
new Date('2019-05-25T21:15:53.500+08:00').toISOString(); // "2019-05-25T13:15:53.500Z"
```

系统切换不同时区后的解析结果：

```js
// 系统时区：UTC+08:00 中国标准时间
new Date('2021-01-27T09:48:00').toISOString(); // "2021-01-27T01:48:00.000Z"

// 系统时区：UTC+00:00 格林尼治标准时间
new Date('2021-01-27T09:48:00').toISOString(); // "2021-01-27T09:48:00.000Z"

// 系统时区：UTC-08:00 北美太平洋标准时间
new Date('2021-01-27T09:48:00').toISOString(); // "2021-01-27T17:48:00.000Z"
```

### 其他格式解析

不同浏览器的 `Date` 有着不同的实现，尤其是对日期的解析差异。比较明显的是 IE9/10/11、Sarari 不支持生活中非常常见的 `2021-01-27 09:48:00` 格式。

```js
new Date('Wed, 27 Jan 2021 01:48:00 GMT') // OK

new Date('2021-01-27T01:48:00.000Z') // OK
new Date('2021-01-27T09:48:00') // OK
new Date('2021-01-27 09:48:00') // 在 Safari 浏览器中返回 `Invalid Date`，Chrome、Firefox 正常
new Date('2021/01/27 09:48:00') // OK

new Date('2021-01-01') // OK
new Date('2021-1-1') // 在 Safari 浏览器中返回 `Invalid Date`，Chrome、Firefox 正常
```

当然，如果遇到无法解析的日期时间，Chrome、Firefox 一样会返回 `Invalid Date`，比如：

```js
new Date(undefined) // Invalid Date
new Date('') // Invalid Date
```

可以通过判断 `Date` 值的时间戳是否为 `NaN` 识别 `Invalid Date`。

```js
isDate(new Date('2021-01-27 09:48:00')); // true
isValidDate(new Date('2021-01-27 09:48:00')); // 在 Safari 中返回 false，在 Chrome 中返回 true
isDate(new Date('')); // true
isValidDate(new Date('')); // false

/**
 * 判断是否是 Date 对象
 * @param {*} value
 */
function isDate(value) {
  // `Invalid Date` 同样是 Date 类型对象
  return Object.prototype.toString.call(value) === '[object Date]';
}

/**
 * 判断是否是合法的 Date 对象
 * @param {*} value
 */
function isValidDate(value) {
  // 对于 `Invalid Date`，`getTime()` 返回 `NaN`
  return Object.prototype.toString.call(value) === '[object Date]' && !isNaN(value);
}
```

## `YYYY` 和 `yyyy` 的问题

年底经常会出现 [你今天因为 YYYY-MM-dd 被提 BUG 了吗](https://www.v2ex.com/t/633650?p=1) 中的情况：

```js
//*******************************************************************
// 参考 https://www.v2ex.com/t/633650?p=1#r_8409381
//*******************************************************************
import java.util.Date;
import java.util.Calendar;
import java.text.DateFormat;
import java.text.SimpleDateFormat;

public class DateFormatExample
{
  public static void main(String[] args)
  {
      Calendar calendar = Calendar.getInstance();
      
      calendar.set(2019, Calendar.DECEMBER, 31); // 2019-12-31
      Date date1 = calendar.getTime();
      
      calendar.set(2020, Calendar.JANUARY, 1); // 2020-01-01
      Date date2 = calendar.getTime();

      DateFormat formatUpperCase = new SimpleDateFormat("YYYY/MM/dd");
      System.out.println("2019-12-31 to YYYY/MM/dd: " + formatUpperCase.format(date1));
      System.out.println("2020-01-01 to YYYY/MM/dd: " + formatUpperCase.format(date2));

      DateFormat formatLowerCase = new SimpleDateFormat("yyyy/MM/dd");
      System.out.println("2019-12-31 to yyyy/MM/dd: " + formatLowerCase.format(date1));
      System.out.println("2020-01-01 to yyyy/MM/dd: " + formatLowerCase.format(date2));
  }
}
```

打印

```
2019-12-31 to YYYY/MM/dd: 2020/12/31
2020-01-01 to YYYY/MM/dd: 2020/01/01
2019-12-31 to yyyy/MM/dd: 2019/12/31
2020-01-01 to yyyy/MM/dd: 2020/01/01
```

Web 开发中遇到这个问题多是出现在 Java 服务端，在 Java 的 `DateTimeFormatter` 定义中，`YYYY` 对应 `week-based-year`，年份的表示基于日期所在周，`yyyy` 对应 `year-of-era`，就是我们日常使用的年份。所以实际使用时应该使用 `yyyy-MM-dd` 代替 `YYYY-MM-dd`，使用 `yyyy-MM-dd HH:mm:ss` 代替 `YYYY-MM-dd HH:mm:ss`。

另外我们需要认识到，由于缺少共识，不同语言、不同库对日期格式化的表示方法有不同的定义、实现，实际使用时需要核对相关文档，避免盲目使用。以前端为例，JavaScript 的 `Date` 对象本身并不支持类似的格式化，完全依赖第三方库实现，像 Moment.js 使用 `YYYY`，date-fns 使用 `yyyy`，我们不必担心出现基于周纪年的问题，因为它们实际上都对应 Java 中的 `yyyy`。

- [Moment.js Format](https://momentjs.com/docs/#/displaying/format/)
- [Day.js Format](https://day.js.org/docs/zh-CN/display/format)
- [date-fns Unicode Tokens](https://github.com/date-fns/date-fns/blob/master/docs/unicodeTokens.md)
- [Java DateTimeFormatter](https://docs.oracle.com/javase/8/docs/api/java/time/format/DateTimeFormatter.html#patterns)

## 思考

在数据存储、交换过程中使用日期和时间时，为了尽量不丢失原始信息，应该使用类似 ISO 8601 `2019-05-25T13:15:53.500Z` 或 RFC 3339 `2019-05-25T21:15:53.500+08:00` 或 `Sat, 25 May 2019 13:15:53 GMT`（对非英语国家来说可读性略差）格式的日期和时间，由使用方（比如前端）来进行格式化处理。既可以避免 Unix 时间戳 Y2038 问题，也可以避免国际化、本地化处理时时区展示出错，让数据的归后端，展示的归前端。目前 TC39 已经有了新的、更好用的日期 API 提案 [Temporal](https://github.com/tc39/proposal-temporal)，前端未来处理日期和时间会更加方便。

## 相关链接

- [Date and Time Formats - W3C](https://www.w3.org/TR/NOTE-datetime)
- [Date - MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date)
- [Intl.DateTimeFormat - MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Intl/DateTimeFormat)
- [ISO 8601 - Wikipedia](https://en.wikipedia.org/wiki/ISO_8601)
- [Temporal proposal - TC39](https://tc39.es/proposal-temporal/docs/index.html)
