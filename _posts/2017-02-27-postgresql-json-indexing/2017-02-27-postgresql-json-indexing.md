---
layout: post
title: PostgreSQL로 비정형 데이터 인덱싱하기
tags: [postgresql, json, data, indexing]
author: 부요한
image: assets/img/character/yohan.png 
type: DATAINFRA
---


# 시작하며

안녕하세요. 버즈니 데이터인프라팀의 요한입니다. 몇 년 전 대용량 데이터를 저장하고 다루기 위해 NoSQL 돌풍이 불었던 것을 기억하시나요? 그 이후에 기존에서 비정형 데이터를 지원하지 않던 상당 수의 RDBMS들이 버전업과 함께 JSON 형식을 통해 비정형 데이터를 다루는 것을 지원하고 있습니다. 덕분에 기존 개발자들에게는 비교적 익숙할 RDBMS와 SQL Query를 이용하여 비정형 데이터를 다루는 것이 가능해 졌으나, 정작 이를 활용하려고 보면 새로운 연산자를 배워야 함은 물론이고 MongoDB와 같은 NoSQL 솔루션에 비교하여 성능 이슈가 검증되지 않아 실제로 RDBMS에서 비정형 데이터를 직접 다루는 경우가 적은 것으로 알고 있습니다. 그래서 이번엔 제가 한 번 RDBMS에서 비정형 데이터 인덱싱 성능과 용도에 따른 인덱스 방법에 대해 테스트해 보았습니다. 

### 

### PostgreSQL

이번 테스트는 PostgerSQL로 진행되었으며, 시작하기에 앞서 이를 간단히 소개 드립니다. PostgreSQL은 1983년 버클리 대학에서 만들어진 RDBMS인 Ingres를 전신으로 하는 오픈소스 RDBMS 입니다. 전신 프로젝트의 상업적 실패와 함께 오픈소스화 된 케이스인데요. 덕분에 연구분야에서 활발히 사용되어 왔고, Extension 시스템이 잘 구성되어 있어 PostgreSQL을 기반으로 한 수많은 부가기능들이 있습니다. 특히 9.x 대 에서는 성능 이슈에 집중하여 웬만한 상업용 RDBMS에서 뒤지지 않는 성능까지 갖춘 RDBMS입니다. 그리고 이번 포스트에서 다룰 비정형 데이터에 대해서는 2012년 9.2 버전부터 JSON을, 2014년 9.4 버전부터 JSONB를 지원하고 있어서 이를 이용하여 보고자 합니다. 

### 

### JSON vs JSONB

PostgreSQL은 비정형 데이터를 위해 JSON과 JSONB 자료형을 지원하며, 일반적인 차이점은 다음과 같습니다. 

JSON
JSONB

저장 방식
TEXT
Binary object

디스크 용량
작음
큼 

Insert/Update
빠름 
느림 

Select
느림 
빠름
이번 포스트에서는 JSON과 JSONB의 성능 차이에 대해서도 다루려고 해서 두 자료형 모두 사용하여 테스트를 진행했습니다. 자세한 차이점은 테스트 내용을 확인하면서 알아보려고 합니다. 이외에도 Array같은 배열형이나, HStore 같은 Key-Value store 같이 다른 형태의 비정형 자료형도 지원하고 있으나 이번 포스트에서는 JSON과 JSONB만 다루도록 하겠습니다. 

### B-Tree vs GIN

PostgreSQL은 일반적인 DBMS들과 같이 B-Tree 기반 인덱싱을 기본값으로 가집니다. 하지만 Oracle에 비견될만한 기능으로도 유명한 만큼 이외에도 다양한 방식의 인덱싱 방식을 사용할 수 있으며 이번 포스트에서는 그 중에서 GIN Index를 활용해 보려고 합니다. GIN는 Generalized Inverted Index의 약자로, 기본적으로 Full Text Search를 위해 만들어진 인덱스입니다. B-Tree 인덱싱은 데이터의 값을 비교 가능한 트리 형태로 저장하여 순차적 비교가 가능한 반면 LIKE '%' 연산과 같이 데이터 중간에 속한 값에 대한 인덱스 적용이 힘든 부분을 보입니다. 반면 GIN은 데이터 내의 각 요소, 즉 문장이라면 각 단어, 비정형 데이터라면 각 요소들을 인덱싱하여 키워드의 등장 위치와 상관 없이 해당 키워드를 사용하여 찾고자 하는 값이 데이터의 중간에 등장하더라도 인덱스를 적용할 수 있습니다. 예를 들어 "Hi, I am Yohan"이라는 데이터가 있고 "Yohan"이라는 키워드를 이용하여 데이터를 찾고 싶은 경우 B-Tree 의 경우 인덱스 적용이 되지 않아 순차 탐색을 해야하지만, GIN의 경우에는 인덱스가 적용되어 빠르게 검색이 가능합니다. 하지만 장점이 있으면 단점도 있어서 GIN의 경우 B-Tree 에 비해 인덱싱 속도가 좀 더 느리고 디스크 용량도 더 차지하게 됩니다. 

# 테스트 준비

### 실행 환경

PostgreSQL
9.6.2 from https://hub.docker.com/_/postgres/

Host
Ubuntu 16.04.4

Intel(R) Core(TM) i7-6700K CPU @ 4.00GHz

DDR3 32GB RAM

200GB SSD
본 포스트는 모두 위와 같은 사양에서 진행되었습니다. PostgreSQL은 Dockerize 된 공식 이미지에서 최신 버전인 9.6.2를 받아서 사용했으며, 별도의 성능 튜닝은 하지 않았습니다. 

### 

### 데이터

테스트를 위한 데이터는 버즈니에서 운영중인 홈쇼핑 모아의 API log 로부터 가져 왔습니다. 총 12,736,719 건의 데이터가 사용 되었으며 형식은 다음과 같습니다.
    
    
    {  
       "url":"/some/path",
       "params":{  
          "key":"value",
          …
       },
       "size":512,
       "date":"2016-12-31T08:27:38+09:00",
       "agent":"\"Mozilla/5.0\"\n",
       "time":0.0,
       "ip":"123.234.123.234"
    }

일반적인 웹 로그와 유사하게 고정으로 등장하는 필드들이 있고, prams 필드의 경우에는 그 내용이 가변적입니다. 

### DB Schema 및 성능 측정

PostgreSQL 상에 작성된 테이블은 사이드 이펙트를 죄소화 하기위해 다음과 같이 JSON/JSONB 단일 컬럼으로 생성 되었습니다. 
    
    
    CREATE TABLEtable_name (dict JSON|JSONB );