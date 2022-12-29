---
layout: post
logo: "assets/images/15balloon.png"
navigation: True
cover: "assets/images/cover1.jpg"
title: "Giuhub Pages 404 error"
author: "15balloon"
date: 2022-12-29 23:20:57 +0900
tags: error
categories: 15balloon
subclass: "post tag-test tag-content"
---

#### Jekyll 테마 적용 중에 발생한 오류 수정

모든 파일을 Git에 제대로 올렸음에도 페이지 메뉴에서 태그별 게시물 보기를 눌렀을 때 404 페이지가 뜨는 문제가 발생했다.  
<img src="{{site.baseurl}}/assets/images/2022-12-29-Github-Page-404-error_1.jpg" width="100%" height="100%" title="problem" alt="Image: 404 Page"/>
뭐가 문제인지 알 수 없어서 적용한 테마의 Github에 들어가서 거기에 올라온 코드와 내가 올린 코드를 비교했다.  
그중 link 태그의 href가 아래와 같이 나오고 있다는 사실을 알게 되어 이 부분을 수정하여 다시 올렸다.

```
<link rel="canonical" href="//asset/...">
```

수정하여 올렸음에도 불구하고 404 페이지가 뜨는 건 똑같았다.  
검색을 통해 link 태그는 주로 .css 같은 스타일시트 파일을 연결하거나 웹 폰트를 적용할 때 사용하는 태그라는 걸 알게 되었다.  
위와 같이 URL이 작성되어도 페이지를 불러오는 데에는 문제는 없다는 것이다.

다시 적용한 테마의 Github에 들어가서 코드를 비교했지만, 다른 점은 하나도 없었다.  
그리고 그제야 구글에 'Github Page 404'라고 검색해봤다.  
메인으로 나온 글을 선택해서 답변을 보았는데, 추천 수가 높은 답변은 내 상황과는 다른 답변이었다.  
답변이 무려 42개나 달린 글이어서 아래로 내리면서 내 상황과 맞는 답변이 있는지 보았다.  
보통 Jekyll을 사용하지 않은 경우 특정 파일을 올리라든지, 다른 branch에 commit을 하라든지 하는 답변들이었다.  
그러다 추천 수가 4개인 답변이 눈에 띄었다.  
Jöcker라는 사용자가 달아 놓은 답변인데 보는 순간 알았다.  
이게 내 문제를 해결해줄 거라는 것을.  
해결책은 간단했다.  
Github Page를 올린 repository에 들어가 Settings를 누르고 Pages 탭을 선택해서 Github Pages를 보여줄 branch를 선택하는 것이었다.  
<img src="{{site.baseurl}}/assets/images/2022-12-29-Github-Page-404-error_2.jpg" width="100%" height="100%" title="solution" alt="Image: Change repository Settings Branch"/>
설정을 바꾸자마자 문제는 해결이 되었다.

##### Reference:

[https://stackoverflow.com/questions/11577147/how-to-fix-http-404-on-github-pages](https://stackoverflow.com/questions/11577147/how-to-fix-http-404-on-github-pages)
