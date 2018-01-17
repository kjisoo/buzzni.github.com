---
layout: post
title: 누구나 읽는 딥러닝, Convolutional Neural Network
tags: [deeplearning, cnn, machinelearning]
author: 유민정
image: assets/img/character/jason.png 
type: DATALABS
---

저는 머신 러닝을 시작한 지 얼마 안 된 개발자입니다. 또한 이론을 깊게 공부하고 연구하는 것보다 빠르게 익히고 적용하는 것을 선호합니다. 이러한 제가 머신 러닝을 어떻게 입문했는가에 대해 써볼까 합니다. 또한 제목에도 쓰여있듯이 글은 딥러닝과 머신 러닝에 대해 자세히 서술하지 않았습니다. 전공자든 비전공자든 누구나 읽을 수 있고 기술의 느낌을 가져가는 것에 초점을 두었습니다. 처음에 머신 러닝을 공부하기 위해 Andrew Ng의 Machine Learning을 봤습니다. 그런데 저는 사실 다 안 봤습니다. 병행해서 공부하던 Convolutional Neural Network이 더 재밌어서. 

## Convolutional Neural Network(CNN)

머신 러닝도 잘 모르는데 뜬금없이 Convolutional Neural Network는 또 뭘까요. 간단히 말하자면 Convolution이라는 연산을 하는 Neural Network입니다. 또한 결론부터 말하자면 Vision Problem에 굉장히 강력한 모델입니다. 어떠한 특성을 가지길래 Vision Problem에 강력할까요? Convolution 이란 연산은 차후에 설명하고 우선은 Neural Network입니다. 

### Neural Network

