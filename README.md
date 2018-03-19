# 버즈니 기술 블로그
버즈니의 새로운 **기술 블로그** 입니다.
**개발자**뿐만 아니라 **디자이너**,**기획자**분들까지도 모두 자유롭게 버즈니에서
일하는 방식등의 이야기들을 쓰실 수 있습니다.

## 개발용 서버 띄우기
내가 쓴 글을 확인하고 싶거나 템플릿등의 화면 수정을 원할땐 
[jekyll](https://jekyllrb.com/)을 설치한 후 **jekyll serve**를 띄어서 확인할 수 있습니다.

### jekyll 설치 및 개발 환경 세팅

```bash
$ cd buzzni.github.com
$ bundle install
```

### 개발 서버 띄우기

```bash
$ bundle exec jekyll serve
```
입력 후 브라우저에서 **127.0.0.1:4000** 으로 들어가면 확인 할수 있습니다.


## 글쓰기
프로젝트의 **_post** 폴더에 **"yyyy-mm-dd-제목"** 형태로 폴더를 만들고
그 안에 마찬가지로 **"yyyy-mm-dd-제목.md"** 형태로 마크다운 글을 작성합니다.

> **제목은 한글이 아닌 영어로 작성해주세요**

### 글에 이미지 넣기
다른 사이트의 이미지링크를 넣거나 업로드해서 넣는 경우는 상관없지만
내 이미지를 넣고싶은경우 프로젝트의 **assets** 폴더의 **img** 폴더에 이미지를 넣고
해당 글 이미지를 **{{ site.url }}/assets/img/이미지.png** 와같은 형태로 링크를
지정해주시면 이미지를 넣을 수 있습니다.

### 글 헤더 보일러플레이트
```
---
layout: post
title: 파이써니스타를 위한 슬랙 봇 작성기
tags: [python, slack, bot, slackbot, chatops]
author: 이동균
image: assets/img/character/vincent.png 
type: ENGINEERING
---
```
**title**은 **글의 제목**, **tags**는 **글의 태그**들,  
**author**은 **저자명**, **image**는 **저자 이미지**, 
**type**은 해당글의 **카테고리**를 의미합니다.


## 글 올리기
해당 내용을 커밋하고 메인 저장소에 푸시하면 됩니다.
```bash
$ git commit -m "2018년 1월 15일 블로그 포스팅"
$ git push origin master
```

## 기타
이 블로그는 **jekyll theme** 중 [type-on-strap](https://github.com/sylhare/Type-on-Strap)라는 theme를 fork하여 만들어 졌고
어떤 형태로 고쳐서 사용하셔도 무방합니다.
감사합니다.


