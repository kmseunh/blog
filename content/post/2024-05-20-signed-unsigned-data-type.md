+++
title = "signed unsigned 데이터 타입"
date = "2024-05-21"
description = "테이블 값 오류 해결하기"
tags = [
    "2024",
    "database"
]
categories = [
    "work"
]
series = ["Worklog"]
+++

항상 출근하면 첫 번째로 하는 일은 대시보드 페이지에 접속해 어제와 현재의 각 공장별 데이터를 확인하여 이상이 없는지 점검하는 것이다.<br>
한동안 이상이 없었던 그래프가 지난주 금요일 데이터만 기하급수적으로 높게 치솟았고, SCADA에서 받아오는 데이터가 너무 높게 나오는 현상이 발생했다.

<p align="center"><img src="https://github.com/user-attachments/assets/fecc2a85-8bc1-44de-b4f7-68601dc4ff8e" width="450"></p>

<!--more-->

## 데이터가 이상하게 저장되다니 완전 럭키비키잔앙

DB 테이블을 조회해 보니 `-1.812019e-41`과 같은 이상한 값이 저장되어 있었다. <br>
SCADA 에서 받아오는 값이 저장되는 거라 이상한 값이 저장되는 부분은 어쩔 수 없지만 데이터를 가져올 때 쿼리문에서 0으로 변환하려고 했다.

```sql
<!-- 사용한 쿼리문 -->
SELECT feedname
    ,max(cast(amount AS unsigned)) amount
    ,'THIS' bdate_type
FROM {table_name}
WHERE bdate IN (
        SELECT DISTINCT bdate
        FROM {table_name}
        WHERE bdate <= % s
            AND bdate > % s
            AND cast(amount AS unsigned) >= 0
        )
GROUP BY feedname
```

데이터베이스 테이블에서 확인된 값 `-1.812019e-41`는 부동 소수점 형식으로 저장되어 있는데, 이 값은 `unsigned`로 형변환할 수 없는 형식으로, `amount` 컬럼이 varchar(20)으로 지정되어 있어서 숫자로 처리할 수 없었다. <br>
( 이러한 이슈로 인해 해당 쿼리문에서는 데이터를 올바르게 처리할 수 없었다. )

## 해결을 하다니 완전 럭키비키니시티 잔앙

쿼리문을 수정하여 발생한 이슈를 해결하려고 했는데, `amount` 컬럼이 `varchar(20)` 형식이었고, `-1.812019e-41` 값만 계속해서 들어오는 문제만 발생했기 때문에, <br>
이를 해결하기 위해 `WHEN amount LIKE '%e-%' THEN 0` 구문을 추가하여 해당 값을 0으로 변경하였고, 또한 `unsigned`를 `signed`로 수정하여 음수 값이 발생할 경우에도 적절히 처리할 수 있도록 했다.

수정된 쿼리문은 다음과 같다.

```sql
<!-- 사용한 쿼리문 -->
SELECT feedname
    ,max(CASE 
        WHEN amount LIKE '%e-%'
            THEN 0
        ELSE cast(amount AS signed)
        END) amount
    ,'THIS' bdate_type
FROM {table_name}
WHERE bdate IN (
    SELECT DISTINCT bdate
    FROM {table_name}
    WHERE bdate <= % s
        AND bdate > % s
        AND cast(amount AS signed) >= 0
    )
GROUP BY feedname
```

<br>
`amount` 컬럼을 `varchar`에서 `float` 또는 `double` 등 숫자 데이터 타입으로 변경하는 것이 더 적합할 수 있다는 생각이 드는데, 서버 DB에서 ORACLE 서버로 데이터를 보내주고 있어서 각 컬럼에 대한 정의를 조율해야 해서 마음대로 변경할 수 없었다. <br>

이번 이슈 처리를 통해 DB의 데이터 타입에 처리에 대해 좀 더 이해할 수 있었고, 멀쩡하게 잘 돌아간다고 안심하고 있으면 안된다는 교훈을 얻었다. <br>
( 항상 긴장감을 늦추지 말자. )
