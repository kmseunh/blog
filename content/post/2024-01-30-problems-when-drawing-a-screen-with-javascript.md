+++
title = "괜히 JavsScript로 화면 그린 그날"
date = "2024-01-30"
description = "JavsScript로 화면 그렸을 때 문제점"
tags = [
    "2024",
    "work",
    "blog",
]
categories = [
    "blog",
]
series = ["Memoirs"]
+++

평소에는 HTML로 화면을 그리는 편인데, 오늘 처음으로 JavsScript로 도전하고 싶은 생각이 들었다. <br>
참지 못하고 저질러버렸는데... div 태그로 각각 그리드 나누고 패딩 조절하는 것보다 훨씬 코드량도 적고 간단해서 좋았다. <br>
라고 생각한 지 10분도 안 돼서 문제가 생겼다..<br>
(코드가 짧고 단순하면 좋은 건 줄 알았지..)

<p align="center"><img src="https://github.com/kmseunh/react-memo/assets/105186724/b74e60d9-a43a-4451-9652-f6322752e19a" width="350"></p>

<!--more-->

## 깝쭉거리지 마라

HTML로 하나씩 코딩을 하기엔 너무 번거롭다는 생각이 들었다. <br>
그래서 반복문을 사용하면 간단하지 않을까 하는 생각? 자만?으로 다음 코드처럼 반복문을 사용해서 한 번에 화면을 그렸다.

```js
//예시 코드
drawBrowser = (row) => {
    let rows = [row];
    $.each(rows, function(index, item) {
        var div = $("<div>").addClass("item").text(item.name);
        $("#container").append(div);
    });
}
```

그러나 이렇게 되면 요소마다 스타일이나 이벤트 처리를 추가해야 하는 경우 코드가 더욱 복잡해진다. <br>
따라서 코드가 길어지고 가독성이 떨어질 수밖에 없다.
&nbsp;

또한, HTML, CSS, JavaScript가 분리되지 않아 유지 보수가 어려워지는데 코드의 변경이 필요한 경우 해당하는 수정하기도 번거로울 수 있다.
&nbsp;

## 항상 겸손해라

위와 같은 이슈로 다시 겸손하게 HTML을 사용해서 화면을 그렸다. <br>

```html
//예시 코드
<div id="container">
    <div class="item1">name1</div>
    <div class="item2">name2</div>
    <div class="item3">name3</div>
    <div class="item4">name4</div>
</div>
```

이번 경험을 통해 JavaScript에서 UI를 조작하는 것보다는 JavaScript와 HTML, CSS를 분리하여 사용해야 한다는 것을 알았고, 그렇게 되면 UI의 구조를 명확하게 나타내고, 가독성을 향상시킬 수 있다는 걸 몸소 느꼈다. <br>

또한 무언가 시도하고 싶으면 시도했을 때 발생할 예외 상황을 꼼꼼하게 생각하고 도전해야 한다는 생각이 들었다.

<p align="center"><img src="https://github.com/kmseunh/react-memo/assets/105186724/aaa4e053-885f-4fe2-b293-4a43c78ff2a7" width="350"></p>
