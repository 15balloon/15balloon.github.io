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
\_config.yml 안에 baseurl을 '/'으로 설정하였음에도 불구하고, baseurl이 계속 empty로 나타나는 문제였다.  
그것도 로컬에서 확인할 때는 문제가 없다가 Git에 올려서 실제 페이지에 적용하면 문제가 발생했다.  
baseurl을 '//'이나 '\/'로 바꾸었을 때는 정상적으로 작동하는걸 보니 코드의 문제는 아닌듯 했다.  
실제 페이지는 Github Action을 통해 main Branch에 올라온 코드를 토대로 정적 페이지를 생성하여 gh-pages Branch에 올리고 있었다.  
이 과정을 통해 Github 블로그 페이지를 볼 수 있게 된다.  
이를 보아 Github Action에서 페이지 Build와 Deploy를 하는 과정에서 '/'처럼 하나의 슬래시만 가지고 있는 baseurl을 빈 칸으로 만들고 있음을 추측할 수 있었다.  
관련 내용을 찾아보려 Jekyll 공식 홈페이지와 Github Issues를 살펴보았으나 유의미한 내용은 찾지 못했다.  
Stackoverflow에서 한 답변에 'baseurl은 비거나 '/' 슬래시로 끝나선 안 된다'라고 적힌 내용을 찾은게 전부이다.

##### Reference:

[https://stackoverflow.com/questions/67567907/jekyll-site-baseurl-doesnt-link-to-homepage-when-published-via-github-page-but](https://stackoverflow.com/questions/67567907/jekyll-site-baseurl-doesnt-link-to-homepage-when-published-via-github-page-but)
