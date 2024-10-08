+++
title = "데이터 증가 시 대처 방법"
date = "2023-08-21"
description = "데이터가 늘어날수록 처리를 빠르게 해야한다"
tags = [
    "2023",
    "data"
]
categories = [
    "indiv"
]
series = ["Study"]
+++

주간 회의 시간에 대표님께서 항상 말씀하시는 것이 있다. <br>
*현재 맡고 있는 프로젝트에 대해 항상 더 발전시키고 향상시킬 생각을 하자.* 라는 말인데 이를 내가 맡은 프로젝트에 적용해 보면 어떤 부분이 있을까 하는 생각이 들었다. <br>
고민을 해보니 대시보드 특성상 데이터가 늘어나고 이에 따라 처리가 느려지는 이슈가 생각이 났다.
<br>

데이터가 늘어날수록 대량의 데이터를 빠르게 저장하고 검색하는 것은 쉽지 않으며, 이로 인해 성능 저하와 함께 데이터 분석에 소요되는 시간이 길어지는 문제가 발생한다. <br>
검색해 보니 이에 대한 해결책으로 일라스틱 서치와 키바나의 조합이 빛을 발하고 있다고 한다.
<br>

따라서 이번 글에서는 위 두가지에 대해 정리해보려 한다.

<p align="center"><img src="https://github.com/user-attachments/assets/c8600d59-8178-419c-ad4f-5b71930f234c" width="400"></p>

<!--more-->

## 일라스틱 서치

일라스틱 서치(Elasticsearch)와 키바나(Kibana)는 주로 데이터 분석 및 시각화를 위해 사용되는 오픈 소스 소프트웨어라고 한다. <br>
두 제품은 주로 로그 및 이벤트 데이터와 같은 대량의 데이터를 저장, 검색 및 시각화하기 위해 설계되었고, Logstash라는 다른 오픈 소스 도구와 결합하여 사용될 때 전체적인 솔루션을 형성한다고 한다.
>> 이들은 일반적으로 ELK 스택이라고도 부른다.

### 일라스틱 서치의 역할

1. 빠른 검색 및 분석 엔진 <br>

    - 일라스틱 서치는 실시간 검색 및 분석 엔진으로, 데이터의 색인화와 검색을 빠르게 처리할 수 있다.
    - 대용량 데이터에 대한 탁월한 성능을 보장하며, 확장성 있는 아키텍처를 통해 데이터 양이 증가하더라도 높은 성능을 유지할 수 있다.

2. 분산 아키텍처 <br>
    - 일라스틱 서치는 데이터를 분산하여 저장하고 처리함으로써 데이터 양의 증가에 유연하게 대응할 수 있다.
    - 클러스터링과 샤딩을 통해 데이터의 안정성과 가용성을 보장하면서도 성능을 향상시킨다.

<br>

## 키바나의 시각화와 인터랙션

1. 시각화 도구로 데이터 이해 <br>
    - 키바나는 Elasticsearch에서 수집된 데이터를 시각적으로 표현할 수 있는 강력한 도구이다.
    - 다양한 차트와 그래프를 활용하여 데이터의 흐름과 경향을 빠르게 파악할 수 있다.

2. 대시보드 구축 <br>
    - 키바나를 통해 직관적이고 사용자 친화적인 대시보드를 구축할 수 있다.
    - 이를 통해 데이터의 상태를 실시간으로 모니터링하고 필요한 정보에 빠르게 접근할 수 있다.

<br>

지금 당장 두 가지를 배우고 사용할 수는 없지만 나중에 프로젝트가 마무리되고 더 개선할 수 있을 것 같다는 생각이 들어 추후에 공부해 보고 싶다.
