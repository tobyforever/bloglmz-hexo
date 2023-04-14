---
title: 主流时间库横向对比
date: 2023-04-11 21:45:51
tags: js
toc: true
published: true
---
# 背景

根据《State of Frontend 2022》问卷调查， 最受欢迎的前五个工具库中，时间处理相关的库占据了两席。时间处理工具为什么如此受前端工程师青睐？JS Date 为什么无法满足开发需求？不同的时间库之间又存在哪些差异？
![](https://cdn.jsdelivr.net/gh/tobyforever/uploadpic/upload/20230412115334.png)

# 时间
## 计量标准
时间包含了时刻和时段两个概念，常规意义上的时段是可以通过两个时刻计算得到的，但地球自转的不稳定，这种方式得到的时段并不是恒定的，因而诞生了两种时间计量系统：
世界时（GMT）：根据地球自转的天文测量得到，并不稳定。
原子时（IAT）：根据原子的单次振动时间得到，可以认为是恒定不变的。
这两种时间尺度速率上的不同每一至二年会带来会差大约1秒的差异，因此每隔一定时间会在原子时的基础上增加或减少1秒，即闰秒，得到接近于世界时的时间，称为协调世界时（UTC）。
## 时区
全球被划分为24个时区。规定英国的格林威治天文台旧址所在经线为基准线，即本初子午线，所在时区为零时区，零时区以东为东 1-12 区，以西为西 1-12 区（东12区和西12区是重合的）。每差一个时区，区时相差一小时，越往东区时越早。为了让世界各地都有一个统一的参照时间，规定零时区的 UTC 时间作为标准时间，简称 UTC 时间。
![](https://cdn.jsdelivr.net/gh/tobyforever/uploadpic/upload/20230412115634.png)

# new Date()
## YYYY-MM-DD 的时区问题
使用 new Date('yyyy-MM-dd') 实例化 Date 对象时，由于没有指定具体时刻，系统会自动设置一个时刻为 '00:00:00' 的 UTC 时间，并在用户访问时，返回系统时区的对应时间。如，位于东八区的开发者访问 Date 显示的时间是 UTC+8。位于西七区的开发者访问 Date 显示的时间是 UTC-7。也就是说，如果直接使用 new Date('yyyy-MM-dd') 设置时间，位于不同时区的用户会获取不同的结果，某些情况下会导致意想不到的bug。
使用 new Date('yyyy-MM-dd HH:mm:ss') 或其他官方推荐的日期格式实例化对象，如new Date(dd, MM yyyy)，可以避免上述时区问题，创建当前时区下 00:00:00 的 Date 对象。
```js
// 时区为中国
new Date('2022-10-24') 
// print: Mon Oct 24 2022 08:00:00 GMT+0800 (中国标准时间)

// 时区为美国
new Date('2022-10-24') 
// print: Sun Oct 23 2022 17:00:00 GMT-0700 (北美太平洋夏令时间)

// 使用 new Date('YYYY-MM-DD HH-MM-SS') 可以创建当前时区的 Date 对象
new Date('2022-10-24 00:00:00')
// print: Mon Oct 24 2022 00:00:00 GMT+0800 (中国标准时间)

// 传入格式为'DD, MM YYYY'的日期
new Date('10, 24 2022')
// print: Mon Oct 24 2022 00:00:00 GMT+0800 (中国标准时间)
```

## Safari 的兼容性问题
Safari 只支持 yyyy/MM/dd 或 MM/dd/yyyy 或 MMMM DD, YYYY 格式的日期，使用 new Date('YYYY-MM-DD') 会报错。
## 无法解析或展示特定格式的日期
特定格式日期的解析需要借助正则表达式来完成。
```js
// 通过正则表达式解析
const datePattern = /^(\d{2})-(\d{2})-(\d{4})$/;
const [, month, day, year] = datePattern.exec('10-24-2022');
new Date(`${month}, ${day} ${year}`);
```
# 主流时间库
## MomentJS
彻底解决解析问题和格式化问题
```js
const date1 = moment('2022-10-24');
date1.format()   // print: 2022-10-24T00:00:00+08:00
date1.toArray()  // print: [2022, 9, 24, 0, 0, 0, 0]，注：月份的起始数为0
date1.toJSON()   // print: 2022-10-23T16:00:00.000Z
```
适配多种甚至自定义的格式写法
```js
const date2 = moment('10/24/2022', 'MM/DD/YYYY');
const date3 = moment('2022-10-24-4-30', 'YYYY-MM-DD-HH-mm');
```
包体积大：MomentJS 包体积过于庞大，接近 300kb，且基于 OOP（Object Oriented Programming）的设计需要先引入 moment 对象，再使用对象中的方法，导致无法通过 tree-shaking 压缩体积，引用后会打包所有方法，容易引发首屏加载的性能问题。
时间对象是可变的（mutable）：对时间对象的计算操作会改变对象本身，通常需要拷贝后操作。
```js
  const startDate = moment(); 
  // print: Sun Oct 23 2022 23:11:34 GMT+0800   const endDate = startDate.add(1, 'year'); 
  // print: Mon Oct 23 2023 23:11:34 GMT+0800
  console.log(startDate === endDate);   
  // print: true
```
目前 moment.js 由于历史包袱的原因升级困难， 加上更好的替代品出现，已经停止维护。对于深度使用 moment.js 但希望更换时间库的项目，可以安装 eslint-plugin-you-dont-need-momentjs 来帮助升级。配置方式如下：
```
// package.json
"extends" : ["plugin:you-dont-need-momentjs/recommended"]
```
## DayJS
是 Moment.js 的轻量化方案，拥有同样强大的 API，但包体积只有 6.5KB。
不可变（Immutable）
体积小。为了减小体积，day.js 将一些复杂功能抽离到插件中，使用时需额外引入
拥有和 MomentJS 相同的 API，迁移成本低。但迁移时需注意：
注1：涉及到更改时间对象的操作，不能简单地替换。
```
import moment from "moment";
const timeEntity = moment();
timeEntity.add(1, "d"); // 天数加1

import dayjs from "dayjs";
const timeEntity = moment();
timeEntity = timeEntity.add(1, "d"); // 天数加1
```
如果项目中大量依赖此类逻辑的话，Day.js 插件提供了适配方案，虽然官方并不推荐。
```
var badMutable = require('dayjs/plugin/badMutable')
dayjs.extend(badMutable) // with BadMutable plugin  const today = dayjs()
today.add(1, 'day') // immutable
```
注2：一些特殊功能需要通过插件额外引入，并进行配置。
```
import dayjs from "dayjs";
import dayOfYear from "dayjs/plugin/dayOfYear";
import objectSupport from "dayjs/plugin/objectSupport";

// 配置插件
dayjs.extend(dayOfYear);
dayjs.extend(objectSupport);

export default dayjs;
utc 时间问题。 utc 插件的.utc()、.utcOffset()、.tz('GMT'/'UTC') 方法会返回 utc 时间对象，但 utc 模式下的格式化显示还存在较多问题，某些情况下会显示本地时间而非UTC时间，因此不建议使用 day.js 的 utc 插件。
dayjs.extend(utc)
dayjs('2022-10-24').utcOffset(0, true).format();
// 2022-10-24T00:00:00Z
dayjs('2022-10-24').utcOffset(0, true).add(0).format(); 
// result: 2022-10-23T16:00:00Z
// expect: 2022-10-24T00:00:00Z

dayjs.extend(utc)
dayjs.extend(timezone)
dayjs.tz(0, 'UTC').format()
// 1970-01-01T00:00:00Z
dayjs.tz(0, 'UTC').add(0,'minutes').format()
// result: 1969-12-31T16:00:00Z
// expect: 1970-01-01T00:00:00Z
```
## Date-fns
Date-fns 的 API 是基于 FP（Functional Programming）的，可以按需导入函数。
不可变（Immutable）
函数导入，但相比对象导入，调用不够灵活，每个工具函数都要从指定路径引入。
```
export addDays from 'date-fns/addDays/index.js'
const newDate = addDays(new Date(), 7);
```
优化：将需要用到的工具函数引入到模块文件中，再对外暴露方法。
```
// custom-date-fns.js
export { default as add } from 'date-fns/add/index.js'
export { default as addBusinessDays } from 'date-fns/addBusinessDays/index.js'
export { default as addDays } from 'date-fns/addDays/index.js'
export { default as addHours } from 'date-fns/addHours/index.js'
```
add 操作，2KB
![](https://cdn.jsdelivr.net/gh/tobyforever/uploadpic/upload/20230412120110.png)
format + add 操作，24.1KB
![](https://cdn.jsdelivr.net/gh/tobyforever/uploadpic/upload/20230412120122.png)
parse 解析日期字符串，98.5KB
![](https://cdn.jsdelivr.net/gh/tobyforever/uploadpic/upload/20230412120135.png)

## 横向对比
![](https://cdn.jsdelivr.net/gh/tobyforever/uploadpic/upload/20230412120155.png)

# 总结
Native Date 无法直接解析自定义格式的时间字符串，且容易引入时区问题。不推荐。
Moment.js 包体积过大，且时间对象存在 mutable 问题，源代码也早已停止维护。不推荐。
Day.js 克服了 moment.js 的缺陷，且 api 与 moment.js 高度吻合，从 moment.js 迁移成本低。但是部分功能需要通过插件引入，其中 utc 插件会引发 utc 时间的显示问题。建议在不涉及 UTC 时间的情况下使用。
Date-fns 同样克服了 moment.js 的缺陷，并支持 tree-shaking，单独使用某些功能时，引入的包体积甚至小于 day.js。但需要从目标目录导入所需的工具函数，上手难度大。在引入了多种工具函数或涉及解析时间字符串时，还会导致包体积过大。推荐轻度需求或 day.js 无法胜任时使用。