---
layout: post
logo: "assets/images/15balloon.png"
navigation: True
cover: "assets/images/cover1.jpg"
title: "Flutter 터치하지 않았는데 onTap 불리는 문제 해결"
author: "15balloon"
date: 2024-03-15 21:01:37 +0900
tags: android error flutter
categories: 15balloon
subclass: "post tag-test tag-content"
---

Card 위젯을 사용해 데이터 목록을 표시하는 레이아웃을 만들었다.   
그리고 하나의 Card 위젯을 선택할 수 있도록 GestureDetector 위젯을 사용했다.   
(이후에 InkWell 위젯도 알게 되었다...!)   
onTap callback 함수를 작성하여 Card 위젯이 선택되면 border를 표시하도록 했다.   

#### 이상 현상
레이아웃이 포함된 화면으로 전환될 때 빨간 화면이 떴다.   
오류를 보니 (대충) 찬물도 위아래가 있는데 UI가 그려지기도 전에 UI를 변경하자고 우기면 어떡하냐는 내용이었다.   
로그를 찍어 봤는데 단지 화면이 전환되었을 뿐인데 onTap이 불렸다.   
터치한게 아니고 화면이 표시되었을 뿐인데 onTap이 불린 것이다.   

#### 문제 해결
바로 오류 상황을 구글에 검색해보니 역시나 누군가가 질문한 글이 있었다.   
답변은 다음과 같다.   
onTap은 GestureTapCallback 함수의 한 종류이기 때문에   
```dart
// Wrong code
onTap: somethingFunction(),
```
이렇게 사용하면 사용자의 Tap 입력없이 바로 함수가 실행된다.   
그러니 다음과 같이 사용해야 한다.   
```dart
// Correct code
onTap: () {somethingFunction()},
```
답변을 보고 코드를 보니 정말 저렇게 잘못 작성했다.   
답변대로 작성하여 문제를 해결할 수 있었다.   

조금만 생각해보면 onTap은 함수 타입이고 somethingFunction은 void 타입이라 오동작했다는 것을 깨달을 수 있었을 텐데 왜 그 당시에는 그런 생각이 안 떠올랐는지 모르겠다.   

#### Conclusion
프로젝트 마감 기한이 촉박해서 어쩔 수 없이 아무런 지식이 없는 상태로 당장 Flutter 개발을 시작하게 되었다.   
지금 글을 작성한 시점 기준 Flutter를 시작한지 2주가 되었다.   
그래서 Dart 언어와 Flutter 기초 지식이 너무나도 부족했다.   
그나마 레이아웃 구성이 Jetpack Compose와 유사하다는 것이 위안이다.   

Java, Kotlin과 Dart가 어떤 부분이 다른지 아예 모르니 코드를 작성하면서 이게 왜 안 되지 하는 순간이 있었다.   
Dart에서 람다식은 무조건 한 줄로만 작성해야 한다는 것도 이번에 처음 알게 되었다.   
그리고 Dart 언어 특성을 살려서 코드를 작성하는 것 같지 않아서(느낌적인 느낌) 아쉽다.   
유지보수하면서 조금씩 고쳐나가야 겠다.   

Dart랑 Flutter 공식 문서가 정말 잘 작성되어 있다.   
어떤 요소를 사용하고 싶은지만 알아내서 공식 문서만 보고 코드를 짜도 괜찮았다.   
아무래도 구글이 출시한 프레임워크라서 그런 것 같다.   

##### Reference:

1. [Why onTap function runs without tapping when it have arguments?](https://stackoverflow.com/questions/75600597/why-ontap-function-runs-without-tapping-when-it-have-arguments)
2. [Dart - 함수](https://dart-ko.dev/language/functions)