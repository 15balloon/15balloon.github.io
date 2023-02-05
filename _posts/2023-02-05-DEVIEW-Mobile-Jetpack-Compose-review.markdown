---
layout: post
logo: "assets/images/15balloon.png"
navigation: True
cover: "assets/images/cover1.jpg"
title: "DEVIEW 2021 모바일: 'Android Jetpack Compose 실제 서비스 적용 후기' 세션 후기"
author: "15balloon"
date: 2023-02-05 21:30:11 +0900
tags: android conference
categories: 15balloon
subclass: "post tag-test tag-content"
---

##### 세션 내용 요약

1. 적용 배경

   - React-Native에서 Android Native로의 앱 리뉴얼 미션을 부여받음
   - 해당 앱이 적은 Spec과 화면을 가졌기에 Trouble Shooting에 부담이 없으리라 판단
   - 기존 Project가 View의 onDraw를 사용하는 자체 Framework이었기에 Jetpack Compose 개념 이해가 수월

2. Jetpack Compose   
    기존의 Native UI는 명령을 통해 Page를 표시한다.   
    View를 XML로 미리 구성하고, findViewById나 binding으로 View를 불러와서 변화를 명령하는 방식이다.   
    선언형 Jetpack Compose는 모든 Page 구성요소가 정의 되어 있고, State에 따라서 표시 여부를 결정한다.   
    그리고 싶은 View를 모두 정의해놓고, State에 맞게 View를 선택적으로 그린다.   
    
    Jetpack Compose를 사용하면 다음과 같은 장점이 있다.
    1. Build 속도가 줄어들어 생산성이 향상한다.
    2. APK 크기가 감소한다.  
      1, 2번에 대한 자세한 내용은 [Reference 3번](https://developer.android.com/jetpack/compose/ergonomics?hl=ko)을 참조하면 된다.
    3. 웹뷰와 같은 Native 기능과 Context를 쉽게 사용할 수 있다.
    4. Kotlin Code만으로 대부분의 UI 개발이 가능하다는 것도 큰 장점이다. XML과 Kotlin Code를 왕복하면서 개발할 필요 없기 때문에 개발자 입장에서는 더 편하다.
    5. RecyclerView와 Adapter 사용 없이 List를 만들 수 있다. RecyclerView를 사용하면 재사용되는 View 처리 등등 신경 써야 할 부분도 많고, 헷갈리는 부분도 많다. 그런데 이것 대신 Jetpack Compose 내에서 편리하게 Row, Column 기능을 사용할 수 있다.
    6. 데이터(literals) 수정을 하면 Emulator에서 실시간(Live Edit)으로 볼 수 있다.
    7. State 변화로 인해 UI 갱신이 일어날 때 UI가 속한 Tree를 전부 탐색하는 것이 아닌, 변경되는 UI만 탐색하기 때문에 가볍다.

    당연히 단점도 존재하는 데 단점은 다음과 같다.
    1. LifeCycle 대응이 안 된다. Composable 내부에서 LifeCycle 변경 여부를 알아챌 수 없어서, Activity에서 LifeCycle 변경이 일어날 때 Composable에 이를 알려줘야 한다.
    2. 같은 화면에 있는 Widget이 서로 다른 함수로 이루어져 있다면 변수를 공유할 수 없다. DI, 전역 관리, 매개변수 등을 사용해서 변수를 공유할 수 있도록 해야 한다.
    3. 같은 Page에 속한 Widget들이 사용하는 State는 Widget의 상위인 Page에서 생성하여 관리한다. 그렇기 때문에 해당 Page가 삭제되면 Widget의 State도 초기화된다. 이처럼 State가 초기화되는 상황에서 값을 유지하기 위해서는 remember, rememberSaveable을 활용해야 한다. State 관리가 제대로 이루어지지 않으면 화면 갱신이 안 되거나, 원하는 데이터를 얻지 못할 수 있다.
    4. UI가 Activity Base 구성이 아니기 때문에, Firebase Tracking 활용 시 화면별 Tracking이 어렵다. 화면 진입 시점을 Logging에 추가해야 한다.
    5. 아키텍처 자료가 부재하다. (2021.11. 발표이므로 이때 기준) MVC, MVVM, Clean Architecture 등 Jetpack Compose 사용과 관련한 아키텍처에 대한 자료를 찾아보기 힘들다.
    6. 모든 Page가 State로 연결되어 관리되기 때문에 잘못된 구성으로 인한 오류 발생 여지가 존재한다. 논리적으로 구성만 잘 한다면 문제가 없겠지만, 세상에 완벽한 건 없는 법이다.

    Jetpack Compose 사용 시 주의해야 할 점은 다음과 같다.
    1. State가 변경되면 해당 State를 사용하는 전체 Composable 함수가 Recomposition되는게 아니라, 해당 함수 내에서 변경된 State를 사용하는 Composable만 Recomposition된다.
    2. State로 Model을 사용할 때 Model의 파라미터만 변경이 된 경우에는 Recomposition되지 않는다. Model 자체가 변경되어야 Recomposition된다.
    3. Composable은 여러 번 호출될 수 있기 때문에, 내부에서 한 번만 수행되어야 하는 함수들은 LaunchedEffect 등을 사용하여 호출이 다량 일어나지 않도록 해야 한다.
    4. Composable 내부에서는 변수 값이 변경되어도 Recomposition 되지 않기 때문에, 변수에 따라서 UI가 변경되어야 하는 경우에는 State를 사용해야 한다.
    5. Composable 내부에서 변수 사용 시 remember를 사용하지 않으면 Recomposition 과정에서 변수가 재생성된다. 객체 또한 마찬가지다.
    6. Parcelable, Serializable Object를 사용했을 때 Object 내부 값이 변경되지 않으면 Recomposition되지 않는다.

---

#### 후기

Jetpack에 Compose라는 선언형 UI Framework가 있다는 사실은 처음 알았다.   
본 세션은 Compose에 대한 기본 개념을 이미 알고 있다는 것을 전제로 이루어졌기 때문에 Android Developer 문서를 간단하게 읽고 시청하였다.   
<br>
실제 서비스를 적용한 후기이다 보니 세세한 장단점을 알 수 있어서 좋았다.   
코드를 짜면서 겪는 사소한 일화들이 담긴 세션이었다.   
다만, Jetpack Compose를 처음 접한 사람에겐 다소 불친절한 세션이었다.   
아무래도 기본 개념을 설명하기엔 시간이 촉박해서 어쩔 수 없었을 것 같다.   
<br>
XML을 작성하지 않고 View를 구성할 수 있다는 점이 굉장히 매력적이었다.   
세션에서 장점으로 언급한 것과 같이, XML과 코드를 번갈아 가며 분석하는 것은 피곤한 일이다.   
Layout뿐만 아니라, Selector, Style 등도 같이 봐야 하니 번거롭다.   
그런데 이러한 파일없이 코드로만 작성하여 View를 편리하게 구현할 수 있으니 흥미로웠다.   
간단한 앱을 개발할 때는 Jetpack Compose 사용을 고려해봐도 괜찮을 것 같다.

##### Reference:

1. [Android Jetpack Compose 실제 서비스 적용 후기 / DEVIEW 2021](https://deview.kr/2021/sessions/478)
2. [https://developer.android.com/jetpack/compose/documentation?hl=ko](https://developer.android.com/jetpack/compose/documentation?hl=ko)
3. [https://developer.android.com/jetpack/compose/ergonomics?hl=ko](https://developer.android.com/jetpack/compose/ergonomics?hl=ko)
