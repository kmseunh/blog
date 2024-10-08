+++
title = "싱글벙글(?) 쿼리문 체험기"
date = "2023-05-05"
description = "데이터베이스에서 원하는 데이터 검색 이슈"
tags = [
    "2023",
    "database",
    "issue"
]
categories = [
    "work"
]
series = ["Worklog"]
+++

데이터베이스에는 프로젝트에 필요한 모든 데이터들이 존재한다. <br> 저장된 데이터들을 그대로 사용하는 경우도 있고 필요한 컬럼들을 가져오거나 데이터를 합치거나 다양하게 수정해서 원하는 데이터로 변환한 후 사용한다.

이것이 바로 내가 백엔드 개발자로 진로를 정한 이유 중 하나였는데, 지금까지 CRUD 쿼리문을 사용하고 테이블에 저장된 컬럼들을 가져오는 것만 경험해 봤기 때문에 데이터 쪽을 너무 쉽게 생각하고 있었다. <br> (어떤 프로젝트든 한 가지 테이블만 사용하는 경우는 없으니까..)

<p align="center"><img src="https://github.com/kmseunh/blog/assets/105186724/56645e45-defc-4b60-87ca-24f866c9d4a1" width="500"></p>

<!--more-->

## 무수히 이어진 SQL 쿼리문

처음 백엔드 쪽 파이썬 코드를 봤을 때 당황스러웠다.

특정 함수의 stmt 변수에 SQL 쿼리문이 담겨 있고, 해당 쿼리를 실행하여 결괏값을 변수에 저장하는 Python 코드들이었는데 우선 SQL 쿼리문 부터 굉장히 길고 복잡해 보였다.

아래는 Python 코드 중 대시보드를 조회하는 쿼리문의 구조를 나타낸 코드이다.

```sql
SELECT 
    type, 
    IFNULL(group2, '전체') AS group2, 
    cycle, 
    b_date, 
    AVG(column6) AS avg_column6, 
    AVG(column7) AS avg_column7, 
    SUM(column8) AS sum_column8, 
    SUM(column9) AS sum_column9 
FROM 
    table_name 
WHERE 
    type = '타입이름 1' 
    AND (
        b_date = '%s' 
        OR (
            b_date BETWEEN DATE_FORMAT(
                LAST_DAY(
                    DATE_SUB('%s', INTERVAL 1 MONTH)
                ), 
                '%Y-%m-01'
            )
            AND LAST_DAY(
                DATE_SUB('%s', INTERVAL 1 MONTH)
            )
        )
    ) 
GROUP BY 
    type, 
    group2, 
    cycle, 
    b_date;
```

<br>
위 쿼리문은 다음과 같은 기능을 수행한다.

1. `table_name` 테이블에서 `type`을 '**타입이름 1**'로 설정하고, `group2` 값이 비어있으면 '**전체**'로 대체하며, 특정 날짜 범위와 '**cycle 이름**'을 충족하는 데이터를 추출한다.

2. 두 부분의 데이터 추출을 하나로 결합한다. <br> 첫 번째 부분은 `b_date`를 입력 날짜('**%s**')와 비교하고, 두 번째 부분은 입력 날짜의 전 월의 마지막 날부터 전 월의 첫 날까지의 범위를 검색한다.

3. 각 `type`에 대해 `b_date`를 형식화하고 '**컬럼 6**' 및 '**컬럼 7**'의 평균, 그리고 '**컬럼 8**' 및 '**컬럼 9**'의 합계를 계산한다.

4. 최종 결과 집합에 `type`, `group1`, `group2`, `cycle`, `b_date`, 계산된 평균 및 합계 값들을 포함하여 결과를 반환한다.

<hr>

Python 코드를 보고 나도 모르게 감탄해버렸다.

출근하고 일을 배우면서 이런 경우가 2번이 있었는데,
첫 번째는 지저분하고 나열되어 있는 코드를 함수를 사용해서 깔끔하게 분리하는 리팩토링을 했을 때였고,
두 번째는 위 쿼리문을 보고 SQL에 직접 입력해서 데이터를 조회했을 때다.

깔끔하게 코드를 분리하여 가독성을 높이는 부분은 어느 정도 할 수 있지만 쿼리문은 많은 공부가 필요할 것 같다.

단순하게 생각하면 '_SQL 쿼리문을 실행하여 결과 값을 변수에 저장한 후, json_data의 값을 기반으로 데이터베이스에서 쿼리를 실행하여 결과를 반환한다._’ 라고 할 수 있다. <br> SQL 문법을 정확히 이해하고 사용할 줄 알아야 원하는 데이터를 가져올 수 있고, 그래야 결과를 반환하기 위한 파이썬 코드를 구현할 수 있을 것 같다. <br> (넓게는 2가지 부족한 부분을 채워야 하고 구체적으로 들어가면…)

<p align="center"><img src="https://github.com/user-attachments/assets/72260fb8-47bc-44cd-8812-d5849f51c957" width="500"></p>

매일매일 부족한 부분을 발견하면서 해야 할 공부가 쌓여가는데, 배우는 내용도 많아지고 있다. <br> 특히 DB는 내가 좋아한다고만 하고 제대로 공부해 보려 하지 않았기 때문에 더 어렵다고 느껴진다.

이제라도 중요하다는 것을 깨달아서 다행이라고 생각하고 조금 더 비중을 두고 공부해야겠다. <br> 그래야지 나에게 데이터 처리 코드나 데이터베이스 코드를 코딩할 기회가 생길 수 있을 것 같다.

**_아니면 그냥.. ‘제가 해볼게요’라고 할까..??_**
