---
layout: post
title: Machine Learning 으로 오늘의 UV 맞추기
excerpt: Regression 모델을 이용하여 바탕으로 오늘의 UV를 예측 해보기
frame: http://i2.wp.com/elenacuoco.altervista.org/blog/wp-content/uploads/2014/09/scikit.png
author: Justin
email: justin@buzzni.com
tags: machine learning, regression, server, mining
publish: true
---

# 시작하며

안녕하세요? 버즈니에서 검색, 데이터 마이닝 부분을 맡고 있는 저스틴 (Justin) 입니다. 

저는 Search Engine 과 Machine Learning 분야를 연구하고 이를 활용한 응용 서비스를 만드는 것을 굉장히 좋아합니다.

2007년도에 버즈니를 창업하고 나서 지금까지 꾸준히 관련 기술을 연구하고 실제 서비스에 많이 적용해 왔습니다. 

이 분야를 정말 좋아한 나머지 업무가 끝나고 나서도 취미로 새로운 기술들을 익히거나 새로운 응용들을 만들어 보기도 합니다.

오늘 소개해드릴 내용은 그런 취미 활동중 하나로,

업무를 하다가 반복적으로 생각 했던 일을 Machine Learning 을 실제로 사용하여 해결한 사례입니다. 

이 주제는 현재 버즈니 개발팀에서 진행하는 Data Mining Study 의 첫번째 실습 예제입니다.


# 개요

저희는 사무실 내에서도 큰 TV 에 현재 접속자 수 등 중요 통계를 항상 띄워 놓을 정도로, 통계를 아주 중요하게 생각하고 있습니다. 

그래서 매 시간마다 현재까지 접속자 수(UV)를 기준으로 오늘 UV 는 대략 얼마 정도가 될 것 같다고 머리속으로 예측을 해왔습니다. 

심지어 단순히 2만 곱하면 거의 그날의 UV 가 되는 시간대까지 찾아내기도 했습니다.

그렇게 매번 수치를 보고 오늘의 트래픽을 머리속으로 계산을 하다가, 문득 이런 생각이 들었습니다.

사람이 과거 데이터 기록을 이용하여 오늘의 트래픽을 예측하듯이 Machine Learning 을 활용하면

오늘의 UV 를 더 정확하게 예측할 수 있지 않을까 하는 생각이 들었습니다.

이전에 공부를 했던 데이터 마이닝 모델들을 쭉 살펴봤더니 그동안 공부만 하고 실제 업무에는 활용을 잘 안했던

Regression 모델을 이용하면 딱이겠다는 생각이 들었습니다.


# 준비물

잘 아시는 분들도 있겠지만, 처음 접하는 분들을 위해서 준비물에 대해서 좀 더 설명을 드리겠습니다. 

1. [Scikit-Learn] : Python 에서 Machine Learning 을 하는 대표적인 Library 입니다. 
2. [IPython] : IPython Notebook 이란 것을 주로 사용합니다. 해외에서 하는 많은 컨퍼런스를 보면 IPython 으로 튜토리얼을 진행하는 것을 상당히 많이 볼 수 있습니다. pycon에 보면 IPython 을 안 쓰는 튜토리얼을 찾아 보기 힘들정도로 많이 사용합니다. web browser 에서 interactive 하게 다양한 파라미터로 실험을 하고 그래프를 그리기가 아주 편하게 되어 있어서 science 용으로 딱 좋습니다.


# 데이터

필요한 데이터는 최근 한달간 `각 시간대별 UV` 및 `각 일자별 UV` 입니다.

버즈니에서는 로그 데이터를 EFK(ElasticSearch, Fluentd, Kibana)를 사용하여 분석하고 관리하고 있습니다.

