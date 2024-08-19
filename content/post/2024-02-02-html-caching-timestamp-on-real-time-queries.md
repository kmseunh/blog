+++
title = "타임스탬프 우습게 보지마라"
date = "2024-02-02"
description = "실시간 조회할 때 html 캐싱 timestamp"
tags = [
    "2024",
    "timestamp"
]
categories = [
    "work"
]
series = ["Worklog"]
+++

실시간으로 화면을 보여주는 기능을 구현해야 해서 개발을 하고 있었는데 브라우저를 리프레시만 하는 간단한 기능이라 크게 어려움은 없었다. <br>
그러나 살짝 쎄한 느낌이 들었는데, 아니나 다를까 실시간으로 업데이트된 내용이 화면에 보여져야 되는데 제대로 전달되지 않는 문제가 생긴 것이다.

<p align="center"><img src="https://github.com/kmseunh/svelte-projects/assets/105186724/91e763ba-b366-4cc7-896f-e60dc173435f" width="400"></p>

<!--more-->

## 어쩐지 이상하더라

업데이트된 데이터가 제대로 보이지 않았던 이유는 캐시 된 파일을 계속 사용하고 있었기 때문이었다. <br>
현재 분리된 `HTML`, `JS`, `CSS` 파일을 사용하여 웹 페이지를 구성하고 있는데, 브라우저는 이러한 파일을 캐싱 하여 성능을 향상시키고 네트워크 요청을 줄인다고 한다.

<p align="center"><img src="https://github.com/kmseunh/svelte-projects/assets/105186724/5f024231-a459-4102-b6d4-4cb83ac5a0ea" width="400"></p>

&nbsp;

## 해결방안

이 문제를 해결하기 위해 파일 이름에 타임스탬프(ts)를 포함하여 자동적으로 변경되도록 설정하여 캐시 무효화를 수행할 수 있도록 했다.

### HTML에서 스크립트 불러오기

```html
<script src="../static/js/jsfile.js?ts=12345"></script>
```

### 자동으로 타임스탬프 업데이트

```html
<script>
    // 현재 시간을 기반으로 타임스탬프 생성
    var timestamp = new Date().getTime();
    // 스크립트 파일을 불러올 URL에 타임스탬프 추가
    var scriptUrl = "../static/js/aa.js?ts=" + timestamp;
    // 스크립트 동적으로 생성하여 페이지에 추가
    var scriptElement = document.createElement("script");
    scriptElement.src = scriptUrl;
    document.body.appendChild(scriptElement);
</script>
```

- `JavaScript`를 사용하여 자동으로 현재 시간을 기준으로 타임스탬프를 생성하고, 이를 파일 불러오는 URL에 추가한다.

이렇게 자동으로 타임스탬프를 업데이트하여 캐시를 무효화하면, 항상 최신 버전의 파일을 로드하여 웹 페이지의 실시간 업데이트를 즉각적으로 확인할 수 있다. <br>
