---
layout: post
logo: "assets/images/15balloon.png"
navigation: True
cover: "assets/images/cover1.jpg"
title: "안드로이드 텍스트뷰 Width 이상 현상"
author: "15balloon"
date: 2023-01-08 22:52:55 +0900
tags: android error
categories: 15balloon
subclass: "post tag-test tag-content"
---

#### Android Textview width 이상 현상 수정

안드로이드 앱 개발을 하던 중 폰트를 수정하면서 Textview의 width가 이상한 문제를 발견했다.  
<img src="{{site.baseurl}}/assets/images/2023-01-08-Strange-Android-Textview-width_1.jpg" width="100%" height="100%" title="problem" alt="Image: Strange Textviews"/>
위 사진에서 볼 수 있듯 같은 문장인데 Textview들의 길이가 차이 났다. (위 사진은 그때의 상황을 재현한 것임)  
둘 다 코드에서 자간이나 폰트 크기 등을 설정해주긴 하지만 모두 동일하게 설정해주고 있었고, 다르게 적용하는 부분은 없었다.  
그나마 차이라 한다면 하나는 Scrollview를 사용하고, 하나는 Textview를 사용한다는 것이다.  
혹시 이 둘의 차이가 있는 건지 검색해보았지만, 당연히 관련된 내용은 찾을 수 없었다.  
차이가 나는 Textview들을 뚫어지게 살펴보니 View 전체가 아니라 자간에서만 차이를 보이는 것을 발견했다.  
그렇지만 코드에서 자간은 모두 동일하게 설정해주고 있었기 때문에 더욱 의문에 빠질 수밖에 없었다.  
게다가 시스템 기본 폰트를 사용할 때는 보이지 않았다가, 새로운 폰트를 적용하니 문제가 보여서 더 혼란스러웠다.  
몇 시간을 구글에서 헤매던 중에 하나의 Textview는 코드 내에서 글자를 조합해서 보여주는 방식이고, 다른 하나의 Textview는 웹페이지에서 입력한 값을 서버를 통해 받아와서 보여주는 방식임을 보고 두 문자열을 로그에 찍어 보았다.  
당연히 로그 상에서는 'It has strange width...'로 동일하게 보였다.  
뭐가 문제일까 생각하다 코드 내에서 다시 로그를 찍어 봐야겠다 하고 아무 생각 없이 로그에 찍힌 두 번째 Textview의 문자열을 복사해서 아래와 같이 붙여 넣었다.  
그랬더니 아래와 같은 광경이 내 눈앞에 펼쳐졌다.

```kotlin
Log.d(TAG, "It has strange width...") // What I expected...
Log.d(TAG, "It[NBSP]has[NBSP]strange[NBSP]width...") // But the following happened...!
```

여기서 [NBSP]는 HTML에서 볼 수 있는 &nbsp가 맞다.  
No-Break Space의 줄임말이며, 줄바꿈 없는 공백이란 뜻이다.  
NBSP의 유니코드는 '\u00A0'이다.  
그런데 일반 공백, 우리가 흔히 스페이스 바로 작성하는 공백의 유니코드는 '\u0020'이다.  
즉, 두 글자는 같아 보이지만 전혀 다른 글자이다.  
기본 폰트는 '\u0020'이든 '\u00A0'이든 거의 비슷한 너비를 가져서 두 Textview들의 길이 차이가 거의 없었지만, 일부 폰트들은 일반 공백인 '\u0020'의 너비만 폰트 컨셉에 맞게 지정해 놓았는데 '\u00A0'의 너비는 별도로 지정해 놓지 않아서 기본 너비만큼만 작성되었기 때문에 차이가 났던 것이라고 결론 내렸다.  
그리고 Textview에 넣을 문자열의 '\u00A0'을 모두 '\u0020'으로 replace하여 사용하도록 수정하여 문제를 해결하였다.

##### Reference:

Just me
