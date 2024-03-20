+++
title = "김승현 세션 DB 사건"
date = "2024-03-19"
description = "김승현이 세션을 생성하고 db에 저장한 사건이다."
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

> 김승현이 SpringBoot로 작업하면서 로그인 시 세션을 생성할 때 DB에 저장해버린 사건을 말한다.

세션(Session)은 웹 애플리케이션에서 사용자의 상태를 유지하고 추적하는 데 중요한 역할을 한다. <br>
일반적으로 세션은 서버 측에서 관리되며, 클라이언트와 서버 간의 통신을 통해 식별된다고 한다.

이러한 세션 기반 로그인 기능을 구현하는 중에 세션의 관리 및 수명에 대한 부분을 잘 이해하지 못해서 바보 같은 짓을 벌이고 말았다.

<p align="center"><img src="https://github.com/kmseunh/FastAPI/assets/105186724/d2724ff5-7b00-44d3-9239-299712ddfe04" width="450"></p>

<!--more-->

## 어떤 이슈를 만들었나

SpringBoot에서 HttpSession을 사용하여 세션을 관리하는 코드를 작성하고 있었는데 로그인 시 세션이 생성되고 DB에 저장되도록 구현했다. <br>
다 끝낸 것 같았지만, 실행 중인 창을 닫으면 세션이 삭제되지 않는 문제가 발생했다. <br>

```java
@PostMapping("/login")
public String login(User user) {
    User existingUser = userMapper.findByUsername(user.getUsername());
    if (existingUser != null && existingUser.getPassword().equals(user.getPassword())) {
        // 로그인 성공 시 세션을 생성하고 DB에 저장
        String sessionKey = httpSession.getId();
        Session session = new Session(existingUser.getUserId(), sessionKey);
        sessionMapper.insert(session);
        return "redirect:/";
    } else {
        return "login";
    }
}
```

보통 HttpSession은 사용자가 웹 애플리케이션에 접속할 때 생성되고, 브라우저를 닫거나 세션을 명시적으로 종료할 때 소멸되는데, DB에 세션을 저장하면 세션이 브라우저가 닫히거나 세션이 명시적으로 종료되지 않는 한 영구적으로 유지될 수 있다고 한다.

일반적으로 보통 세션의 수명은 세션 생성 시 설정할 수 있으며, 기본적으로 브라우저가 닫힐 때 세션이 종료되도록 설정된다. <br>
하지만 DB에 세션을 저장하는 경우, 이러한 종료 조건을 적용하기가 어려울 수 있다. <br>
따라서 세션의 수명을 관리하고 세션을 종료하는 방법에 대해 추가적인 고려가 필요하다.

## 그럼 언제 DB에 저장하나요??

세션 데이터를 DB에 저장하여 관리하는 것이 유용할 때도 있다. <br>
이를 통해 세션 데이터의 영속성, 클러스터 환경에서의 세션 공유, 그리고 세션 관리의 확장성과 안정성을 향상시킬 수 있다고 한다.

- 사용자의 로그인 후 일정 시간 동안 세션 데이터를 유지해야 할 때 DB에 저장하여 영속성을 확보할 수 있다.
- 클러스터 환경에서 여러 서버 간에 세션을 공유해야 할 때 DB에 세션 데이터를 저장하여 일관된 세션 상태를 유지할 수 있다.
- 대규모 웹 애플리케이션에서 DB에 세션 데이터를 저장하면 서버의 확장성을 높이고 안정성을 확보할 수 있다.

세션을 DB에 저장하는 것은 유용하지만, 성능과 보안에 대한 고려 사항이 있으므로 프로젝트의 요구 사항과 환경을 고려하여 결정해야 한다고 한다.
