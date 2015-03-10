---
layout: post
title: SQL 최적화 팁 소개
excerpt: SQL 최적화 팁 시작 해보자
frame: http://www.neustar.biz/blog/big-data.jpg
author: roland
email: roland@buzzni.com
tags: sql, query optimization, index
publish: true
---

# 시작하며 

## 1부 데이터베이스 쿼리 최적화 시작

데이터 베이스 운영을 하다 보면 슬로우 쿼리*(slow query)*가 발생하게 됩니다.

이렇게 슬로우 쿼리가 발생되는 큰 이유는 초기 개발 당시 적은량의 데이터로 개발하다 보면 쿼리 최적화를 소홀히 하게 되기 때문입니다.

그리고 또 하나의 이유는 약간 느린 쿼리가 실제 서버 환경에서는 슬로우 쿼리로 변할 여지가 충분하기 떄문에 개발 당시 놓치기 쉬운 문제입니다.
 
예로 하나의 쿼리를 실행 할때 60ms가 소요된다면 적당한 쿼리로 생각할 수 있습니다.

하지만 동접 100명, 1000명, 10000명이 해당 쿼리를 동시에 실행 하게 되면 갑자기 슬로우 쿼리로 변경되고 전체적인 서비스 속도 악화를 야기할 수 있습니다.
 
그렇기에 개발 초기부터 어느정도의 쿼리 최적화를 해야 하며, 서비스 운영 중에도 슬로우 쿼리는 잡아야 서비스 안정화에 도움이 되기 때문에 쿼리 최적화는 필요 합니다.

본 글에서는 쿼리 최적화 및 데이터베이스에 관련하여 팁을 적고자 합니다.

_ _ _


### 인덱스는 Index Range Scan은 빠르고 Full Scan 느리다

- Index Range Scan
- Index Unique Scan
- Index Full Scan
- Index Fast Full Scan
- Index Skip Scan
- Index Min/Max Scan
- Index Join

인덱스의 종류는 위와 같이 여러 종류가 있지만 개발자로서 크게 신경 써야 하는 부분은 

* Index Range Scan
* Index Unique Scan
* Index Full Scan

입니다. 나머지 인덱스 관련해서는 다루지 않겠습니다.

Index Range Scan은 group이라는 컬럼에 인덱스가 존재 한다면 해당 인덱스를 통해서 테이블을 조회를 하게 됩니다.

> ID, group 컬럼에 각각 인덱스가 존재 한다는 가정하에 진행합니다.

##### Index Range Scan

``` sql
SELECT
	ID,
	nickname,
	group
FROM
	"user"
WHERE
	group = 'buzzni'
```

위와 같이 쿼리를 실행하게 되면 group이 buzzni인 모든 데이터를 출력하게 됩니다.

즉 한개의 데이터가 아닌 다중 데이터를 조회 할때는 Index Range Scan으로 쿼리가 실행하게 됩니다.


##### Index Unique Scan

``` sql
SELECT
	ID,
	nickname,
	group
FROM
	"user"
WHERE
	ID = 1
```

반면 위와 같이 단일 결과물 추출하는 쿼리가 수행이 된다면 Index Unique Scan 실행계획으로 동작하게 됩니다.

이렇게 Index Unique Scan으로 수행하게 될 경우 일정 성능은 보장 할 수 있다고 봅니다.
 

##### Full Scan

일반적인 Index Scan과 반대되는 성향인 Full Scan은 모든 인덱스 혹은 데이터를 검색하면서 조회를 하게 됩니다.
 
> Full Scan은 처리 범위가 테이블 크기 만큼 증가하기 때문에 성능저하를 가져온다.

``` sql
SELECT
	MIN(ID), MAX(ID), AVG(ID)
FROM
	"user"
WHERE
	ID IS NOT NULL
```

위의 쿼리를 수행하게 되면 ID의 실제 값이 어떤 값인지 모르기 때문에 불가피 하게 Full Scan를 해야 하는 경우가 발생한다.
 
- 모든 데이터를 참조 해야 하는 경우 테이블보다 크기가 작고 메모리에서 참조가 가능한 인덱스를 액세스 하기 때문에 성능이 향상이 됩니다.

- MIN, MAX는 인덱스에서 실제 값이 첫 부분과 마지막 부분이기에 불필요한 검색 없이 검색 할 수 있습니다.

- - -


#### PostgreSQL 예제

저희가 주로 사용하는 PostgreSQL에서는 쿼리 옵티마이저가 상당히 발전하여 최적화가 잘 되어 있습니다.

``` sql
SELECT
	MIN(ID)
FROM
	"user"
```

*Index Only Scan using* 로 쿼리 수행하여 한번의 데이터를 추출하는 모습을 볼 수 있습니다.

``` sql
SELECT
	MAX(ID)
FROM
	"user"
```

반대로 MAX기능을 수행 시 *Index Only Scan Backward using* 로 쿼리 수행하여 단 하나의 데이터를 추출 합니다.


## 정리하며

쿼리 최적화라는 주제를 시작하기 앞서서 어떤 주제로 어떤 예제로 작성해야 하는지 생각이 많았습니다.

두서 없이 위에서 여러 내용을 한번에 설명 하였지만 쿼리 최적화에 대한 중요성을 표현 하고 싶었습니다.

예를 들자면 데이터베이스 마다 처리방식에 따른 성능, 버전별 마다 수행되는 방법 등에 따라

똑같은 쿼리를 막연하게 사용해서는 안된다는 점입니다. 
 
다음글부터는 순차적으로 예제와 쿼리 최적화에 대한 내용 및 데이터베이스 내용을 담고자 합니다.
