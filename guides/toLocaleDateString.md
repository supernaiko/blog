---
title: 'toLocaleDateString 详解'
pubDate: 2026-08-03
description: '解释 Date 对象的 toLocaleDateString 方法、作用、比喻和常见报错'
author: '学习笔记'
tags: ["javascript", "date", "前端基础"]
---

# toLocaleDateString 是什么

## 一句话作用

`toLocaleDateString()` 是 **JavaScript Date 对象的方法**,作用是**把日期对象转成"符合当地人阅读习惯"的字符串**。

## 生活比喻

想象你拿着一张国际航班机票,上面印的时间是机器格式:`2022-07-01T00:00:00.000Z`。

不同国家的人看这个格式会觉得别扭:

- 中国人想看:`2022年7月1日` 或 `2022/7/1`
- 美国人想看:`7/1/2022`(月在前)
- 德国人想看:`1.7.2022`(日在前)

`toLocaleDateString('zh-CN')` 就像一个**翻译官**:
- 你告诉它"按中国习惯格式化"
- 它就把日期翻译成中国人爱看的格式

## 基本用法

```js
const date = new Date('2022-07-01');

date.toLocaleDateString()           // 默认按系统语言,可能输出 "7/1/2022"
date.toLocaleDateString('zh-CN')     // 按中文习惯,输出 "2022/7/1"
date.toLocaleDateString('en-US')    // 按美国习惯,输出 "7/1/2022"
date.toLocaleDateString('de-DE')     // 按德国习惯,输出 "1.7.2022"
```

## 关键点

1. **它是 Date 对象的方法**——只有 Date 对象能调用
2. **字符串没有这个方法**——`"2022-07-01".toLocaleDateString()` 会报错
3. **普通对象没有这个方法**——`{year:2022}.toLocaleDateString()` 会报错
4. `'zh-CN'` 这种参数叫"语言区域代码",zh=中文,CN=中国

## 语言区域代码小知识

`'zh-CN'` 这种代码是 BCP 47 标准(互联网语言标签规范),格式是 `语言-地区`:

| 代码 | 语言-地区 | 含义 |
|---|---|---|
| `zh-CN` | 中文-中国大陆 | 中国大陆简体中文 |
| `zh-TW` | 中文-台湾 | 台湾繁体中文 |
| `en-US` | 英语-美国 | 美国英语 |
| `en-GB` | 英语-英国 | 英国英语 |
| `ja-JP` | 日语-日本 | 日本日语 |

为什么需要"语言+地区"?因为同一个语言在不同地区习惯不同:

- `en-US`(美国):日期格式 `7/1/2022`(月/日/年)
- `en-GB`(英国):日期格式 `01/07/2022`(日/月/年)

同样是英语,日期格式完全相反!所以光说"英语"不够,还得说"哪个地区的英语"。

## 常见报错:`toLocaleDateString is not a function`

### 报错原因

这个报错的意思是:你调用的那个变量**不是 Date 对象**,所以没有这个方法。

### 三种常见触发场景

#### 场景 1:变量是字符串

```js
const dateStr = "2022-07-01";        // 这是字符串,不是 Date 对象
dateStr.toLocaleDateString();        // ❌ 报错:is not a function
```

修复:用 `new Date()` 包装

```js
const dateStr = "2022-07-01";
new Date(dateStr).toLocaleDateString();  // ✅ 正常工作
```

#### 场景 2:变量是普通对象(原型链丢失)

```js
const dateObj = new Date('2022-07-01');
const copied = JSON.parse(JSON.stringify(dateObj));  // 深拷贝后变成普通对象
copied.toLocaleDateString();        // ❌ 报错:is not a function
```

这种情况在**数据序列化传输**时很常见:
- Date 对象经过 `JSON.stringify` → 变成字符串
- 字符串经过 `JSON.parse` → 变成字符串(不会自动还原成 Date)
- 或者某些框架内部转换时,Date 原型链丢失,变成普通对象

#### 场景 3:在 Astro 的 mdx layout 里

这是本项目遇到的具体问题。

