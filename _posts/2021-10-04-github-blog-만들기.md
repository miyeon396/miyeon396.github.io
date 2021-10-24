---
layout: "single"
title: "GitHub Blog 만들기"
---

참고 youtube : https://www.youtube.com/watch?v=ACzFIAOsfpM
위 유투부 참고하여, 블로그 만든 내용 담에도 써먹기 위해 정리

1. 테마 따기 -> 우측 상단에 포크
첨에 인기가 적은거 썼다가 잘안됨.
스타가 많은데에는 다 이유가 있다. 미니멀로 다시 작업 진행..

2. setting의 url을 바꿔주자
{{id}}.github.io로 rename을 해주자

3. edit 후 &#95;config.yml 바꿔줌
엥간한 설정은 여기서 이제 시작이니 url 만 바꿔주자

4. 블로그에 접속을 해보자
https://miyeon396.githubio.io

5. post를 추가해보자
add 를 누르고 첫 게시글이니 &#95;post 경로 추가 후 파일 이름 지정
&#95;post/2021-10-04.md

  - 규칙 참고
  1. jekyllrb.com/docs/posts/
  2. https://teddylee777.github.io/jekyll/Jekyll-%EC%82%AC%EC%9A%A9%EC%9D%84-%EC%9C%84%ED%95%9C-markdown-%EB%AC%B8%EB%B2%95

6. 장점
소스코드를 블로그로 발행함 
download ad markdown 연도-월-일자-제목.md(제목이 블로그 글 제목이 된다함 한글 지양)
한 곳에 잘 정리하면 블로그 발행이 쉬움. 주피터 노트북을 쓰시던데 노션도 가능할 듯.
파일 생성 시 타이틀이 아닌 타이틀을 바꾸고 싶다면
&#45;&#45;&#45;
layout: "single"
title: "타이틀입니다"
&#45;&#45;&#45;

7. 기타 참고 사항
이미지 관리의 경우 이미지 폴더를 하나 만들어서 블로그에서 참조하도록하면 좋습니다.
대부분의 수정사항은 &#95;config.yml에서 이루어짐.
마크다운을 잘 모른다면 typora 라는 에디터가 있음. 
