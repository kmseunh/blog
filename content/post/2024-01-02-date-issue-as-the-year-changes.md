+++
title = "새해 에러 많이 받으세요(?)"
date = "2024-01-02"
description = "해가 바뀌면서으로 생기는 날짜 이슈"
tags = [
    "2024",
    "issue"
]
categories = [
    "work"
]
series = ["Worklog"]
+++

새해가 되면서 기존에 진행했던 프로젝트의 `Date` 이슈가 우수수 쏟아졌다. <br>
현재 년도와 이전 년도의 데이터를 DB에서 `Date` 기준으로 조회를 해서 가져와야하는데 그러지 못했다. <br>
`devtools`에서 수많은 에러 로그들을 마주했을 때 기분이란 참.. 허허

<p align="center"><img src="https://github.com/kmseunh/til/assets/105186724/1ee48d9f-997f-409c-8663-1f4740b26131" width="450"></p>

<!--more-->

## 문제점

기존 코드에서 지난달과 지지난달을 계산하려면 아래 코드처럼 이번 달에서 1과 2를 뺀 값을 사용했다.

```js
let targetDate = bas_dt.substr(0, 7);
let thisYear = parseInt(targetDate.substr(0, 4));
let lastYear = thisYear - 1;

let thisMonth = parseInt(targetDate.substr(5, 2));
let lastMonth = thisMonth - 1;
let lastMonth2 = thisMonth - 2;
```

- 이렇게 되면 아래 코드는 1월일 때, 이전 달은 0이 되며, 2월일 때 지지난달 또한 0이 되기 때문에 값을 가져올 수 없다.

```js
//현재 월과 DB에서 가져온 값을 조회하여 SUM
let last_year_price = row.filter((item) => {
    let itemYear = parseInt(item.bas_dt.substr(0, 4));
    let itemMonth = parseInt(item.bas_dt.substr(5, 2));
    return itemYear === lastYear && itemMonth === lastMonth;
});
```

## 해결방안

`if`문을 사용하여 현재 연도에 따른 월, 주, 일이 어떻게 계산되고 있는지 검사하는 코드를 추가했다.

```js
if (lastMonth === 0) {
    lastMonth = 12;
} else if (lastMonth === -1) {
    lastMonth = 11;
}

if (lastMonth2 === 0) {
    lastMonth2 = 12;
} else if (lastMonth2 === -1) {
    lastMonth2 = 11;
}
```

- 이슈가 발생하는 달은 1월과 2월이기 때문에 해당 월일 때 잘못 계산된 값이 나올 경우 정확한 값을 지정해 주었다.

<br />
이외에도 `filter` 조건을 계산하지 않고 `2023`이라고 지정해주어서 오류가 발생한 부분도 있었다. <br>
다행이 빠르게 에러가 발생한 부분을 파악하고 대처했지만 코드애 대한 검토를 꼼꼼하게 했다면 어땠을까 하는 아쉬움이 든다. <br>
또한 `if`문을 자주 사용는 습관을 들여서 항상 데이터를 검사하는 것도 중요하다는 생각이 들었다.
