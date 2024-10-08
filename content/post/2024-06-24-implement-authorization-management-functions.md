+++
title = "좌충우돌 권한관리 기능 구현하기"
date = "2024-06-24"
description = "좌충우돌 권한관리 기능 구현하기"
tags = [
    "2024",
    "impl"
]
categories = [
    "work"
]
series = ["Worklog"]
+++

매일 데이터 집계 로직을 작성하고 검증하는 작업을 주로 하던 중, 프로젝트에서 권한 관리 기능이 필요하게 되었다. <br>
권한 관리는 이전부터 구현해 보고 싶었던 기능 중 하나였고, 머릿속으로는 이미 구상이 다 되어 있어 자신만만하게 시작했지만, 역시나 쉽지 않았다. <br>
( 특히, 관리 측면에서 신경 써야 할 부분이 많았다. )

<p align="center"><img src="https://github.com/kmseunh/blog/assets/105186724/f6802f08-1949-4c7c-80c5-5fac10830551" width="450"></p>

<!--more-->

## 쉽게 만드실 수 수 수 수퍼노바

처음엔 각 팀별로 접근 가능한 아이디와 전체 접근 가능한 아이디를 만들어, 각 아이디의 권한에 따라 페이지 접근 시 보여지는 기능을 구분하면 될 것이라고 생각했다.

기존 계정 관리 테이블에 각 공장과 팀별 고유 시퀀스를 할당할 수 있도록 `user_acc` 테이블을 만들어 UserId를 매핑했다. <br>
이후, 웹 페이지에서 DataTable을 사용해 각 계정별 접근 권한이 주어진 팀을 체크할 수 있는 권한 관리 페이지를 만들었다. <br>
(원래는 HandersonTable을 사용하려고 했지만...)

<p align="center"><img src="https://github.com/kmseunh/blog/assets/105186724/db15686d-bea0-49d5-9b88-c4383d990f53" width="450"></p>

하지만, 예기치 못한 문제가 발생했다. <br>
Flask를 사용하고 있었기 때문에 전역 객체 `g`를 활용하여 로그인 시 해당 ID의 권한 목록을 가져오려 했다. <br>
그러나 전체 접근이 가능한 아이디로 권한 목록을 세션에 저장하려 할 때, 허용된 권한이 5개라면 유저 정보도 5개를 가져와 저장하기 어려운 상황이 발생했다.

또 다른 문제는 JavaScript를 사용해 `selectbox`를 처리할 때, 특정 팀의 데이터만 볼 수 있도록 고정했지만, js 코드 중복으로 인해 의도한 대로 작동하지 않았다. <br>
다른 js 파일에서 `selectbox`를 생성하는 방식 때문에, 렌더링 템플릿으로 적용되지 않았다.

## 얘도 중복, 쟤도 중복

#### 1. 유저 정보 중복 문제 해결

권한 개수만큼 유저 정보를 가져오는 문제를 해결하기 위해 `g` 객체에 권한 데이터를 딕셔너리로 추가하는 방법을 사용했다. <br>
먼저, 로그인 시 아이디와 비밀번호를 통해 유저 정보를 확인하고, 해당 유저의 ID로 DB에서 권한을 조회하여 가져왔다. <br>
마지막으로, 세션에서 사용하기 쉽게 딕셔너리 형태로 변환하여 저장했다.

#### 2. JavaScript 중복 문제 해결

JavaScript가 중복되는 문제는 다른 js 파일에서 `select box`를 생성하기 때문에 렌더 템플릿으로 적용되지 않았다. <br>
해당 함수를 수정하여 일반 유저로 로그인 시 담당 공장만 선택할 수 있도록 변경했다. <br>
구체적으로, `select box`를 동적으로 생성하여 사용자에게 허용된 선택지만 제공하는 방식으로 변경하여 중복 문제를 해결했다.

<hr>

### 배운점

비록 간단한 권한 관리 기능이었지만, 생각보다 신경 쓸 부분이 많았고, 특히 다음과 같은 점을 배웠다.

- 테이블 분리의 중요성: 기능을 한 테이블에 모두 담으려 하지 말고, 세분화하여 관리하는 것이 중요하다는 교훈을 얻었다.
- JavaScript 코드 관리: JavaScript 코드가 중복되지 않도록 신경 써서 관리해야 함을 깨달았다.
권한 관리 기능을 통해 작은 문제라도 놓치지 말고 세심하게 살피는 것이 중요하다는 점을 다시 한번 되새기게 되었다.
