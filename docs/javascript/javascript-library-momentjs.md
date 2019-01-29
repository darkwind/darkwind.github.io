---
layout: default
title: Moment 사용법
---

# Moment 사용법

Moment
===

Format Dates
``` js
moment().format('MMMM Do YYYY, h:mm:ss a'); // June 19th 2018, 12:57:48 pm
moment().format('dddd');                    // Tuesday
moment().format("MMM Do YY");               // Jun 19th 18
moment().format('YYYY [escaped] YYYY');     // 2018 escaped 2018
moment().format();                          // 2018-06-19T12:58:21+09:00
```

Relative Time
``` js
moment("20111031", "YYYYMMDD").fromNow(); // 7 years ago
moment("20120620", "YYYYMMDD").fromNow(); // 6 years ago
moment().startOf('day').fromNow();        // 13 hours ago
moment().endOf('day').fromNow();          // in 11 hours
moment().startOf('hour').fromNow();       // an hour ago
```

Calendar Time
``` js
moment().subtract(10, 'days').calendar(); // 06/09/2018
moment().subtract(6, 'days').calendar();  // Last Wednesday at 12:59 PM
moment().subtract(3, 'days').calendar();  // Last Saturday at 12:59 PM
moment().subtract(1, 'days').calendar();  // Yesterday at 12:59 PM
moment().calendar();                      // Today at 12:59 PM
moment().add(1, 'days').calendar();       // Tomorrow at 12:59 PM
moment().add(3, 'days').calendar();       // Friday at 12:59 PM
moment().add(10, 'days').calendar();      // 06/29/2018
```

Multiple Locale Support
``` js
moment.locale();         // ko
moment().format('LT');   // 오후 1:00
moment().format('LTS');  // 오후 1:00:40
moment().format('L');    // 2018.06.19.
moment().format('l');    // 2018.06.19.
moment().format('LL');   // 2018년 6월 19일
moment().format('ll');   // 2018년 6월 19일
moment().format('LLL');  // 2018년 6월 19일 오후 1:00
moment().format('lll');  // 2018년 6월 19일 오후 1:00
moment().format('LLLL'); // 2018년 6월 19일 화요일 오후 1:00
moment().format('llll'); // 2018년 6월 19일 화요일 오후 1:00
```

Moment Timezone
===

Format Dates in Any Timezone
``` js
var jun = moment("2014-06-01T12:00:00Z");
var dec = moment("2014-12-01T12:00:00Z");

jun.tz('America/Los_Angeles').format('ha z');  // 5am PDT
dec.tz('America/Los_Angeles').format('ha z');  // 4am PST

jun.tz('America/New_York').format('ha z');     // 8am EDT
dec.tz('America/New_York').format('ha z');     // 7am EST

jun.tz('Asia/Tokyo').format('ha z');           // 9pm JST
dec.tz('Asia/Tokyo').format('ha z');           // 9pm JST

jun.tz('Australia/Sydney').format('ha z');     // 10pm EST
dec.tz('Australia/Sydney').format('ha z');     // 11pm EST
```

Convert Dates Between Timezones
``` js
var newYork    = moment.tz("2014-06-01 12:00", "America/New_York");
var losAngeles = newYork.clone().tz("America/Los_Angeles");
var london     = newYork.clone().tz("Europe/London");

newYork.format();    // 2014-06-01T12:00:00-04:00
losAngeles.format(); // 2014-06-01T09:00:00-07:00
london.format();     // 2014-06-01T17:00:00+01:00
```