**背景**:mdx 文件 frontmatter 里写 `pubDate: 2022-07-01`,YAML 会解析成 Date 对象。

**问题**:Astro 在把 frontmatter 传给 mdx 的 layout 组件时,会做序列化处理,Date 对象的原型链丢失,变成了普通对象或字符串。

**结果**:
- 直接渲染 `{frontmatter.pubDate}` → 显示 `[object Object]`(因为它是个普通对象,不知道怎么转字符串)
- 调用 `frontmatter.pubDate.toLocaleDateString()` → 报错 `is not a function`(因为不是 Date 对象了)

**修复方法**:用 `new Date()` 重新包装:

```jsx
<p>发布日期:{new Date(frontmatter.pubDate).toLocaleDateString('zh-CN')}</p>
```

`new Date(...)` 会把传进来的东西(不管是字符串还是被破坏的对象)重新变成一个真正的 Date 对象,这样就能调用 `toLocaleDateString` 了。

## 什么是原型链(补充知识)

上面的报错都跟"原型链"有关,这里解释一下。

### 比喻

把对象想象成一个人。每个人都有自己的名字、身份证号。但人还能"继承"父辈的能力——比如你能用筷子吃饭,是因为你爸教你用的,你爸能用是因为你爷爷教他用筷子。这个"能力传递链"就是原型链。

### 在 JavaScript 里

```js
const date = new Date('2022-07-01');
date.toLocaleDateString();   // 能用
```

`date` 这个对象本身并没有 `toLocaleDateString` 这个方法,但它的"父亲"(原型)是 `Date.prototype`,`Date.prototype` 上有 `toLocaleDateString` 方法。JS 会沿着原型链往上找,找到这个方法来用。

### 原型链丢失会怎样?

如果对象被序列化(比如 `JSON.parse(JSON.stringify(date))`),结果会变成一个**普通对象**,它的"父亲"变成了 `Object.prototype`(普通对象的父亲),而不是 `Date.prototype`。

`Object.prototype` 上没有 `toLocaleDateString` 方法,所以调用就报错。

这就像:你本来是厨师的儿子,会做菜。但突然某天你失忆了,不记得自己爸爸是厨师,你也就不会做菜了。`new Date()` 就是帮你"找回记忆",重新认 Date 当爸爸。

## 同类方法(拓展)

Date 对象还有两个兄弟方法,用法类似:

| 方法 | 输出什么 | 例子 |
|---|---|---|
| `toLocaleDateString()` | 只输出日期 | `2022/7/1` |
| `toLocaleTimeString()` | 只输出时间 | `上午12:00:00` |
| `toLocaleString()` | 日期 + 时间 | `2022/7/1 上午12:00:00` |

```js
const date = new Date('2022-07-01T08:30:00');

date.toLocaleDateString('zh-CN');   // "2022/7/1"
date.toLocaleTimeString('zh-CN');   // "上午8:30:00"
date.toLocaleString('zh-CN');        // "2022/7/1 上午8:30:00"
```

## 替代方案

如果你不想用 `toLocaleDateString`,还有别的方式格式化日期:

### 方案 1:手动拼接

```js
const date = new Date('2022-07-01');
const str = `${date.getFullYear()}年${date.getMonth() + 1}月${date.getDate()}日`;
// "2022年7月1日"
```

注意:`getMonth()` 返回 0-11(0=一月),所以要 +1。

### 方案 2:用 toISOString

```js
new Date('2022-07-01').toISOString().split('T')[0];  // "2022-07-01"
```

适合需要"年-月-日"机器格式的场景。

### 方案 3:用第三方库 date-fns / dayjs

如果项目里日期处理很多,推荐用库:

```js
import { format } from 'date-fns';
format(new Date('2022-07-01'), 'yyyy年MM月dd日');  // "2022年07月01日"
```

## 总结

- `toLocaleDateString()` 是 Date 对象的方法,把日期转成本地化字符串
- 只有真正的 Date 对象能用,字符串和普通对象不能
- 如果变量被序列化过,用 `new Date()` 重新包装
- 在 Astro mdx layout 里,frontmatter 的 Date 类型会丢失原型链,需要重新包装
