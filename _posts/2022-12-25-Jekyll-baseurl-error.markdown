---
layout: post
logo: "assets/images/15balloon.png"
navigation: True
cover: "assets/images/cover1.jpg"
title: "Jekyll baseurl error"
author: "15balloon"
date: 2022-12-25 04:20:29 +0900
tags: error
categories: 15balloon
subclass: "post tag-test tag-content"
---

#### Jekyll 테마 적용 중 발생한 오류 수정

Jekyll 테마를 적용하는 중에 baseurl이 적용되지 않는 문제가 발생했다.  
\_config.yml 안에서 baseurl을 '/'으로 작성하였음에도 불구하고, baseurl이 계속 empty로 나타나는 문제였다.  
그것도 로컬에서 확인할 때는 문제가 없다가 Git에 올려서 Github Page를 확인했을 때 문제가 발견되었다.  
baseurl을 '//'이나 '\/'로 바꾸었을 때는 동일한 문자열을 가져와서 작동하는걸 보니 코드의 문제는 아닌듯 했다.  
실제 페이지는 Github Action을 통해 'main' Branch에 올라온 코드를 토대로 정적 페이지를 생성하여 'gh-pages' Branch에 올린다.  
이 과정을 통해 Github Page를 볼 수 있게 된다.  
이를 보아 Github Action에서 페이지 Build와 Deploy를 하는 과정에서 '/'처럼 하나의 슬래시만 가지고 있는 baseurl을 빈 칸으로 만들고 있음을 추측할 수 있었다.  
관련 내용을 찾아보려 Jekyll 공식 홈페이지를 살펴보았으나 유의미한 내용은 찾지 못했다.  
Github의 Issue를 살펴보니 다음과 같은 내용이 적혀있었다.

> See http://ben.balter.com/jekyll-style-guide/config/#baseurl (and the linked post) for more information on s. The baseurl should never be. baseurl: /  
> baseurl을 '/'으로 작성하면 안 된다. (링크된 포스트 내용) '/example'과 같은 하위 경로가 있는 경우에만 baseurl을 작성해야 하고, 하위 경로가 없다면 아무것도 작성하지 않는다.

이에 따라 코드 내에서 baseurl을 사용하는 모든 곳에 '/'를 추가 작성하여 해결하였다.

##### Reference:

[https://github.com/github/pages-gem/issues/350](https://github.com/github/pages-gem/issues/350)
