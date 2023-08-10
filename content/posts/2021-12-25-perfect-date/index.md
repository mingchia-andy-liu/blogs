---
title: "The Perfect Date?"
date: 2021-12-25T17:43:44-08:00
slug: the-perfect-date
tags:
  - javascript
coverCaption: Photo by [Kyrie kim](https://unsplash.com/@kyrie3) on [Unsplash](https://unsplash.com/)
---

## Introduction


Today is Christmas 🎄. I want to write something small and fun. Tis' the season, right? How about a perfect date? 


### The Perfect Date

It is probably not something you are thinking about. I am talking about the `Date` object in JavaScript. Handling date and timezone in any language is always a pain to deal with. In JavaScript, this is no exception and perhaps a little worse than others. Here is a quiz:

```js
// browser or nodejs
const date1 = new Date('1970/12/31');
const date2 = new Date('1970-12-31');
// what is the value of `isSame`?
const isSame = date1.valueOf() === date2.valueOf();
```

&nbsp;

### Answer


"it depends". 🤷‍♀️. I know, the answer is not very satisfying.


The answer depends on the **"timezone"** the code at. Because in JS, if you have the format `YYYY/MM/DD`, it will be parsed to your local timezone. But with `YYYY-MM-DD`, it will be parsed to the `UTC` timezone. 🎉 Surprise? So if the timezone is `UTC`, the answer is `true`, if not, then `false`.


Let's look at why? See the MDN
> Given a non-standard date string of "March 7, 2014", parse() assumes a local time zone, but given a simplification of the ISO 8601 calendar date extended format such as "2014-03-07", it will assume a time zone of UTC (ES5 and ECMAScript 2015). Therefore Date objects produced using those strings may represent different moments in time depending on the version of ECMAScript supported unless the system is set with a local time zone of UTC.


Enjoy the holiday.
