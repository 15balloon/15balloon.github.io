---
layout: post
logo: "assets/images/15balloon.png"
navigation: True
cover: "assets/images/cover1.jpg"
title: "View Binding Migration 문제 해결"
author: "15balloon"
date: 2024-01-30 23:19:59 +0900
tags: android error
categories: 15balloon
subclass: "post tag-test tag-content"
---

회사에 대략 6년 정도 된 앱이 하나 있는데, 그 앱을 담당했던 분이 퇴사하면서 내가 그 앱의 담당자가 되었다.   
전 담당자분은 앱 개발자는 아니었지만, 개발하셨던 것 같다.   
(타 부서라 잘 몰랐고, 그래서 이런 앱이 있는지도 몰랐다)   
아무튼 오래된 앱이고 개발자의 전문 분야가 앱이 아니다 보니 전체적으로 기술이 낡아 있었다.   
앱 전체가 Java로 되어 있었고, View Binding이나 Retrofit2도 사용하지 않았다.   
이 앱 담당자가 되었을 때부터 기술을 조금이라도 최신화하고 싶었지만, 진행하는 주 프로젝트가 있었고 이건 서브였기 때문에 언젠가 틈이 나기만을 기다렸다.   
그러다 이번에 시간이 조금 나서 일부 코드를 Kotlin으로의 Migration과 View Binding을 사용하도록 수정하는 작업을 진행하였다.   

#### 이상 현상
먼저 Activity와 같은 주요 클래스들을 먼저 수정하였는데, 이 과정에서 문제가 발생했다.   
setContentView() 수행 시 R.layout.activity_xxx 대신 View Binding을 사용하도록 변경하였는데 View가 다르게 보였다.   
<img src="{{site.baseurl}}/assets/images/2024-01-30-ViewBinding-Migration-Issue_1.jpg" width="100%" height="100%" title="normal view" alt="Image: Original View"/>
이렇게 보여야 하는 View인데,
<img src="{{site.baseurl}}/assets/images/2024-01-30-ViewBinding-Migration-Issue_2.jpg" width="40%" height="40%" title="problem view" alt="Image: Problem View"/>
이렇게 보였다.   

#### 코드 구조
해당 View의 간략한 구조는 다음과 같다.   
```xml
<LinearLayout
    android:layout_width="100dp"
    android:layout_height="match_parent">
    <TextView
        android:layout_width="wrap_content"
        android:layout_height="40dp"/>
</LinearLayout>
```

최상단에 LinearLayout이 있고 width가 100dp로 고정되어 있다.   
그리고 그 하위에 TextView가 있고 width는 wrap_content로 설정되어 있다.   

다른 View들은 괜찮았는데 딱 그 View만 문제였다.   
다른 View와 차이가 뭔지 살펴보니 이 View는 Manifest에 theme가 Dialog로 설정되어 있었다.   

#### 문제 해결
Dialog Theme는 해당 View에 포함된 실제 View의 크기에 따라 공간을 차지하게 되어 있다.   
그렇다 하더라도 LinearLayout의 width가 고정값이기 때문에 해당 크기에 맞춰 공간을 차지해야 하는데 이 값이 무시되었다.   
처음엔 View Binding에 문제가 있다고 판단하였는데, 그것이 아니고 overloading된 setContentView() 메소드에 차이가 있었다.   
setContentView(@LayoutRes int)를 수행할 때는 기존에 설정된 레이아웃 매개변수를 그대로 사용하지만, setContentView(View)를 수행할 때는 레이아웃 매개변수가 무시되고 width와 height가 match_parent로 설정된다.   
(width와 height가 match_parent로 설정된 건 Layout Inspector Tool로 확인할 수 있다)   
<img src="{{site.baseurl}}/assets/images/2024-01-30-ViewBinding-Migration-Issue_3.jpg" width="100%" height="100%" title="setContentView" alt="Image: setContentView"/>
이 설명은 Activity.class 파일에서 확인할 수 있다.   

따라서 아래와 같이 inflater를 사용하여 view 객체를 생성한 경우 setContentView(View) 메소드를 사용하므로 LinearLayout의 width 값이 match_parent로 바뀌어서 View가 다르게 보이게 된다.   
```kotlin
val inflater = getSystemService(Context.LAYOUT_INFLATER_SERVICE) as LayoutInflater
val view = inflater.inflate(R.layout.activity_xxx, null) as View
setContentView(view)
```

#### 해결 방법
이를 해결하기 위해서는 Activity.class 파일에 쓰여 있는 설명대로 setContentView(View, ViewGroup.LayoutParams) 메소드를 사용하면 된다.   

#### Conclusion
Migration 과정에서 발생한 문제를 해결하며 findViewById와 View Binding의 동작 원리에 대해서 다시 한번 살펴보게 되었고, setContentView() 메소드 종류가 다양하다는 것과 동작에 차이가 있다는 것을 알게 되었다.   

##### Reference:

1. [Why using View Binding is changing the layout?](https://stackoverflow.com/questions/61695769/why-using-view-binding-is-changing-the-layout)
