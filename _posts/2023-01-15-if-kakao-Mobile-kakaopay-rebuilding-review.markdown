---
layout: post
logo: "assets/images/15balloon.png"
navigation: True
cover: "assets/images/cover1.jpg"
title: "if(kakao)dev2022 모바일: '카카오페이 앱 리빌딩 스토리' 세션 후기"
author: "15balloon"
date: 2023-01-15 17:57:13 +0900
tags: android conference
categories: 15balloon
subclass: "post tag-test tag-content"
---

##### 세션 내용 요약

1. 앱 리빌딩 배경

   - 서비스 증가에 따른 코드 품질과 사용성 저하 발생
   - 새로운 사용자 경험뿐 아니라, 생산성 강화를 위해 DI, Module화 작업과 UI 컴포넌트 통합을 진행

2. DI 개선 및 모듈화  
    DI란?  
    Dependency Injection의 준말로, 한국어로는 '의존성 주입'이라고 한다. Car라는 클래스와 Engine이라는 클래스가 있다고 하자. Car에는 Engine이 꼭 필요하므로 두 클래스는 의존 관계에 있다고 말할 수 있다. 이것이 의존성이다. 그리고 Car에 Engine을 삽입해야 한다. 이것이 주입이다. 한마디로 필요한 클래스를 자체적으로 생성하지 않고 받아서 사용하는 것이 DI이다.

   카카오페이 앱에서는 Koin 라이브러리를 사용하여 DI를 구현한 상태였으나, 객체들이 올바른 scope 내에서 생성 되었는지 알 수 없어 관리에 어려움을 느꼈다.  
    타 서비스의 DI 시스템 상황과 컴포넌트 관리 기능을 제공한다는 점을 고려하여 새로운 DI 시스템으로 Hilt를 도입하였다.  
    서비스 내의 객체들을 올바른 주기로 재사용할 수 있게 해주는 컴포넌트와 Scope Annotation을 이용할 수 있는 장점이 있었다.  
    Koin의 Custom Scope와 Hilt의 Activity Scope가 일치하지 않아 고민했으나, Navigation graph Scope를 사용하여 문제를 해결하였다.  
    <br>
   이전 앱에서는 UI를 담당하는 app 모듈과 대부분의 비즈니스 로직을 가진 core 모듈로 구성되어 있었다.  
    더 좋은 생산성을 위해 Feature 기반 모듈화를 진행하였다.  
    크게는 Non-Feature 모듈과 Feature 모듈로 구분하였다.  
    Non-Feature 모듈에는 design-system, core, util로 세분화되어 있으며, core 모듈은 이전과 다르게 통신 혹은 보안을 위한 코드들로 구성했다.  
    Feature 모듈은 money, payment, stock, home으로 세분화 되어 있으며, 이는 독립적인 서비스 기능 모음을 모듈화한 것이다.  
    이 모듈은 또 presentation, domain, data 레이어로 나누었다.  
    presentation 레이어에는 Activity, Fragment, Viewmodel가,  
    domain 레이어에는 Usecase, Repository Interface, Model이,  
    data 레이어에는 Repository 및 DataSource가 포함되었다.  
    Feature 사이에서 Activity들을 효과적으로 호출하기 위해 인터페이스로 구성된 navigator 모듈을 생성하였다.  
    이러한 모듈화를 바탕으로 생산성이 증가하여 모듈의 수는 점차 증가하고 있으며 코드량이 많은 모듈이 생김에 따라 빌드 시간도 늘어나는 결과를 보였다.  
    <br>
   UI 컴포넌트를 통일하여 개발과 소통을 위한 비용을 줄이고 사용자 경험 향상 위해 디자인 시스템이 필요하였다.  
   Foundation, Component, Module, Template 4가지로 정의하여 작업을 수행하였다.  
   디자인 시스템 도입 이전에는 xml을 직접 작성하여 세부 요소 작업을 놓치기 쉬웠으나, 도입 이후에는 CustomView와 Style만으로 UI를 구성할 수 있어 일관된 UI를 제공할 수 있게 되었다.  
   또한 확장성을 고려한 CustomView 개발로 개발 편의성이 증가하였다.  
   그리고 디자인 시스템 샘플 앱을 만들어 여러 케이스를 쉽게 테스트할 수 있게 환경을 구성하였다.  
   WindowCompat 함수 호출을 통해 ViewGroup 영역과 System Bar 영역의 색상이 통일되도록 하여 몰입도 높은 화면을 제공하였다.  
   이때 System Bar 영역이 ViewGroup 영역과 겹치지 않도록 view에 inset 값으로 padding을 적용하였다.  
   System Bar 활용을 통해 유연한 화면구조 설계와 휴먼 에러 감소라는 이점을 얻게 되었다.

---

#### 후기

먼저, DI에 대한 개념은 알고 있는 상태였다.  
다만 사용해본 적은 없어서 개발 측면에서 어떤 이점이 있는지 어떤 어려움이 있는지는 알지 못했다.  
라이브러리에 따라 객체들의 scope를 관리하는 등의 기능을 제공한다는 것을 알게 되었다.  
그리고 scope 제어를 위해 Navigation graph를 활용할 수 있다는 사실도 알게 되었다.  
<br>
모듈화 부분은 굉장히 관심있게 봤다.  
코드량이 많아지다 보면 아무래도 유지보수 측면에서 어려움이 생기는데, 모듈화를 통해 어느정도 어려움을 해소할 수 있지 않을까 하는 생각이 들었기 때문이다.  
Presentation, Domain, Data 레이어와 같은 클린 아키텍처 부분은 개념은 들어봤지만 추상적인 내용이 많다보니 아무래도 100% 이해한 것은 아니다.  
그래도 꾸준히 들여다 보며 이해를 해나가고 있는 영역이다.  
모듈화를 통해 결합도를 낮춘다면 테스트 코드도 더 효율적으로 작성할 수 있을 것 같은데, 테스트 코드에 대한 내용은 없어서 아쉽다.  
<br>
UI 개선 부분은 생각해보지 못했던 부분이었다.  
디자인 영역이라고 생각해서, 이를 활용해서 개발 부분의 이점을 얻을 수 있을 거라곤 생각하지 않았다.  
이번 세션에서 가장 새로운 세상을 깨닫게 해준 부분이었다.  
<br>
리빌딩 과정을 통해 DI, 모듈화, UI 개선 작업 내용을 들을 수 있었다.  
이미 알고 있었지만 직접 해보진 않았던 내용들도 있었고, 난생 처음 들었던 내용들도 있었다.  
라이브러리 결정 과정과 기타 문제 해결 방안들을 들을 수 있어서 유익했다.  
앱 리빌딩 진행 시 참고하여 진행하면 좋을 듯 하다.

##### Reference:

[카카오페이 앱 리빌딩 스토리 / if(kakao)2022](https://youtu.be/Y0szAIW_tFs)
[https://developer.android.com/training/dependency-injection](https://developer.android.com/training/dependency-injection)
