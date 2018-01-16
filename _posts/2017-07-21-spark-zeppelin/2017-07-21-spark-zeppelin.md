---
layout: post
title: Spark와 Zeppelin 시작하기
tags: [spark, zeppelin, visualization, data]
author: 김영덕
image: assets/img/character/roland.png 
type: DATAINFRA
---

#### **시작하며**

spark, zeppelin 시작하기 위한 quick start의 개념을 정리하고자 글을 작성 했습니다.  

#### **스파크 소개**

**Apache Spark™** is a fast and general engine for large-scale data processing. 스파크는 예전 부터 주목 받기 시작하였고 현재는 해외는 물론 국내에서도 많이 사용하는 플랫폼으로 자리 잡기 시작했습니다. 스파크에 가장 큰 특징은 이름에서도 느껴지는 분위기로도 매운 빠른 "속도" 입니다. 기존 Hadoop(하둡)대비 속도가 최대 100배 이상 빠르다고 합니다. ![logistic-regression](https://boilerbuzzni.files.wordpress.com/2017/07/logistic-regression.png) Logistic regression in Hadoop and Spark 이렇게 매운 빠른 속도로 데이터치리가 가능한 이유는 메모리를 적극적으로 활용하기 때문이라고 하는데 하둡 같은 경우는 Map & Reduce(맵리듀스) 처리한 결과물을 HDFS에 바로 저장 해야 하기 때문에 디스크 사용 빈도가 높을 수 밖에 없고 비효율적으로 다음 단계 맵리듀스로 데이터 전송 할 수 밖에 없기 때문에 느린속도로 처리됩니다. 스파크는 RDD라는 메모리 구조에서 효율적으로 빠르게 전달하여 처리속도 향상을 가져오게 됩니다. 또한 스파크의 속도가 가장 큰 장점이지만 부가적으로 다양한 기능들이 존재 합니다. 

  * Hadoop & Hive의 SQL 구문 처리
  * Hadoop의 Map & Reduce
  * Storm의 realtime streaming data
위와 같은 장점 말고도 MLib(machine learning), GraphX등과 다양한 동작 환경들을 제공 하고 있습니다. 만약 위와 같은 기능들을 하둡에서  처리 한다고 생각한다면 수 많은 설정과 복합적인 프로그램 설치등의 골치 아픈 일들이 하나 두개가 아니지만 스파크는 위의 있는 기능들을 간단한 설정 만으로 전부 처리가 가능하다는 점입니다. 게다가 기존 방식보다 속도가 매우 빠른 처리가 가능하니 스파크를 선택 안 할 이유가 없어 보입니다.  

#### 왜 스파크를 선택 하였는가?

회사에서 로그 분석을 위해서 ElasticSearch(ES)를 사용하며 전체 서비스의 로그를 분석하고 있습니다. 하지만 ES는 메모리를 매우 많이 필요하며 단편적인 데이터 검색에서는 막강하나 분석을 위한 쿼리는 지원하지 않거나 비 효율적으로 사용되는 경우가 있어서 여러 직군에서 대쉬보드를 쉽게 조작하기란 쉽지 않습니다. 따라서 새로운 장기 데이터 분석시스템이 필요 하고 메모리에서 비교적 너프한 솔루션이 필요하여 스파크를 선정되어 적용중에 있습니다. 선정된 이유: 

  * 로그 파일을 아마존 S3에 저장하는데 스파크(EC2)와 뛰어난 호환성 보임
  * parquet으로 저장할 경우 처리 속도와 저장 공간 효율성 증대
  * EC2로 인한 분산 클러스터 구축이 용이함
  * 장기 데이터 처리 할 경우에만 EC2 클러스터 개수를 증가 시켜 속도와 비용 관리가 용이함
 

#### 스파크 시작 해보기

  1. 사전 준비 스파크를 사용하기 위해서는 스파크와 JDK가 설치 되어 있어야 합니다. 아래의 사이트에서 다운로드를 합니다. https://spark.apache.org/downloads.html http://www.oracle.com/technetwork/java/javase/downloads/index.html https://zeppelin.apache.org/download.html
  2. bash.rc에 스파크 패스값을 넣습니다. export SPARK_HOME=~/spark-2.0.0-bin-hadoop2.7
  3. SPARK_HOME/conf/spark-env.sh에 아래의 값을 넣습니다. export SPARK_MASTER_HOST=0.0.0.0 export SPARK_MASTER_PORT=7077 export SPARK_MASTER_WEBUI_PORT=8088
  4. 간단한 예제 실행 먼저 zepplin에 연결 하기 전에 스파크가 제대로 설정이 되었는지 확인 을 해봐야 하기 때문에 아래에 있는 간단한 워드 카운트 예제를 실행을 해봅니다. ./bin/pyspark 

* * *

> >> textFile = spark.read.text("README.md") >> textFile.count()

126 

* * *

그리고 localhost:8088에 접속하게 되면 스파크 메인 화면을 볼 수 있습니다. ![스크린샷 2017-07-21 오후 5.07.24](https://boilerbuzzni.files.wordpress.com/2017/07/ec8aa4ed81aceba6b0ec83b7-2017-07-21-ec98a4ed9b84-5-07-24.png)
  5. zeppelin을 실행하기 위해서는 기본 자바 라이브러리가 필요합니다. hadoop-aws-2.7.3.jarjets3t-0.9.4.jaraws-java-sdk-1.7.4.jar3개의 jar파일을 아래의 있는 spark, zeppelin에 복사합니다.ZEPPELIN_HOME/interpreter/spark/dep/SPARK_HOME/jars/
  6. zeppelin를 실행합니다. 실행에 관한 사용법 : https://zeppelin.apache.org/docs/0.7.2/install/install.htmlzeppelin 웹관리자 : http://localhost:8080 ![스크린샷 2017-07-21 오후 5.56.47.png](https://boilerbuzzni.files.wordpress.com/2017/07/ec8aa4ed81aceba6b0ec83b7-2017-07-21-ec98a4ed9b84-5-56-47.png)
  7. zeppelin의 spark 인터프리터 설정을 수정합니다. URL : http://localhost:8080/#/interpreter 

master
spark://localhost:7077 

master
Zeppelin 

spark.cores.max
4 

spark.executor.memory
1g 

spark.hadoop.fs.s3a.access.key

spark.hadoop.fs.s3a.secret.key

spark.hadoop.fs.s3a.impl
org.apache.hadoop.fs.s3a.S3AFileSystem 

#### 

#### 마무리

모든 설정이 끝났으면 간단한 쿼리문을 작성 할 수 있다면 zeppelin를 사용한 대쉬보드를 만들 수 가 있습니다. ![스크린샷 2017-07-24 오후 10.03.46](https://boilerbuzzni.files.wordpress.com/2017/07/e18489e185b3e1848fe185b3e18485e185b5e186abe18489e185a3e186ba-2017-07-24-e1848be185a9e18492e185ae-10-03-46.png) ![스크린샷 2017-07-24 오후 10.03.29.png](https://boilerbuzzni.files.wordpress.com/2017/07/e18489e185b3e1848fe185b3e18485e185b5e186abe18489e185a3e186ba-2017-07-24-e1848be185a9e18492e185ae-10-03-29.png) 위와 같이 S3에 저장된 parquet 파일로 부터 테이블을 생성하게 되면 SQL 쿼리문으로 바로 대쉬 보드를 제작 할 수 있게 됩니다. 그리고 대쉬보드는 정적인 데이터 처리 뿐만 아니라 결과물에 대한 인터렉티브한 데이터 처리 기능을 제공 하고 있으니 여러 직군에서 같이 사용하기에 알맞는 솔루션이 아닐 수 없습니다.