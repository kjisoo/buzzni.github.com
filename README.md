# Buzzni 개발자 블로그

## 사전준비

블로그에 글을 쓰기 위해서는 [Jekyll] 을 설치하셔야 합니다.

      $ gem install jekyll
      

## 쓰기
1. _post 디렉토리 아래에 "yyyy-mm-dd-제목.md" 형태로 파일을 작성합니다.
2. [Markdown] 문법으로 포스트를 작성합니다.

## 글 미리보기

      $ jekyll serve
      

위 명령으로 서버를 키시고 http://localhost:4000 에 들어가서
자신의 글이 제대로 작성되었는지 확인합니다.

## 출판하기
해당 내용을 커밋하고 메인 저장소에 푸시합니다.

      $ git commit -am "커밋 메시지를 적어봅니다."
      $ git push
      

## Writing Tips
1. _post 폴더 내의 파일에서 YAML Front Matter block에 <code>author</code>와 <code>author-email</code>을 넣을수 있습니다. <code>author</code>는 작성한 사람, 그리고 <code>author-email</code>은 작성자 이메일, 이렇게 추가 정보를 입력할 수 있고 실제 포스트에는 mail링크가 걸리게 됩니다.
2. _post 폴더 내의 파일에서 YAML Front Matter block에 <code>publish</code>란에 <code>false</code>를 입력하면 게시물을 볼 수 없습니다. 포스팅 리스트에는 뜨지만 글 내용은 Coming Soon이 뜹니다. draft작업이 다 끝나면 <code>publish</code>를 <code>true</code>로 하거나 혹은 그냥 <code>publish</code> 자체를 지워주시면 됩니다.


  [Jekyll]: http://jekyllrb.com
  [Markdown]: http://daringfireball.net/projects/markdown/
