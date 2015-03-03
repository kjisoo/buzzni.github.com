---
layout: post
title: SQL 최적화 팁 소개
excerpt: SQL 최적화 팁 시작 해보자
frame: http://www.neustar.biz/blog/big-data.jpg
author: roland
email: roland@buzzni.com
tags: sql
publish: false

---

# 1부 데이터베이스 쿼리 최적화 시작

## 쿼리(SQL) 최적화

 데이터 베이스 운영을 하다 보면 슬로우 쿼리*(slow query)*가 발생하게 된다. 이렇게 슬로우 쿼리가 발생되는 가장 큰 이유는 초기 개발 당시에는 적은량의 데이터로 개발하다 보면 성능 최적화된 쿼리를 생각 하지 못 할 경우가 발생하게 된다. 그리고 또 하나의 이유는 약간 느린 슬로우 쿼리가 실제 서버의 환경에서는 슬로우 쿼리로 변할 요지가 충분 하기 떄문에 개발 당시 놓치기 쉬운 문제이다.
 
 예로 하나의 쿼리를 실행 할때 60ms가 소요된다면 적당히 느린 쿼리로 생각할 수 있다. 하지만 실제 서버에서 동접 100명으로 해당 쿼리가 실행되면? 동접 1000명이 해당 쿼리를 실행하면? 갑자기 슬로우 쿼리로 변경되고 전체적인 서비스 속도 악화로 나올 수 있다.
 
그렇기에 초기 개발 당시 쿼리 최적화는 어느정도 해야 하며 서비스 운영 중에도 슬로우 쿼리는 잡아야 서비스 안정화에 도움이 되기 때문에 쿼리 최적화는 필요 하다.

	쿼리 최적화 혹은 데이터베이스 관련하여 팁을 적고자 한다.

_ _ _


#### 인덱스는 Index Range Scan은 빠르고 Full Scan 느리다?

인덱스의 종류는 아래와 같이 여러 종류가 있지만 개발자로서 크게 신경 써야 하는 부분은 Index Range Scan, Index Unique Scan, Index Full Scan이다. 나머지 인덱스 관련해서는 다루지 않겠습니다.

- Index Range Scan
- Index Unique Scan
- Index Full Scan
- Index Fast Full Scan
- Index Skip Scan
- Index Min/Max Scan
- Index Join

Index Range Scan은 group이라는 컬럼에 인덱스가 존재 한다면 해당 인덱스를 통해서 테이블을 조회를 하게 된다. 
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

위와 같이 쿼리를 실행하게 되면 group이 buzzni인 모든 데이터를 출력하게 된다.


Index Unique Scan은 Index Range Scan과 같지만 쿼리의 결과가 존재 하지 않거나 단 하나의 쿼리가 출력이 될 경우 Index Unique Scan를 수행하게 된다.
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

> 위와 같이 수행이 된다면 어느정도 성능은 보장 할 수 있다고 본다.(데이터 추출 범위 및 다른 조건이 없을 경우)



- - -
Index Range scan 인덱스를 통해서 데이터를 검색 할 경우에는 빠르다고 생각 할 수 있다. 그에 반대되는 성향은 Index Full Scan으로 모든 인덱스를 검색하면서 조회를 하게 되는 경우 이다.
> Index Full Scan은 처리 범위가 테이블 크기 만큼 증가하기 때문에 성능저하를 가져온다.

``` sql
SELECT
	ID,
	nickname
FROM
	"user"
WHERE
	nickname = 'roland'
ORDER BY
	"name"
```

위와 같은 쿼리를 실행 할 경우 Index Range Scan으로 동작해야 하지만 ORDER BY "name"이 WHERE절에 포함되지 않기 때문에 Index Full Scan으로 동작되야한다.
> nickname이 roland 데이터만으로는 name이 순서대로 있다는 것을 모르기에 Index Full Scan를 해야 한다.


`만약 불가피 하게 Full Scan를 해야 하는 경우 쿼리에 맞는 인덱스로 정렬이 되어 있다면 오히려 득을 볼 수 있다.`

- 모든 데이터를 참조 해야 하는 경우 테이블보다 크기가 작고 메모리에서 참조가 가능한 인덱스를 액세스 하기 때문에 성능이 향상이 된다.
- 인덱스를 서비스에 맞게 ASC, DESC를 하게 되면 최적화되어 매우빠르게 액세스를 할 수 있다. 예를 들자면 게시판 같은 경우는 최근 글을 많이 보기 때문에 DESC가 성능 향상을 가지고 올 수 있다.
