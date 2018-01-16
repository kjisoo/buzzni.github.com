---
layout: post
title: PostgreSQL 비정형 vs 관계 모델 데이터
tags: [postgreSQL, json, relation, model, query]
author: 부요한
image: assets/img/character/yohan.png 
type: DATAINFRA
---

# 시작하며

안녕하세요. 버즈니 데이터인프라팀의 요한입니다. 지난번 포스팅에서는 PostgreSQL에서 JSON을 다루기 위한 두가지 타입(JSON, JSONB)을 비교해 보고, 인덱싱을 적용하는 방법과 그 성능에 대해 알아보았었습니다. 이번에는 기존의 RBDMS의 관계 모델(Relation Model)과 비교하여 어떤 차이가 있는지 알아보려 합니다. 

### PostgreSQL과 Parallel Query

PostgerSQL은 9.6 버전부터 Parallel Query를 지원하고 있습니다. Oracle은 지원하지만, MySQL은 지원하지 않는 기능이죠. Parallel Query는 단일 쿼리를 여러 부작업으로 나누어 병행 수행하는 개념을 말합니다. 쿼리를 여러 조각으로 쪼개서 각각의 쿼리를 별도의 worker에서 병렬(Parallel)로 수행하고, 그 결과값을 합쳐서 결과를 뽑는 것으로 로그 분석이나 배치 작업과 같이 주로 대용량 자료(High Volume 혹은 Large Data)를 다룰 때 유리합니다. 이번 포스팅에서 다룰 데이터는 지난 번 보다 훨씬 많기 때문에, Parallel Query를 활용해서 진행해 보고자 합니다. 하지만 PostgreSQL에서 Parallel Query를 사용하기 위해서는 woker와 shared memory에 대한 설정을 해 줘야만하기에, 지난번과 다르게 별도의 설정을 진행 했습니다. PostgreSQL의 Parallel Query에 대한 자세한 정보는 다음 링크에서 확인할 수 있습니다. https://www.postgresql.org/docs/current/static/parallel-query.html 

# 

# 테스트 준비

### 실행 환경

PostgreSQL
9.6.2 from <https://hub.docker.com/_/postgres/>

Host
Ubuntu 16.04.2 LTS

Intel(R) Core(TM) i7-3930K CPU @ 3.20GHz

DDR3 48GB RAM

2TB RAID 0 (4ea of 500GB Samsung SSD)

본 포스트는 모두 위와 같은 사양에서 진행되었습니다. PostgreSQL은 Dockerize 된 공식 이미지에서 최신 버전인 9.6.2를 받아서 사용했으며, 다음 과 같은 옵션들이 수정되었습니다. 

max_worker_processes
12

max_parallel_workers_per_gather
12

work_mem
128MB

shared_buffers
24GB

### 

### 데이터

테스트를 위한 데이터는 버즈니에서 운영중인 홈쇼핑 모아의 API log 로부터 가져 왔습니다. 총 105,151,017 건의 데이터가 사용 되었으며 형식은 다음과 같습니다.
    
    
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

성능 비교를 위해 JSONB 테이블과, 관계  모델 테이블이 각각 생성되었습니다. 
    
    
    # JSONB 테이블
    CREATE TABLE log_json (dict JSONB);
    
    # 관계 모델 테이블
    CREATE TABLE log (
     id BIGSERIAL PRIMARY KEY,
     ip TEXT,
     url TEXT,
     agent TEXT,
     date TIMESTAMP WITH TIME ZONE,
     size INT,
     time FLOAT
    );
    
    CREATE TABLE log_param (
     id BIGSERIAL PRIMARY KEY,
     log_id BIGINT,
     name TEXT,
     value TEXT
    );

또한 다음과 같은 인덱싱을 사전에 수행해 두었습니다. 인덱스가 없이 수행된 테스트의 경우 각 테스트케이스에 별도로 표시합니다. 
    
    
    # 관계 테이블 인덱스
    CREATE INDEX ON log(url);
    CREATE INDEX ON log(date);
    CREATE INDEX ON log_param(log_id);
    CREATE INDEX ON log_param(name);
    CREATE INDEX ON log_param(value);
    
    # JSONB 테이블 인덱스
    CREATE INDEX ON log_json USING gin(dict);
    CREATE INDEX ON log_json((dict ->> 'date'));
    CREATE INDEX ON log_json((dict ->> 'url'));

 

# 테스트

### Insert

인서트의 경우 인서트하는 테이블의 수가 다르고 Primary Key로 인한 인덱싱으로 인해 정확한 속도 비교가 힘든 점이 있어 생략되었습니다. 

**Table**
**log_json**
**log**
**log_param**

**Rows**

105,151,017

105,151,017

527,457,298

**용량**

101,149 MB

22,843 MB

38,436 MB

![스크린샷 2017-03-24 오후 3.38.26.png](https://boilerbuzzni.files.wordpress.com/2017/03/e18489e185b3e1848fe185b3e18485e185b5e186abe18489e185a3e186ba-2017-03-24-e1848be185a9e18492e185ae-3-38-26.png)JSONB의 경우 약 100GB, 관계 모델의 경우 자식 테이블까지 합하여 약 60GB로 훨씬 작은 테이블 용량이 보여집니다. 이는 JSONB과 관계 모델의 컬럼 최적화 및 데이터 타입에서 오는 차이로 보입니다.  

### Simple Select

단일 테이블에 대한 단순 Select의 경우 다음과 같은 결과가 나왔습니다. 

**항목**
**JSONB**