일반적인 대다수의 머신 러닝 모델은 y = f(x)의 함수로 나타낼 수 있을 것입니다. x라는 input을 넣었을 때 y라는 output을 반환하는 것입니다. 이때 x와 y는 벡터, 텍스트, 이미지 등 어떠한 것도 될 수 있습니다. 영문-한글 번역기는 영문이란 텍스트를 input으로 넣고 한글이 output으로, 그 유명한 알파고는 대국 상황이 input으로 다음 착점이 output이 됩니다. 그렇다면 결국 모델을 만든다, 학습시킨다는 것은 x에 대한 함수 f(x)를 적절하게 결정한다는 것입니다. 간단한 예제로 살펴보겠습니다.  아래 내용은 Oxford의 [Deep Learning for Natural Language Processing: 2016-2017](https://github.com/oxford-cs-deepnlp-2017/lectures) 강의 교안의 내용을 발췌했습니다. [caption id="attachment_470" align="alignnone" width="2128"]![스크린샷 2017-02-21 오후 10.01.15.png](https://boilerbuzzni.files.wordpress.com/2017/02/e18489e185b3e1848fe185b3e18485e185b5e186abe18489e185a3e186ba-2017-02-21-e1848be185a9e18492e185ae-10-01-15.png) 이미지 출처 : [Oxford deepnlp 2017 Lecture 1b](https://github.com/oxford-cs-deepnlp-2017/lectures)[/caption] 마을에 괴물이 나타났습니다. 이 괴물은 사과를 1개 주면 바나나를 0개 줍니다. 사과를 5개 주면 바나나를 16개 주고, 사과를 6개 주면 바나나를 20개 줍니다. 주는 사과의 개수와 받는 바나나 개수의 상관관계가 있을까요? 머신 러닝을 이용하여 이 문제를 해결해봅니다. 이 문제의 input은 사과의 개수 x입니다. output은 바나나의 개수 y가 됩니다. 또한 우리는 임의의 수 x가 들어왔을 때 y를 반환하는 머신 러닝 모델을 학습시키려고 합니다. 머신 러닝 모델을 학습시킨다는 것은 함수를 결정하는 일입니다.  우리가 아는 가장 단순한 함수인 일차함수로 모델을 만들어 봅시다. y = ax + b a와 b는 변수입니다. 이제 또한 a와 b만 결정하면 함수가 완성됩니다. a와 b를 각각 1로 가정해봅시다. y = 1x + 1 이 모델은 사용될 수 있을까요? 다행히도 저희는 무려 3개의 데이터 셋이 있습니다. x에 각각 과거에 주었던 사과의 개수인 1, 5, 6을 넣어봅니다. y = 1 * 1 + 1 = 2 ≠ 0 y = 1 * 5 + 1 = 6 ≠ 16 y + 1 * 6 + 1 = 7 ≠ 20 ![%e1%84%89%e1%85%b3%e1%84%8f%e1%85%b3%e1%84%85%e1%85%b5%e1%86%ab%e1%84%89%e1%85%a3%e1%86%ba-2017-02-21-%e1%84%8b%e1%85%a9%e1%84%92%e1%85%ae-9](https://boilerbuzzni.files.wordpress.com/2017/02/e18489e185b3e1848fe185b3e18485e185b5e186abe18489e185a3e186ba-2017-02-21-e1848be185a9e18492e185ae-91.png) 초록선 만큼의 오차가 발생하였고 함수가 제대로 결정되지 않았다는 것을 알 수 있습니다. 저희는 a와 b를 계속 바꿔서 모델이 예측하는 값이 과거에 모인 데이터 셋의 y와 오차가 최대한 적게 발생하는 값을 찾아야 합니다. 다양한 시도를 통해 최적의 a, b값(위와 같은 경우는 4, -4가 됩니다.)을 찾는 과정을 모델을 학습시킨다고 합니다. 이때 a와 b를 업데이트 하는 방법중에 가장 대표적인 방법은 Gradient Descent입니다.  많은 머신 러닝 강좌에서 굉장히 중요하게 다루는 내용이니 꼭 살펴보시기 바랍니다. ![스크린샷 2017-02-21 오후 9d3.png](https://boilerbuzzni.files.wordpress.com/2017/02/e18489e185b3e1848fe185b3e18485e185b5e186abe18489e185a3e186ba-2017-02-21-e1848be185a9e18492e185ae-9d3.png) y = 4x -4 사실 대다수의 머신 러닝 모델이 위와 같은 일차함수로 나타낼 수 없는 경우가 많습니다. 세상의 모든 input과 output은 비례관계에 있지 않기 때문입니다. 또한 input이 이미지이고 output이 이미지가 개인지 고양이인지 맞추는 경우 이미지의 각 픽셀이 x가 되어 무수히 많은 x가 필요할 수도 있고 sigmoid와 같은 비선형 함수가 필요한 경우 또한 많습니다. 만약에 input이 x1, x2이고 이 값에 따라 output인 y가 빨간색, 파란색으로 결정되는 데이터 셋이라면 어떨까요? ![이름 없음.jpg](https://boilerbuzzni.files.wordpress.com/2017/02/e1848be185b5e18485e185b3e186b7-e1848be185a5e186b9e1848be185b3e186b7.jpg) 여러 x가 필요하며 이에 따른 y 또한 선형적으로 결정되지 않습니다. 나이, 몸무게와 비만의 상관관계를 예측하는 모델 등 실생활에서 쉽게 볼 수 있는 예시입니다. 그래서 이러한 문제를 해결하기 위해 y = g(f(g(f(x)) + g(f(x)))) 와 같이 여러 함수가 더해지고 곱해지는 꼴의 모델이 필요할 수도 있습니다. 이러한 함수는 매우 단순한 예시이고 실제로는 훨씬 더 많은 함수가 필요하고 함수끼리 더 복잡한 연산이 필요할 때가 많습니다. 아래의 그림은 처음 input이 각각 4개의 함수를 거치고 다시 그 output이 4개의 함수를 거친 후 마지막으로 하나의 함수를 통해 하나의 output이 결정되는 모습입니다. 가장 시작의 input은 input layer, 가장 마지막의 output은 output layer가 되며 그 외의 부분은 hidden layer라고 부릅니다. [caption id="attachment_453" align="alignnone" width="442"]![스크린샷 2017-02-21 오후 9.51.31.png](https://boilerbuzzni.files.wordpress.com/2017/02/e18489e185b3e1848fe185b3e18485e185b5e186abe18489e185a3e186ba-2017-02-21-e1848be185a9e18492e185ae-9-51-31.png?w=884)   
이미지 출처 : [Stanford CS231n](http://cs231n.github.io/neural-networks-1/)[/caption] 뇌의 뉴런들이 연결되어 있는 모습과 매우 닮지 않았나요? 위와 같은 형태가 Neural Netwrok이고 Layer가 많아질수록 Deep 하다고 하며 모델이 다양한 데이터에 대응할 수 있게 됩니다. 

## Convolution

다음은 Convolution 연산입니다. Convolution은 그렇게 어렵지 않습니다. 이미지 프로세싱에서 사용하는, 카메라 어플에서도 흔히 이용되는 이미지 필터입니다. [caption id="attachment_489" align="alignnone" width="268"