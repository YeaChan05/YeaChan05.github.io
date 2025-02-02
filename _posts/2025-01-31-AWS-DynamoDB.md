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
하나의 테이블에는 거의 무한대에 가까운 아이템을 저장 가능하다
RDBMS의경우 scale up을 해야하지만 DynamoDB는 scale out이 가능해 계속해 데이터를 증가 가능하다

논리적인 개념의 테이블 아래에는 물리적으로 파티션 단위로 관리됨

그리고 이것은 DynamoDB가 내부적으로 파티션을 관리해줌

각 파티션은 초당 1K 쓰기, 3K 읽기, 10GB 데이터 저장이라는 제약조건이 있다

아이템은 여러개의 attribute로 구성됨(RDB기준 column)
Partition Key: 무한하게 많은 파티션 중 조회하려는 아이템의 파티션 위치를 특정지어주는 식별자, equal만 사용 가능
Sort Key: 선택적으로 사용 가능, 동일한 PK 안에서 1:N 관계 모델링 가능, 정렬 가능, 여러개의 condition 연산자 사용 가능


DynamoDB는 파티션에 값을 입력하기 전에 PK값을 HASH하여 최대한 파티션들에 골고루 나누어주도록 노력함

그렇기에 굳이 파티션이 잘 나눠지도록 노력하지 않아도됨

Replication도 유연하게 서로 다른 AZ에 진행되기 때문에 하나의 AZ가 사용 불가능하더라도 안정적으로 읽기/쓰기가 가능하다

제약조건
테이블은 원하는만큼의 처리량 설정 가능
읽기 처리량과 쓰기 처리량을 독립적으로 사용 가능
읽기 단위는 4K, 쓰기는 1K까지 요청당 처리

Eventually Consistent 읽기는 8K까지 읽을수는 있지만 읽기 일관성이 미지수(최근것이 읽힐수도 아닐수도 있음)

하나의 아이템의 최대 크기는 400K
한글짜만 바뀌어도 다시쓰기 때문에 가능한 아이템 크기를 작게 하는것이 용이

각 파티션은 1000WCU/s 또는 3000RCU/s를 초과하면 파티션이 늘어남

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