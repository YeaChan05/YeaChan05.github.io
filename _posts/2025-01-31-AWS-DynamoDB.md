---
title: AWS-DynamoDB
date: 2025-01-31
categories: AWS
tags:
  - AWS
  - NoSQL
  - DynamoDB
toc: true
pin: true
sitemap:
  changefreq: daily
  priority: 1
image:
  path: assets/img/posts/2025-01-31-AWS-DynamoDB/3ae1eca91844b3fef96d987403c42639_MD5.jpeg
---
동아리 지원 플랫폼을 개발하며 DynamoDB를 적용해봤다.

그 과정에서 RDBMS로는 감당하기 어려운 쓰기 작업 부하 문제를 만난 덕분에, 자연스럽게 DynamoDB를 선택하게 된 이유와 그 개념, 그리고 DynamoDB의 Single Table Design에 대해 정리해본다.


## 임시저장 기능과 RDBMS의 한계


동아리 지원 플랫폼을 개발하면서 사용자들이 지원서를 작성 중간에 임시 저장할 수 있는 기능을 구현해야 했다. 그러나 매번 임시 저장할 때마다 RDBMS에 데이터를 영속화하는 과정에서 몇 가지 문제가 발생했다.

-  과도한 쓰기 작업: 임시 저장 기능은 사용자가 입력할 때마다 빈번하게 데이터베이스에 쓰기 작업을 발생시킨다. RDBMS는 이러한 반복적인 쓰기 작업에 부하를 많이 받게 되어 성능 저하와 비용 상승이 발생할 수 있다.
- 비정형 데이터에 대한 요구사항: 지원서의 형태가 정형화되지 않을 가능성이 있었다. 지원서의 질문 형태가 지원 직군에 따라 다양하게 존재할 수 있고, 지원 기수에 따라 그마저도 변경 가능하다고 한다.
- 데이터 적재 수준 미지수: 임시저장 지원서에 대한 저장이 어느정도 수준으로 지속되야 할지에 대한 요구사항이 명확히 나오지는 않았다. batch process를 사용해 주기적으로 임시저장본을 삭제할지, 아니면 계속 아카이빙을 할지는 요구사항이 명확히 나오지 않았다. RDBMS로도 지속 증가하는 데이터에 대해 대처 가능하지만 scale up 방식을 주로 사용하던가 아니면 클러스터화에 대한 기술적 고민이 필요해보인다. 즉, 관리 차원에서 기술부채가 상당히 증가한다.

<br><br>

하지만

1. 쓰기 작업 부담 해소 & 스케일 아웃
	- 무한에 가까운 아이템 저장: 하나의 테이블에 거의 무한대에 가까운 아이템을 저장할 수 있다. RDBMS가 하드웨어 성능 향상(스케일 업)에 의존하는 반면, DynamoDB는 자동으로 파티션을 추가(스케일 아웃)해 데이터가 계속 증가해도 문제없이 처리할 수 있다.
	-  완전 관리형: 관리 차원의 기술 이해도가 너무 높지는 않다. 생각보다 기술 설명을 공식 홈페이지나 유튜브에 잘 정리되어있다.
	- 상당한 조회속도: 단건 조회에 10ms 미만의 성능을 보장해주기에 임시저장본을 불러오는 기능을 구현하는데에 꽤나 유용하다.

2. 비용 효율 & 프리 티어
	- 프리 티어: AWS의 프리 티어를 활용해 초기 개발 및 테스트 단계에서 비용 부담 없이 사용 가능하다.
	- 처리량 설정: 테이블 별로 원하는 만큼의 읽기/쓰기 처리량을 독립적으로 설정할 수 있어, 실제 운영 환경에 맞춰 유연하게 조정할 수 있다.
<br><br>

같은 이점이 있기 때문에 DynamoDB를 선택하게 되었다. (사실 100% 적합한 상황이라기 보다는 기술적 경험을 위한것도..)

### DynamoDB의 아키텍처와 기본 개념
  

DynamoDB는 논리적인 테이블 아래에 물리적으로 **파티션** 단위로 데이터를 관리한다. 이 구조 덕분에 데이터의 증가에도 유연하게 대응할 수 있다.


**파티션 관리 및 제약조건**

- **자동 파티셔닝:** DynamoDB는 테이블에 입력되는 각 아이템의 **Partition Key(PK)** 값을 해싱하여 여러 파티션에 골고루 분산 저장한다. → 개발자가 굳이 파티션 분배를 신경 쓸 필요가 없다.

<br>
<br>

### 용량 유닛
- 프로비저닝 모드에서 동작 단위
- WCU: 쓰기 용량 단위, 초당 1KB를 한번 쓸 수 있다
- RCU: 읽기 용량 단위, 초당 4KB를 한번 읽을 수 있다.

이때 읽기 방식에는 세가지가 있는데,
- strongly consistent read: 항상 최신 데이터를 반환해야하기 때문에, 4 KB까지 1RCU 필요.
- eventually consistent read: 즉각적 조회가 아니기에, 4 KB까지의 항목에 대해 0.5RCU 필요.
- transactional read: 한번에 여러개를 동시에 읽고 수정 가능. 4 KB까지 2RCU 필요.

용량유닛은 비용과 직결되니 필요한만큼 사용해야한다.


하지만 나의 경우에는 요청 쇄도 시기를 정확히 측정하기에 어렵다고 생각되어 일단 온디맨드를 사용했다.


### DynamoDB Key

DynamoDB는 아이템을 식별하기 위해 Primary Key를 사용하는데, 이 Key에는 두가지 종류가 있다

1. Partition Key
	- 아이템의 저장 파티션을 결정짓는 요소로, 해싱되어 여러 파티션에 물리적으로 분산된다.
	- 파티션을 지정해주는데에 필요하기에 필수적으로 요구된다
2. Sort Key
	- 복합 Primary Key를 구성하는 두번째 Key로, 필수적이지 않다
	- 정렬이나 기타 DynamoDB에서 지원하는 조건에 사용 가능하다

뿐만아니라 index도 존재한다
DynamoDB에서는 이를 Secondary Indexes라 부른다
- GSL(Global Secondary Index)
	- 기본 테이블과 별도로 독립적인 Partition Key 및 Sort Key를 지정할 수 있는 인덱스
	- 데이터 모델링의 유연성을 크게 높여주며, 다양한 쿼리 요구사항에 대응
	- 기본적으로 eventual consistency(최종적 일관성)를 제공하지만, 옵션에 따라 강한 일관성을 선택 가능
- LSI(Local Secondary Index)
	- 기본 테이블 생성 시 함께 정의되어야 하며, 동일한 Partition Key를 공유하면서 다른 Sort Key를 제공하는 인덱스
	- 테이블 생성 이후에는 생성 불가능
	- 강한 일관성을 지원하며, 최신 데이터 기반 조회 가능



### DynamoDB Key 디자인 패턴


<script src="https://giscus.app/client.js"
        data-repo="YeaChan05/YeaChan05.github.io"
        data-repo-id="R_kgDONnXleQ"
        data-category="Announcements"
        data-category-id="DIC_kwDONnXlec4Cl2a5"
        data-mapping="pathname"
        data-strict="0"
        data-reactions-enabled="1"
        data-emit-metadata="0"
        data-input-position="bottom"
        data-theme="preferred_color_scheme"
        data-lang="ko"
        crossorigin="anonymous"
        async>
</script>