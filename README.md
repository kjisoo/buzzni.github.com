# Buzzni 개발자 블로그

## 사전준비

블로그에 글을 쓰기 위해서는 [Jekyll] 을 설치하셔야 합니다.

    $ gem install jekyll

현재 프로젝트를 클론해서 해당 폴더로 이동합니다.

    $ git clone https://github.com/buzzni/buzzni.github.com.git
    $ cd buzzni.github.com


## 쓰기
* _post 디렉토리 아래에 "yyyy-mm-dd-제목.md" 형태로 파일을 작성합니다.
* 제목은 slugify된 영어로 작성해주세요.

    > ex) 2015-03-02-my-first-post.md

* [Markdown] 문법으로 포스트를 작성합니다.


## 글 미리보기

    $ jekyll serve --watch
    
위 명령으로 서버를 키시고 http://localhost:4000 에 들어가서
자신의 글이 제대로 작성되었는지 확인합니다.

    $ open http://localhost:4000


## 출판하기
해당 내용을 커밋하고 메인 저장소에 푸시합니다.

    $ git commit -am "커밋 메시지를 적어봅니다."
    $ git push


## 글 헤더 보일러플레이트

    ---
    layout: post # _layouts 폴더의 어떤 템플릿을 쓸지 정의, 이 경우 _layouts/post.html
    title: 왜 지금 플랫디자인인가? # 제목
    excerpt: 왜 플랫디자인과 미니멀리즘의 시대가 시작되었는지에 대한 생각을 정리해봤습니다. # 부제목
    frame: http://nextecommerce.com.br/wp-content/uploads/2013/07/icones.jpg # 포스트 상단에 표시될 이미지
    author: vincent # 작성자의 사내 영어이름
    email: vincent@buzzni.com # 작성자의 사내 이메일 주소
    tags: design, flat, flat design, minimalism, trend, ui # 태그 목록
    publish: false # 드래프트중인지 확인
    ---

## 도움!!
1. 팀원정보는 _data/members.yml 에서 수정하시면 됩니다. 이메일 주소가 바뀌면 ```images```폴더의 이미지파일도 ```이메일주소.png``` 형태로 수정해주셔야 합니다.


  [Jekyll]: http://jekyllrb.com
  [Markdown]: http://nolboo.github.io/blog/2014/03/25/github-flavored-markdown/
