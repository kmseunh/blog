+++
title = "개인 서버가 있었으면 조켄네"
date = "2024-08-18"
description = "오라클 인스턴스 생성 및 서버 만들기"
tags = [
    "2024",
    "server",
    "oracle"
]
categories = [
    "indiv"
]
series = ["Study"]
+++

내가 만든 서비스를 다른 사람들이 사용하는 모습을 상상하면서, 개인 서버를 구축하고 싶다는 마음이 들었다. <br>
처음에 떠오른 것은 AWS 프리티어였는데, 채용 공고에서 AWS 경험을 중요시하는 경우가 많아 필수로 경험해야겠다고 생각했다. <br>
하지만 이전에 프리티어 계정을 만들었던 경험이 있어, 몇 개월 후 예기치 않게 과금이 발생했던 기억이 있어서 조금 걱정이 되었다.

그러던 중, 오라클 프리티어가 평생 무료로 서버를 제공한다는 매력적인 옵션을 발견했는데, 제한이 있긴 하지만 경제적인 부담 없이 개인 서버를 운영할 수 있다는 점에서 큰 매력을 느꼈다. <br>
그래서 바로 도전해 보기로 결심하고, 과정에서 겪은 여러 시행착오와 배운 점들을 기록하기로 했다.

<p align="center"><img src="https://github.com/user-attachments/assets/f149f134-4264-4f11-80a3-4dd4329dca75" width="400"></p>

<!--more-->

## 계정 만들기

예상치 못한 상황에서 많은 시간을 소모하게 되었는데 바로 계정 생성 과정이었다. <br>
인터넷에는 오라클 인스턴스에 대한 다양한 정보가 있어 비교적 쉽게 진행될 줄 알았지만, 실제로는 5일 동안 계정을 생성하지 못하고 허송세월을 보내게 되었다.

주된 원인은 트랜잭션 오류였는데, 카드 등록을 마친 후에 계정 생성을 완료했지만, 로그인이 되지 않았다. <br>
인터넷에서 “시간을 두고 다시 시도하면 된다”는 정보를 참고해 몇 시간 후 다시 시도해 봤지만 여전히 실패했다.

비밀번호를 찾기 위해 이메일로 전송 요청을 해도 메일이 오지 않았고, 결국 라이브 챗으로 문의를 넣었다. <br>
하지만 시차 문제로 새벽까지 기다리며 번역기를 돌려가며 대화한 결과, 메일을 보내주겠다는 답변을 받았는데, 2일 후에 도착한 메일의 내용은 <span style='color: #2D3748; background-color: #fff5b1'>&nbsp;“Unfortunately, we are unable to resolve this or process the transaction. This is all the information we can provide.”&nbsp;</span> 즉, 해결할 수 없다는 말이었다.

결국 카드와 이메일을 변경한 후 다시 시도했더니 이번에는 빠르게 진행이 되었다. <br>
( 진짜 어이없네,, )

<p align="center"><img src="https://github.com/user-attachments/assets/9de25cf9-2de3-46ff-a53f-c3683e46283d" width="450"></p>

## 서버 만들기

인스턴스를 생성하는 방법은 간단했는데, 다음과 같은 3단계로 요약할 수 있다.

```
ID & 보안 Compartment 생성 → 가상 클라우드 네트워크 VCN 생성 → 인스턴스 컴퓨트 인스턴스 생성
```

인스턴스 생성 시 `Ubuntu 24.04` 이미지와 `VM.Standard.E2.1.Micro(AMD)` 타입으로 두 개의 인스턴스를 생성하려 했다. <br>
각각 WAS와 DB 서버로 사용할 계획이었다. <br>
하지만 `VM.Standard.E2.1.Micro(AMD)`는 _1 CPU 1 GB RAM_ 을 제공하고, `VM.Standard.A1.Flex(ARM)`는 _4 CPU 24 GB_ 를 제공하는 것을 알고 급하게 삭제하고 새로운 인스턴스를 생성하려 했지만 자리가 없어 생성이 되지 않았다.

ARM 인스턴스를 사용하려면 자리가 날 때를 노리거나 유료 계정으로 전환해야 했다. <br>
유료 계정 전환이라는 말이 별로 내키지 않았지만, 전환 후 프리티어 제한을 넘지 않으면 과금이 되지 않는다는 말에 바로 전환을 신청했다. <br>
( 결과는 대 성 공 ~ )

<p align="center"><img src="https://github.com/user-attachments/assets/5bc1cadd-4a52-405e-a259-dedb7406dd18" width="400"></p>

<hr>

이제 `Ubuntu 20.04` 설정을 하고, `Docker`를 세팅할 예정이다.<br>
정말 간단한 부분이지만 재미있어서 시간 가는 줄 모르는 중이다.
