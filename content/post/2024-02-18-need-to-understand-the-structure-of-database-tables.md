+++
title = "테이블구조를 안본건 너무 심했잖아.."
date = "2024-02-18"
description = "데이터를 DB에 저장할 때 테이블 구조를 잘 보자"
tags = [
    "2024",
    "database"
]
categories = [
    "work"
]
series = ["Worklog"]
+++

기존에 진행한 프로젝트에 계정 관리 페이지를 만들어 달라는 요청이 있었다. <br>
이전에 비슷한 작업을 많이 해보았기 때문에 큰 어려움 없이 개발할 수 있을 거라고 생각했다. <br>

하지만 로그에 에러가 계속 찍혀서 확인해 보니 회원가입 양식에 정보를 입력하고 제출하면, 해당 정보를 데이터베이스에 저장하는 과정에서 문제가 발생한 것이었다. <br>
(기존 로그들이 너무 많아서 정신 나가는 줄...)

<p align="center"><img src="https://github.com/kmseunh/kmseunh/assets/105186724/44fdb377-9ae9-4ff9-91dd-ee4383eea7ae" width="450"></p>

<!--more-->

## 테이블구조를 안본건 너.. 무심했잖아

테이블 구조는 다음과 같았다.

```sql
CREATE TABLE users (
    member_id INT AUTO_INCREMENT PRIMARY KEY,
    user_id VARCHAR(20) NOT NULL,
    name VARCHAR(20) NOT NULL,
    password VARCHAR(100) NOT NULL,
    email VARCHAR(50) UNIQUE NOT NULL
);
```

위와 같이 데이터베이스의 테이블 구조를 정의했지만, 클라이언트 측에서는 다음과 같은 코드를 사용하여 데이터를 서버로 전송하고 있었다.

```js
$('#addBtn').on('click', function() {
        let USERID = $('#add_USERID').val();
        let NAME = $('#add_NAME').val();
        let PASSWORD = $('#add_PASSWORD').val();
        let EMAIL = $('#add_EMAIL').val();
        
        let addedData = {
            USERID: USERID,
            NAME: NAME,
            PASSWORD: PASSWORD,
            EMAIL: EMAIL,
        };

        $.ajax({
            url: '/api/account/save',
            type: 'POST',
            contentType: 'application/json',
            data: JSON.stringify(addedData),
            success: function(response) {
                window.location.reload();
            },
            error: function(xhr, status, error) {
                console.error(error);
            }
        });
    });
```

문제는 사용자 입력값과 데이터베이스의 테이블 구조가 일치하지 않아 발생했다. <br> 사용자가 입력한 데이터의 길이가 데이터베이스의 컬럼 길이 제한을 초과했기 때문에 발생한 문제였던 것이다.

예를 들어, 사용자가 입력한 이름(`name`)이 20자를 초과하거나 비밀번호(`password`)가 100자를 초과하면 데이터베이스에 저장되지 않는다.

## 해결 방안

해결 방법은 입력값의 길이를 제한하는 것이다. <br> 사용자가 입력한 데이터의 길이를 제한하고 이에 대한 예외 처리도 해야 한다.

### HTML 코드 수정

```html
<input type="text" id="add_USERID" class="form-control" maxlength="50">
<input type="password" id="add_PASSWORD" class="form-control">
<input type="text" id="add_NAME" class="form-control" maxlength="50">
<input type="email" id="add_EMAIL" class="form-control">
```

`<input>` 태그의 `maxlength` 속성을 사용하여 입력값의 길이를 제한했고, `type="password"`와 `type="email"`을 사용하여 해당 필드의 형식을 명시함으로써 사용자가 올바른 형식의 데이터를 입력하도록 유도했다.

```md
`type="email"` 속성을 사용하는 경우, 이메일 주소의 최대 길이를 제한하는 것은 일반적으로 
권장되지 않는데, 그 이유는 이메일 주소의 최대 길이가 정해져 있지 않고, 일부 이메일 시스템은 
긴 주소를 처리할 수 있기 때문이라고 한다.
```

&nbsp;

### JavaScript 코드 수정

```js
let USERID = $('#add_USERID').val();
let PASSWRD = $('#add_PASSWORD').val();
let NAME = $('#add_NAME').val();
let EMAIL = $('#add_EMAIL').val();

if (!USERID || !PASSWRD || !NAME || !EMAIL) {
    alert('값을 입력해 주세요.');
    return; 
}
```

`if` 문을 추가하여 사용자가 입력 필드를 채우지 않은 채로 제출하려고 할 때 발생하는 예외 상황을 처리하도록 했다.

&nbsp;

### Python 코드 수정

```python
import hashlib

def hash_password(password):
    hashed_password = hashlib.sha256(password.encode()).hexdigest()
    return hashed_password
```

사용자가 등록할 때 비밀번호를 해시화하여 데이터베이스에 저장한다. <br> 이렇게 하면 데이터베이스에는 사용자의 실제 비밀번호가 저장되지 않으므로, 보안이 향상된다.

<hr>

이 경험을 통해 데이터베이스 테이블 구조를 설계할 때 입력값과의 일치성을 반드시 고려해야 한다는 것을 깨달았다. <br> 데이터베이스의 테이블 구조를 정의할 때에는 사용자가 입력할 데이터를 고려하여 세심하게 설계하는 것이 중요하고, 입력값과 테이블 구조의 일치성을 확보함으로써 데이터의 무결성과 보안을 보장할 수 있다고 한다.