이 데이터를 추출할때 Kibana 를 통해서 ElasticSearch 에 어떤 Query 를 만들어야 시간당 UV, 일간 UV 를 가져오는지 알 수 있습니다. 
![img](https://dl-web.dropbox.com/get/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202015-03-04%2021.32.47.png?_subject_uid=22018269&w=AABPmuA57gtVoowTWNOYF87nInusxEGnm94k_oDuX7dofQ)


# IPython 노트북 
 
## 목표

 * 현재 시간까지의 시간대별 UV 합을 입력하면, 오늘 하루의 UV 예측치를 알려주는 시스템을 만들어봅니다.
 * 각 시간대별로 모델을 만든다. 이때 Linear Regression 모델을 활용합니다.


## 필요한 모듈 import

{% highlight python %}
# coding: utf-8

import json
import requests
import numpy as np
from dateutil import parser
from sklearn.linear_model import LinearRegression
{% endhighlight %}


## 학습데이터 준비

 * X train 데이터 : 최근 한달 동안 각 시간대별 누적 UV , 예를 들어 `3시`라고 하면, 1시부터 3시까지 쌓인 UV 수의 합
 * Y train 데이터 : 최근 한달 동안 각 일자별 하루 UV 수


{% highlight python %}
index_str="_all"
_LOG_ES_URL = "http://localhost:9200/"
{% endhighlight %}

 - 아래는 Log 데이터가 저장된 Elastic Search 에 요청할 Query 입니다.
 - Query를 어떻게 요청해야 할지는 Kibana 를 통해서 알아냈습니다.

{% highlight python %}
es_daily_query = {
  "size": 0,
  "query": {
    "filtered": {
      "query": {
        "query_string": {
          "query": "ps:nginx",
          "analyze_wildcard": True
        }
      },
      "filter": {
        "bool": {
          "must": [
            {
              "range": {
                "@timestamp": {
                  "gte": 1422806671815,
                  "lte": 1425398671815
                }
              }
            }
          ],
          "must_not": []
        }
      }
    }
  },
  "aggs": {
    "2": {
      "date_histogram": {
        "field": "@timestamp",
        "interval": "1d",
        "pre_zone": "+09:00",
        "pre_zone_adjust_large_interval": True,
        "min_doc_count": 1,
        "extended_bounds": {
          "min": 1422806671814,
          "max": 1425398671814
        }
      },
      "aggs": {
        "1": {
          "cardinality": {
            "field": "xid"
          }
        }
      }
    }
  }
}

es_daily_query = json.dumps(es_daily_query)
url = _LOG_ES_URL + index_str + '/_search?pretty'
daily_result = requests.get(url, data=es_daily_query).json()
{% endhighlight %}

- 각 일별 UV 데이터를 daily_uv_dict 변수에 저장합니다.

{% highlight python %}
daily_uv_dict = {}
for each in daily_result[u'aggregations']['2'][u'buckets'][:-1]:
    date_str = each['key_as_string'].split("T")[0]
    daily_uv_dict[date_str] = each[u'1'][u'value']
{% endhighlight %}

- 각 시간대별 UV 수를 요청하는 Query 입니다.

{% highlight python %}
es_hour_query ={
  "size": 0,
  "query": {
    "filtered": {
      "query": {
        "query_string": {
          "query": "ps:nginx",
          "analyze_wildcard": True
        }
      },
      "filter": {
        "bool": {
          "must": [
            {
              "range": {
                "@timestamp": {
                  "gte": 1422806539236,
                  "lte": 1425398539236
                }
              }
            }
          ],
          "must_not": []
        }
      }
    }
  },
  "aggs": {
    "2": {
      "date_histogram": {
        "field": "@timestamp",
        "interval": "1h",
        "pre_zone": "+09:00",
        "pre_zone_adjust_large_interval": True,
        "min_doc_count": 1,
        "extended_bounds": {
          "min": 1422806539235,
          "max": 1425398539235
        }
      },
      "aggs": {
        "1": {
          "cardinality": {
            "field": "xid"
          }
        }
      }
    }
  }
}

es_hour_query = json.dumps(es_hour_query)

url = _LOG_ES_URL + index_str + '/_search?pretty'
hour_uv_result = requests.get(url, data=es_hour_query).json()

hour_uv_dict = {}
for hour in range(24):
    hour_uv_dict[hour] = []
y_list = []
hour_model_dict = {}
hour_model_dict2 = {}
TRAIN_DATE = '2015-02-20'
{% endhighlight %}


## 모델 학습

 - 각 시간대별로 모델을 별도로 만듭니다.
 - 1,2 두가지 모델을 만드는데, 1은 현재시간까지의 UV 합 데이터만을 이용해서 학습을 하고
 - 2 모델은 현재 시각의 UV 와, 현재 시간까지의 UV 합 2가지 데이터를 이용해서 학습합니다.
 - 2015-02-20 이전 날짜의 데이터를 Train 데이터로 사용하고, 이 이후의 데이터를 Test 데이터로 사용합니다.

{% highlight python %}
for hour in range(0,24):
    x_list1 = []
    y_list1 = []
    x_list2 = []
    y_list2 = []    
    key_count_sum = {}
    pre_day = 0 
    # 현재 시간대 hour 에 대해서 각 날짜별로 해당 시간까지의 UV 합을 구한다. 
    for each in hour_uv_result[u'aggregations']['2'][u'buckets']:
        parse_date = parser.parse(each[u'key_as_string'])
        day = parse_date.strftime("%Y-%m-%d")   
        hour_uv = each['1']['value']
        if not daily_uv_dict.has_key(day):
            continue                
        if day > TRAIN_DATE:
            continue
        
        if not key_count_sum.has_key(day):
            key_count_sum[day] = 0
                
        key_count_sum[day]+=hour_uv
        
        if parse_date.hour == hour:
            day_ct = daily_uv_dict[day]
            x_list1.append(key_count_sum[day])
            x_list2.append([hour_uv,key_count_sum[day]])
            y_list2.append(day_ct)
            y_list1.append(day_ct)

    X = np.array(x_list1)[:, np.newaxis]
    
    model = LinearRegression(normalize=True)
    model.fit(X,y_list1)

    hour_model_dict[hour] = model

    g_model = LinearRegression(normalize=True)
    g_model.fit(np.array(x_list2),y_list2)
    hour_model_dict2[hour] = g_model
    if hour%4 == 0:
        print "model learn",hour,"hour"
{% endhighlight %}


## 모델 사용

 - 만들어진 모델을 이용하여 실제 그날 그날의 UV 를 예측 해봅니다.
 - 예측한 일 UV 와 실제 일 UV 를 비교해서 어느정도 오차로 예측했는지를 봅니다.
 - TRAIN_DATE 이후의 학습데이터로 사용하지 않은 데이터로 테스트를 해봅니다.
    
{% highlight python %}
diff1_list = []
diff2_list = []
for hour in range(0,24):
    x_list = []
    y_list = []
    x_list2 = []
    y_list2 = []    
    key_count_sum = {}

    for each in hour_uv_result[u'aggregations']['2'][u'buckets']:
        parse_date = parser.parse(each[u'key_as_string'])
        day = parse_date.strftime("%Y-%m-%d")   
        hour_uv = each['1']['value']
        if not daily_uv_dict.has_key(day):
            
            continue        
        if day <= TRAIN_DATE:
            continue            
        
        if not key_count_sum.has_key(day):
            key_count_sum[day] = 0
        
        key_count_sum[day]+=hour_uv
        
        if parse_date.hour == hour:

            day_ct = daily_uv_dict[day]

            
            pred1 = hour_model_dict[hour].predict(key_count_sum[day])[0]
            pred2 = hour_model_dict2[hour].predict([hour_uv,key_count_sum[day]])
            diff1_list.append(abs(int((day_ct-pred1))))
            diff2_list.append(abs(int((day_ct-pred2))))
        
print "diff1:",sum(diff1_list)/len(diff1_list)
print "diff2:",sum(diff2_list)/len(diff2_list)
{% endhighlight %}


# 정리하며
 - 이상으로 Regression 모델을 활용한 간단한 UV 예측 모델을 만들어 보았습니다. 
 - 실제로 똑같이 따라하면 정확히 나오는 데이터 Source 까지 포함된 Ipython Notebook 을 올리고 싶은 마음도 굴뚝 같지만 UV 가 내부 데이터라 외부에 공개할 수 없어서 데이터 소스는 http://localhost:9200 로 해놓은 점은 양해 부탁 드립니다.
 - 위 코드는 참고만 하시고, 개념을 이해한 후에 개별로 적용을 하면서 익혀 나가시면 될것 같습니다.
 - [UV Regression Ipython Notebook](https://www.dropbox.com/s/ubtjfbd39xo6ski/UV_regression.ipynb?dl=0)
 - 그리고 저희 개발팀과 함께하실 서버 개발자 및 안드로이드 개발자를 모십니다. 많이들 지원해주세요~ (recruit@buzzni.com)
 - 그럼 모두들 즐코(즐거운 코딩) 하세요~    


[Scikit-Learn]: http://scikit-learn.org
[IPython]: http://ipython.org
