---
theme: default
background: clock.jpg
class: "text-center"
highlighter: shiki
# show line numbers in code blocks
lineNumbers: false
---

# Deno の DateTime ライブラリ Ptera を作った話

<div class="pt-12">
  <h2 class="px-2 py-1 rounded cursor-pointer" hover="bg-white bg-opacity-10">
    Takuro Iwamoto
  </h2>
</div>

<div class="abs-br m-6 flex gap-2">
  <a href="https://github.com/Tak-Iwamoto" target="_blank" alt="GitHub"
    class="text-xl icon-btn opacity-50 !border-none !hover:text-white">
    <carbon-logo-github />
  </a>
</div>

---

# 自己紹介

<div class="flex items-center justify-between">
  <div>
    Takuro Iwamoto
  </div>
  <div>
    <img class="w-48 h-48 rounded-full" src="tak_iwamoto_profile.jpeg">
  </div>
</div>

- Software Engineer at STORES 予約
- 最近の興味は Deno と Rust
- Twitter: @tak_rockbook

<br>
<br>

---

# Ptera

<div class="flex items-center mb-2 justify-between">
  <div>
    <li>DateTime Library For Deno</li>
    <li>Fully written in Deno</li>
    <li>Inspired By Day.js, Luxon and Moment.js</li>
  </div>
  <div class="ml-24">
    <img src="ptera.svg">
  </div>
</div>

```ts
import { datetime } from "https://deno.land/x/ptera/mod.ts";

// parse ISO 8601
datetime("2021-06-30T21:15:30.200");

// timezone
datetime().toZonedTime("Asia/Tokyo");

// locale
datetime().setLocale("fr");
```

---

# Why does Ptera exist?

## Node.js の DateTime ライブラリも Deno で使用できる。

```ts
import dayjs from "https://cdn.skypack.dev/dayjs@1.10.4";
import { DateTime } from "https://cdn.skypack.dev/luxon";
import { format } from "https://deno.land/x/date_fns/index.js";

const dayjsDate = dayjs();
const luxonDate = DateTime.now();
format(new Date(), "'Today is a' eeee");
```

<div class="mb-4"></div>

- 単純に Deno で何かライブラリを書いてみたかった！
- Day.js や Luxon のように独自のクラスを提供している Deno のライブラリはなかった。
  - deno_std/datetime に date-fns 的な Date クラスに対する utilities は存在している。
- dayjs は Deno で使用する際に補完が効かない。

---

# datetime

```ts
import { datetime } from "https://deno.land/x/ptera/mod.ts";

// now in local time
datetime();

datetime("2021-06-30T21:15:30.200");

// JavaScript Date
datetime(new Date());

// Object
datetime({ year: 2021, month: 3, day: 21 });

// Unixtime
datetime(1625238137000);

// Array
datetime([2021, 6, 11, 13, 30, 30]);
```

---

# Parse and Format

```ts
const parsedDate = datetime().parse(
  "5/Aug/2021:14:15:30 +0900",
  "d/MMM/YYYY:HH:mm:ss ZZ"
);
// support locale
datetime().parse("2021 лютий 03", "YYYY MMMM dd", { locale: "uk" });

// format to ISO 8601
const dt = datetime({
  year: 2021,
  month: 7,
  day: 21,
  hour: 23,
  minute: 30,
  second: 59,
});
dt.toISO(); // 2021-07-21T23:30:59.000Z
dt.toISODate(); // 2021-07-21
dt.toISOWeekDateDate(); // 2021-W29-3
dt.toISOTime(); // 23:30:59.000
dt.format("YYYY/MMMM/dd"); // 2021/July/21

// support locale
dt.setLocale("fr").format("YYYY/MMMM/dd"); // // 2021/juillet/21
```

---

# Timezone and Intl

```ts
// Timezone
const dt = datetime().toZonedTime("America/New_York");

dt.offsetHour(); // -4
dt.offsetMin(); // -240

const utc = dt.toUTC();

// Intl
const dtFr = datetime("2021-07-03").setLocale("fr");
dtFr.toDateTimeFormat({ dateStyle: "full" }); // samedi 3 juillet 2021;
```

# Basic

```ts
datetime("2045-01-01").isBefore(); // false
datetime("2045-01-01").isAfter(); // true
datetime("2045-01-01").isAfter(datetime("2030-01-01")); // true

datetime("2020-01-01").isLeapYear(); // true
datetime("2021-13-01").isValid(); // false
```

---

# Math

```ts
import {
  diffInMillisec,
  latestDateTime,
  oldestDateTime,
} from "https://deno.land/x/ptera/mod.ts";
const dt1 = datetime("2021-08-21:13:30:00");
const dt2 = datetime("2021-01-30:21:30:00");

diffInMillisec(dt1, dt2); // 17510400000

const latest = datetime("2021-08-21");
const oldest = datetime("2021-01-30");
const middle = datetime("2021-04-30");
const datetimes = [latest, oldest, middle];

latestDateTime(datetimes); // latest
oldestDateTime(datetimes); // oldest

const dt = datetime("2021-08-21:13:30:00");
// { year: 2021, month: 8, day: 21, hour: 13, minute: 30, second: 0, millisecond: 0, }

dt.add({ year: 1 }).substract({ month: 2 });
// { year: 2022, month: 6, day: 21, hour: 13, minute: 30, second: 0, millisecond: 0, }
```

---

# 今後について

## Temporal

ECMAScript 標準の日付 API(2021 年 8 月現在、Stage3)

```ts
const zonedDateTime = Temporal.ZonedDateTime.from({
  timeZone: "America/Los_Angeles",
  year: 1995,
  month: 12,
  day: 7,
});

const record = Temporal.Duration.from({
  minutes: 26,
  seconds: 17,
  milliseconds: 530,
});
```

Temporal API が実装されれば Ptera を含めた、ラッパークラスを提供しているライブラリは不要になる？

Temporal 版 date-fns のようなライブラリにできると面白そう

---

<div class="flex justify-center items-center mt-10"><h1>Denoで日付を扱う際にぜひお試しください！</h1></div>
