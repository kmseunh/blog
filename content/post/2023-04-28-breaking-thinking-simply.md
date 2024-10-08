+++
title = "유연한 사고, 단순하지 않기"
date = "2023-04-28"
description = "코드를 짤 때 가독성을 고려하지 않아서 생긴 이슈"
tags = [
    "2023",
    "issue"
]
categories = [
    "work"
]
series = ["Worklog"]
+++

어느 날 개발자는 항상 모든 일에 생각을 해야 한다는 말을 들었다.

코드 한 줄을 짤 때에도 단순하게 ‘이렇게 하면 되겠지?’ 하고 바로 완성하는 게 아니라 어떻게 해야 더 간단하고 가독성이 높은 코드로 완성할 수 있을까에 대한 생각을 해야 한다.

다른 주니어 개발자들은 어떤지 모르겠지만 나는 단순하게 코드를 완성하고 잘 작동이 되면 더 이상 신경을 쓰지 않았다. <br> 처음에는 에러만 안 나면 상관없겠지라는 생각이었지만, 점점 코드가 많아지면 많아질수록 가독성이나 효율적인 부분에서 떨어지게 된다는 것을 알았다.

<p align="center"><img src="https://github.com/kmseunh/blog/assets/105186724/4814cdda-194e-466a-8592-b102d05c7a4d" width="500"></p>

<!--more-->

## 길다고 다 좋은 건 아니다

처음 코드를 공부할 때 알고리즘에 흥미를 느껴서 매일 2–3문제씩 도전해 본 적이 있다.

조금 어려운 문제를 해결할 때에는 코드가 좀 길게 완성될 때가 있었는데, 당시에는 테스트를 통과하는 목적만 달성되면 된다라고 생각하고 단순하게 테스트 통과만 신경을 썼었다.

다른 사람들의 문제풀이를 볼 수 있어서 살짝 구경해 볼까 하고 들어가 봤는데 신선한 충격을 받았다. <br> 1시간 동안 머리 쥐어짜서 10줄 넘게 작성되어 있는 내 코드를 보다가 4줄도 안되는 코드를 봤을 때의 기분이란..

프로젝트에 실제 사용되는 코드랑 알고리즘이랑은 연관되어 있는 듯 없는 듯 아직 내가 판단하기에는 애매하기 때문에 좋은 예시라고 할 수는 없다. <br> (그래도 공감..해줘)

항상 일정 부분 개발을 완료하거나 이슈사항이 생기면 대표님과 함께 동료검토를 한다. <br> 그러면서 코드 리뷰도 하는 경우도 있는데 한번은 `if ~ else` 문이 범벅이거나 계속 코드를 나열해가면서 데이터를 필터링하는 내 코드를 `for` 문으로 간략하게 리팩토링을 해주셨다.

### 수정 전 예시

```js
function processData(data) {
    let result = '';
    if (data === 'A') {
        result = 'Apple';
    } else if (data === 'B') {
        result = 'Banana';
    } else if (data === 'C') {
        result = 'Cherry';
    } else if (data === 'D') {
        result = 'Date';
    } else if (data === 'E') {
        result = 'Elderberry';
    } else {
        result = 'Unknown';
    }
    return result;
}
```

### 수정 후 예시

```js
// for 문을 사용한 코드
function processData(data) {
    const dataMap = {
        'A': 'Apple',
        'B': 'Banana',
        'C': 'Cherry',
        'D': 'Date',
        'E': 'Elderberry'
    };
    let result = 'Unknown';
    for (const key in dataMap) {
        if (data === key) {
            result = dataMap[key];
            break;
        }
    }
    return result;
}
```

_확실히 가독성도 높아지고 간단해보인다._

<hr>

코드는 사람에 따라서 다르게 짜일 수 있기 때문에 재밌는 것 같다. <br> 어디선가 ‘_컴퓨터가 이해할 수 있는 코드는 누구나 짤 수 있고, 좋은 프로그래머는 사람이 이해할 수 있는 코드를 짠다._’는 문구를 본 적이 있다.

항상 다양하게 생각하는 습관을 가져서 리팩토링뿐만 아니라 개발하는 데에 있어 좋은 프로그래머가 되고 싶다.
